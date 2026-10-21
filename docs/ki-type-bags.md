---
title: Ki Type Bags
nav_order: 11
---

# Ki Type Bags: Design v0.1

**Status:** Design accepted 2026-10-21 (the decisions of section 8 are Dan's). Implementation starts with TB1 (section 7); nothing is built yet.
**Date:** 2026-10-21
**Scope:** An extension mechanism in Ki.Core that lets a library add value types, functions, units and constants to KD, KS, Boa, KQ and RevDB at once, and five libraries built on it: Ki.Math, Ki.Sci, Ki.Physics, Ki.Chem, Ki.Bio. Replaces `io.kixi.labnexus.lab` in LabNexus.
**Reads with:** RevDB-Design-v0.7.md (3.1, 4, 5.4, 5.5, 7.4), Ki.Core `io.kixi.lang` (the embedded-language registry this copies), Ki.KS `KSMembers`.

---

## 1. Why

Ki.Core's `Type` enum is the value set of KD, KS, KQ and RevDB; the four agree because they share it. Domain types (an element, a molecule, a DNA strand) must keep that property, so they cannot be added to RevDB, or to KS, or to LabNexus: they have to enter at the bottom, through Ki.Core, once, and every layer has to pick them up without knowing which domains exist. Ki.Core does not depend on any domain either: a chemistry library depends on Ki.Core, never the reverse.

The mechanism exists in miniature already. `EmbeddedLanguage` lets RevDB add KQ to KS through `META-INF/services` with no dependency from KS on RevDB; `Unit.addUnit` lets anyone add a unit; `Parseable.parseLiteral` is how `.id(`, `.ref(`, `.geo(` reach KD. A type bag is the same shape, generalized: a library that registers value types, functions, units and constants, discovered at runtime.

Principles, in order:

1. **One definition, every layer.** A type is written once in its bag (parse, format, binary form, order, members). KD, KS, Boa, KQ and RevDB consume it through the contract; none of them has code for a particular domain type.
2. **Databases outlive bags.** A RevDB file written with a bag installed must open, read and write every document without it. An extension value whose bag is absent is an opaque value: it round-trips, prints as its literal, and is not an error.
3. **Few and durable.** A bag's types are the handful that carry identity and operations of their own (an element, a sequence, a molecule). Everything else is a document shape with a KDS schema. A new value type costs a literal, a codec, an order, members and tests in five places; a document shape costs a schema.
4. **No dependency creep.** Ki.Core and RevDB keep their dependencies (RevDB: four). A bag depends on Ki.Core (and on Ki.Sci when it extends Ki.Sci). A bag that needs a heavy library (cheminformatics, numerics) is split: a thin bag with the types, an optional one with the library.
5. **Kotlin's behavior unless there is a reason, written down** (RevDB design 4.1), for the bags' Kotlin API as for everything else.

---

## 2. The contract (Ki.Core, package `io.kixi.types`)

```kotlin
/** A library of value types, functions, units and constants for one domain. */
interface TypeBag {
    val name: String                       // "sci", "bio": the `use` name and the prefix in RevDB's encoding
    val version: Int                       // codec version, bumped when an encoding changes; decoders accept every older one
    val requires: List<String>             // bags this one extends ("sci"); loaded first
    val types: List<ValueType<*>>
    val functions: List<BagFunction>       // named functions for KS, Boa and KQ
    val units: List<Unit>                  // registered with Unit.addUnit at load
    val constants: Map<String, Any?>       // e.g. "avogadro" to 6.02214076e23/mol
}

/** One value type: how it is written, stored, ordered and navigated. */
interface ValueType<T : Any> {
    val name: String                       // "DNA", "Element": the KDS type name and what `is` checks
    val kotlinClass: KClass<T>
    val literal: String?                   // the dot-literal name: ".dna(" ; null for a type with no literal of its own
    fun parse(text: String): T             // the text inside the literal's parentheses (a KD string or bare token)
    fun format(value: T): String           // the literal as KD writes it, including the dot and parentheses
    fun encode(value: T): ByteArray        // RevDB's stored form, self-contained (no references to the bag's tables)
    fun decode(bytes: ByteArray, version: Int): T
    fun sortKey(value: T): ByteArray?      // order-preserving bytes, or null for a type that does not order
    fun members(value: T): KSMembers?      // the value's members for KS, Boa and KQ; null when the value is a KSMembers itself
}

/** A function the bag contributes to KS, Boa and KQ. */
class BagFunction(val name: String, val arity: IntRange, val call: (List<Any?>) -> Any?,
                  val resultType: String? = null)      // a ValueType name or a Ki.Core Type name, when known

/** The registry: bags on the classpath (META-INF/services/io.kixi.types.TypeBag) plus explicit ones. */
object TypeBags {
    fun register(bag: TypeBag)
    val all: List<TypeBag>
    fun byName(name: String): TypeBag?
    fun typeFor(value: Any): ValueType<*>?             // by kotlinClass, for formatting, encoding, members
    fun literal(name: String): ValueType<*>?           // ".dna" -> Bio's DNA
    fun typeNamed(name: String): ValueType<*>?         // "DNA" -> for KDS and `is`
}
```

Rules:

- **Names are global and checked.** Two bags may not register the same literal name or type name; the second registration fails at load with both bag names in the message. Bag names are lowercase identifiers.
- **Loading order follows `requires`.** Bio requires Sci; Sci's types exist before Bio registers types that embed them.
- **Literals are KD's.** A bag's literal is `.name(…)` with the usual KD rule for what is inside: a string (`.dna("ATG")`), or a bare token where the type's grammar is unambiguous (`.element(Na)`, `.el(Na)`). The bag's `parse` sees the raw inside text. The dot-literal convention (RevDB design 4.1, item 3) is unchanged; bags extend the set `.id`, `.ref`, `.geo`, `.blob`, `.coordinate` carry today.
- **`Type.Ext`.** Ki.Core's `Type` enum gains one entry, `Ext`, for "a registered extension type"; `Type.of(value)` returns it when `TypeBags.typeFor(value)` is non-null. KDS type names resolve through `TypeBags.typeNamed` when they are not a core type, so `attribute sequence type=DNA` works once Bio is loaded.
- **Equality and hashing are the Kotlin class's** (`equals`/`hashCode`), as for every Ki.Core type.
- **Versioning.** A bag's `version` is written with every encoded value; `decode` must accept every version the bag ever shipped. An encoding change without a version bump is a bug the RevDB corpus test catches (5.5's round-trip test runs over every bag's sample values).

---

## 3. What each layer does with it

### 3.1 KD (Ki.KD-JVM)

- The literal parser, on `.name(`, asks `TypeBags.literal(name)` when the name is not a built-in; the bag's `parse` makes the value. `Ki.format` asks `TypeBags.typeFor(value)` to write it back. A KD document with `.dna("ATG")` reads and writes without KD knowing what DNA is.
- When no bag has the literal, KD produces an `ExtLiteral(name, text)` value: an opaque literal that formats back to exactly what was read. This is the "databases outlive bags" rule at the text level: a KD file written with Bio reads without it.
- KDS: an unknown type name in a schema resolves through the registry; validation of an attribute of type `DNA` checks `TypeBags.typeNamed("DNA").kotlinClass.isInstance(value)`.

### 3.2 KS (Ki.KS-JVM)

- **Literals** need no import: a KS source reads them through KD's literal path.
- **Members**: the curated member dispatch gains one branch before the reflection backstop: `TypeBags.typeFor(obj)?.members(obj)?.ksMember(name)`, or `obj` itself when it implements `KSMembers`. So `seq.gc`, `mol.formula`, `el.atomicNumber` resolve in every `hostLang` mode, portable included.
- **Types**: `checkType` accepts a registered type name, so `x is DNA`, `catch e: ...`, `let s: DNA = ...` work.
- **Functions and constants** come in with `use`: `use ki.bio` binds Bio's `functions` and `constants` by name in the current scope (`revcomp(s)`, `translate(s)`, `avogadro`); `use ki.bio.revcomp` binds one; `use ki.sci as sci` binds the bag as an object (`sci.element("Na")`). `ki.<bag>` is a new import source beside JVM classes and KS types in `ImportRegistry`; nothing else in `use` changes. Why an import for functions and not for members: a member is reached through a value that already is the bag's type, so no ambiguity; a bare function name is global and should be asked for.
- **Construction** without a literal: `DNA("ATG")` after `use ki.bio` (the type's name is bound as a callable that calls `parse`).

### 3.3 Boa (Ki.Boa-JVM)

The same three paths through `Interop`: literals via KD, members via `typeFor(value).members`, functions and constants via `import ki.bio` (Boa's import statement, resolving `ki.<bag>` as KS's `use` does). Boa is later; the contract does not change for it.

### 3.4 KQ (RevDB-JVM)

- **Members**: `Evaluator.member` adds the same registry branch as KS, so `s.sequence.gc` and `c.structure.formula` work in a `where` or `select`.
- **Functions**: the checker's `FUNCTIONS`/`ARITY` tables become a registry: built-ins first, then every loaded bag's `functions`. Arity is checked at plan time as today; a bag function's `resultType` lets the checker type the result when the bag says what it is.
- **Comparison and order**: `Semantics` compares two values of one extension type by their `sortKey` when the type has one, and refuses `<` for a type without one (as it does for a grid); `==` is Kotlin's `equals`. `orderby` and `distinct` use the key; `KeyEncoding` carries it (5.4).
- **Literals** in queries are KD literals already (7.4), so `.dna("GAATTC") in s.sequence` needs nothing new once `in` understands sequences (a bag function or member: `s.sequence.contains(.dna("GAATTC"))`).

### 3.5 RevDB storage and indexes

- **Encoding** (5.5): one new value tag, `T_EXT`, followed by the bag name, the type name, the bag version and the bytes from `encode`. Decoding looks the bag up; when it is absent, the value becomes `ExtValue(bag, type, version, bytes)`, an opaque value that encodes back identically and formats as `.<literal>(...)` only if the bag also stored a text form. To make that always possible, `T_EXT` stores both: the bytes and the literal text. The text costs a little space and buys the opaque round trip through KD too (export and import without the bag). Decision: store both.
- **Indexes** (5.4): a value with a `sortKey` is indexable; the key is `KIND_EXT` + bag name + type name + the sort key, so values of one type sort together and never compare with another kind, exactly as quantities of different dimensions do. A type without a sort key is stored, not indexable, like a grid.
- **Schema**: `attribute x type=DNA` validates through KDS as above.
- **Documents are not affected**: a value that is a bag type behaves as any other value in a tag; molecules and sequences are attribute values of compound and gene documents, which is the shape the example corpus already has with strings.

---

## 4. Packages, artifacts and importing

A bag is three things with one name:

| | Kotlin | KS / Boa | RevDB |
|---|---|---|---|
| Artifact | `io.kixi:Ki.Bio-JVM` (Gradle) | on the classpath of the host (LabNexus) | discovered through `META-INF/services` |
| Package | `io.kixi.bio` (`import io.kixi.bio.DNA`) | `use ki.bio` | `bio` in `T_EXT` and in index keys |
| Entry | `object Bio : TypeBag` | the bag object, bound by `use ki.bio as bio` | `TypeBags.byName("bio")` |

So: a package is part of a bag, not the other way round. One bag is one artifact, one Kotlin package root and one registration. The scripting name `ki.<bag>` is deliberately not the Kotlin package name: scripts should not care that the implementation is on the JVM, and Boa will say the same `import ki.bio`. A bag may contain several Kotlin packages (`io.kixi.bio.seq`, `io.kixi.bio.codon`); the scripting surface is flat and curated (`functions`, `constants`, `types`), which keeps script names stable while Kotlin internals move.

Repositories: `kixi-io/Ki.Math-JVM`, `Ki.Sci-JVM`, `Ki.Physics-JVM`, `Ki.Chem-JVM`, `Ki.Bio-JVM`, each a composite-build sibling like the others, Kotlin 2.3.0 and Kotest 6.0.7 (they are consumed by KS and RevDB, which are on 2.3; Ki.Core and Ki.KD stay on 2.0.20). Version `0.1.0`. LabNexus includes them through `includeBuild` and substitution as it includes RevDB.

---

## 5. The five bags

Dependencies: Math and Sci depend on Ki.Core; Physics, Chem and Bio depend on Sci (and so on Math, transitively, when Sci uses Measure). Bio depends on Chem only if it needs reactions, which v1 does not; keep Bio on Sci alone and revisit.

### 5.1 Ki.Math (`ki.math`)

The numeric types the sciences share, with no domain in them.

- `Measure`: a value with uncertainty, `12.3nM ± 0.4nM`, over a number or a quantity; arithmetic propagates uncertainty (sum in quadrature for independent errors); literal `.measure(12.3nM ± 0.4nM)` or, if KD gets the `±` syntax planned for v1.1, a plain literal. This is RevDB's v1.1 item, moved here.
- `Vector<T>`: a packed 1D numeric array (Int, Long, Float, Double, Dec) for time series, spectra and chromatograms; literal `.vec<Double>(1.0 2.5 3.1)` or `.vec<Float> { ... }` in the grid's style; stored packed; `length`, `min`, `max`, `mean`, `sum`, slicing, `at(i)`. KQ aggregates accept it as a sequence. Orders by length then elements (rarely useful, but defined).
- `Interval` (closed numeric interval with units, for calibration and model ranges): the `Fit` range checks in LabNexus's `Fit.kt` use this.
- `Fit`: the model-fitting values in LabNexus's `Fit.kt` (four-parameter logistic for dose-response, linear calibration), as values with `parameters`, `r2`, `predict(x)`, `inverse(y)`; `zprime` moves here from KQ's built-ins (KQ keeps its name, implemented by the bag when loaded, by RevDB when not, until RevDB depends on nothing new).
- Functions: `mean`, `sd`, `median`, `quantile`, `linreg`, `fit4pl`, `zprime`, `round(x, places)`.
- Constants: none.

### 5.2 Ki.Sci (`ki.sci`)

The models of matter that Physics, Chem and Bio all read, with the extension points they use.

- `Element`: the periodic table (number, symbol, name, standard atomic weight with its interval where IUPAC gives one, group, period, block); literal `.element(Na)`; orders by atomic number; `Elements.bySymbol`, `.byNumber`. LabNexus's `Element.kt` is the seed.
- `Isotope`: `Element` plus mass number and exact mass; `.isotope(13C)`.
- `Formula`: an empirical or molecular formula (`C6H12O6`) as an ordered map of element to count, with mass, `times(n)`, Hill-order formatting; literal `.formula(C6H12O6)`. LabNexus's formula parsing in `Molecule.kt` is the seed.
- `Molecule`: a structure: atoms (element, charge, isotope, aromatic flag, hydrogen count), bonds (order, stereo), read from and written to SMILES; `formula`, `mass` (average and monoisotopic), `atomCount`, `rings`, canonical SMILES for equality; literal `.smiles("CCO")`. InChI: `.inchi("InChI=1S/…")` is a value with a validated layer structure and a derived InChIKey *when the InChI library is present* (5.4, Ki.Chem.Ext); without it, the InChI string is stored and compared as text, and the key is `nil`. Equality of two molecules is canonical-SMILES equality; InChI equality is text equality.
- **Extension points** (what makes Sci the base): `Molecule` and `Element` carry an open `annotations: Map<NSID, Any?>` that other bags fill (Chem: reactivity classes; Bio: residue names), and `Molecule` has a `kind` (`small`, `polymer`, `biopolymer`) that Bio's types set. A domain bag adds *members* to Sci's types through its own `members(value)` for values it recognizes (the registry asks every loaded bag, in `requires` order, so Bio can answer `mol.residues` for a peptide Sci cannot). A domain bag never redefines Sci's encoding.
- Functions: `element(x)`, `formula(x)`, `molecule(x)`, `mass(x)`.
- Units: none beyond Ki.Core's (`Da`, `mol` and the SI set are already there).
- Constants: `avogadro`, `electronMass`, `protonMass`, `neutronMass`, `boltzmann`, `planck`, `speedOfLight`, `gasConstant`, `faraday`, `elementaryCharge` (CODATA 2022, as quantities).

### 5.3 Ki.Physics (`ki.physics`)

The smallest of the five in v1: units and constants beyond the SI set, and the values a lab instrument reports.

- Units: `eV` (and keV, MeV), `Gy`, `Sv`, `Bq`, `Ci`, `barn`, `Torr`, `atm`, `bar`, `cal`, `kWh`, `G` (gauss), `T` (tesla) if Ki.Core lacks them, `rpm`, `×g` (relative centrifugal force, for Bio's benches).
- `Spectrum`: a `Vector` of intensities with an axis (wavelength, wavenumber, m/z, time) and its unit; `peaks(threshold)`, `at(x)`, `integrate(a, b)`; this is the type a plate reader's spectral scan, a UV-Vis trace or a mass spectrum lands in. Literal: none (documents carry it as a tag with two vectors); the type exists for members and functions.
- Functions: `rcf(rpm, radius)`, `rpm(rcf, radius)`, conversions between energy forms (`eV` to `J`, wavelength to energy).
- Constants: the CODATA set in Sci, plus `gravity`, `vacuumPermittivity`, `stefanBoltzmann`.

### 5.4 Ki.Chem (`ki.chem`)

Reactions and solutions, over Sci's molecules.

- `Reaction`: reactants and products with stoichiometry, balanced or not, `balance()`, `yield(actual, limiting)`, `limitingReagent(amounts)`; literal `.reaction("2 H2 + O2 -> 2 H2O")`. LabNexus's `Reaction.kt` is the seed.
- `Solution`: a solute (molecule or formula) at a concentration in a solvent, with `dilute(to)`, `volumeFor(amount)`, `massFor(volume)`; the solution-preparation half of LabNexus's `LabKit`.
- `pKa`, `pH` helpers as functions, not types.
- Functions: the `LabKit` set (molar mass, moles from mass, dilution, mix), named as they are in RevGrid today so formulas keep working.
- **Ki.Chem.Ext** (optional, separate artifact `Ki.Chem-Ext-JVM`): the cheminformatics bag that depends on CDK (or RDKit through JNI): substructure and similarity search, depiction, InChI generation (`inchiOf(molecule)`, InChIKey), property prediction. It registers no new types; it adds functions and members to Sci's `Molecule` through the extension point. Nothing in LabNexus depends on it; when it is on the classpath, `mol.inchiKey` stops being `nil` and `substructure(a, b)` exists.

### 5.5 Ki.Bio (`ki.bio`)

- `Sequence<Alphabet>`: one type with an alphabet parameter rather than four types: `DNA`, `RNA`, `Protein`, and the XNAs (`TNA`, `PNA`, `LNA`, custom) as alphabets, so `.dna("ATG")`, `.rna("AUG")`, `.protein("MKT")`, `.seq(TNA, "ATG")`. Members: `length`, `gc` (nucleic), `complement`, `reverseComplement`, `transcribe`, `reverseTranscribe`, `translate(frame, table)`, `codons(frame)`, `orfs(min)`, `find(motif)`, `contains`, `molecularWeight`, `kind`, `alphabet`, `slice(a, b)`. Storage: 2 bits per base for the four-letter alphabets, 4 bits with IUPAC ambiguity codes, 5 bits for protein, bytes otherwise; a 10 kb plasmid is 2.5 KB. Orders lexically within one alphabet. LabNexus's `Sequence.kt` is the seed; `GeneticCode` becomes `CodonTable` with the NCBI tables (standard, vertebrate mitochondrial, bacterial, ...).
- `Motif`: a pattern over an alphabet with IUPAC ambiguity (`GAANNNNTTC`), `matches(seq)`, `find(seq)`; literal `.motif("GAATTC")`.
- `Residue` and `Nucleotide`: the monomers, as Sci `Molecule`s with Bio annotations (this is the extension point in use: a nucleotide is a molecule Bio recognizes).
- `Primer`: a `DNA` with `tm` (nearest-neighbour), `gcClamp`, `hairpin` checks: members on a sequence, not a type, unless annealing parameters need storing, which they do (salt, concentration), so `Primer` is a small value: sequence plus conditions.
- Functions: `revcomp`, `translate`, `transcribe`, `gc`, `tm`, `align(a, b)` (Needleman-Wunsch and Smith-Waterman, small inputs), `hamming`, `levenshtein`, `orfs`.
- Units: `bp`, `nt`, `aa` as counting units; `kb`, `Mb`.
- Constants: codon tables by NCBI id.

---

## 6. Migration from `io.kixi.labnexus.lab`

| Today (LabNexus) | Goes to |
|---|---|
| `Element.kt`, `Elements` | Ki.Sci `Element` |
| `Molecule.kt` (formula-based), `CommonCompounds` | Ki.Sci `Formula` and `Molecule`; `CommonCompounds` becomes a Chem constant table |
| `Sequence.kt`, `SeqKind`, `GeneticCode` | Ki.Bio `Sequence`, alphabets, `CodonTable` |
| `Reaction.kt` | Ki.Chem `Reaction` |
| `Fit.kt`, `FitRangeError` | Ki.Math `Fit`, `Interval` |
| `LabKit.kt` | Ki.Chem functions (solutions, stoichiometry) and Ki.Math (statistics) |
| `RGLabBindings` (KS and Boa wrappers) | gone: the bags bind themselves through `use ki.chem`; RevGrid's formula language does `use ki.chem` and `use ki.bio` at startup |

LabNexus keeps nothing of `lab` except, for one release, type aliases marked deprecated so plugins compile. RevGrid formulas keep their function names.

---

## 7. Milestones

| | Deliverable | Depends on |
|---|---|---|
| TB1 | The contract in Ki.Core (`io.kixi.types`), `Type.Ext`, KD literal dispatch and `ExtLiteral`, KDS resolution; RevDB `T_EXT` with text, opaque values, `KIND_EXT` index keys, KQ members and functions from the registry; KS members, `is`, `use ki.<bag>`; tests with a sample bag inside each repo's tests | |
| TB2 | Ki.Math and Ki.Sci, from the LabNexus seeds; the corpus round-trip test over their sample values | TB1 |
| TB3 | Ki.Bio; LabNexus switches `lab` to the bags; RevGrid formulas and the KS examples use `use ki.bio` | TB2 |
| TB4 | Ki.Chem (and the decision on Ki.Chem.Ext) | TB2 |
| TB5 | Ki.Physics | TB2 |
| TB6 | Boa imports | TB1, when Boa is next touched |

TB1 is the one that touches public surfaces in Ki.Core, Ki.KD, Ki.KS and RevDB at once; it is the one to review hardest and build first, with a throwaway sample bag, before any domain code.

---

## 8. Decisions (2026-10-21)

1. **Cheminformatics.** CDK (Java, LGPL) is accepted as the dependency of the optional `Ki.Chem.Ext` artifact. Ki.Sci parses SMILES natively and treats InChI as a validated text value; with Ki.Chem.Ext on the classpath, `mol.inchiKey` is computed and substructure and similarity search exist. CDK 2.11 modules only (core, interfaces, data, standard, smiles, valency, fingerprint, isomorphism, inchi with `jna-inchi` natives per shipped platform), never `cdk-bundle`.
2. **`Measure`** lives in Ki.Core, with its literal, as planned for RevDB v1.1; Ki.Math holds the statistics over it. This document's Ki.Math section is read with that change.
3. **The import name is `use ki.bio`** (and `import ki.bio` in Boa): the `ki.` prefix is kept.
4. **This document lives in Ki.Docs** (`docs/ki-type-bags.md`), with a copy in the RevDB project docs; the RevDB design will carry a short storage section (5.x) that references it when TB1 is built.
5. **Opaque values store both bytes and literal text** in `T_EXT`, so a database and a KD file written with a bag round-trip without it.
6. **Vectors** are a Ki.Math type with the grid's brace syntax (`.vec<Float> { 1.0 2.5 3.1 }`), not a grid of one row.
