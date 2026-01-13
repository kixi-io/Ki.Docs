---
title: FAQ
nav_order: 3
---

# FAQ
{: .no_toc }

* TOC
{:toc}

## Q: Is KD a replacement for JSON?

It is a better option for certain purposes. JSON has many advantages. It's simple, easy to learn, easy to parse, widely adopted and available on every platform that has a JavaScript parser because it is a perfect subset of the language. In many cases it is good choice as a lingua franca data description language. That being said, KD has a number of advantages for scientific apps, finance apps, rules engines and other domains that require intricate data structures, benefit from easy expression of higher level types, or need stronger type validation.

KD is terser, more expressive and easier to read than equivalent JSON. Let's look at a simple example of a version range:

**JSON**
```
{
    "name": "widget",
    "lowerVersion": [2, 3, 1, "beta", 2],
    "upperVersion": [5, 2, 0, "RC"]
}
```
**KD**
```
widget 2.3.1-beta-2 .. 5.2.0-RC
```

Not only is the KD version easier to read, but it produces a single tag "widget" with a Range\<Version\> type for the
value. Range\<Version\> is a Ki (KD's base library) type ported by all compliant Ki implementations. They either provide
a Ki Range type or map to a suitable existing type if one is available. 

Additional advantages include:
1. Higher order portable types such as Range, Date, LocalDateTime, ZonedDateTime, Duration, Version, List and Map
2. A [modern type system](ki-types.md) with support for nullable and non-nullable types, generics and validators
3. Sophisticated, easy to use [schema](kd-schema.md)
4. Support for [Quantities](ki-data.md#quantity) with SI Units of Measure
5. Stronger validation: The parser maps literals to 20 types, each with their own validators.
6. Higher precision numbers: Ki's Decimal type is 128 bits. This is often necessary in financial and scientific domains.
7. XML-style namespaces

For more details and advice on when to use KD or JSON: [KD vs JSON](kd-or-json.md)

## Q: Is KD a replacement for XML?

No. XML is an excellent format for marking up documents, describing metadata, mixing tags in free form text, and many
other applications.

XML is also commonly used for properties files, configuration files, and object serialization. For these purposes, KD offers a terser and more perspicuous syntax. KD is type aware and allows data structures such as ranges, lists, maps, trees and grids to be easily represented.

Lets look at a snippet from a configuration file represented in both languages.

**XML**

```xml
<path id="prog.src">
    <pathelement path="${src.dir}" />
    <pathelement path="${res.dir}" />
    <pathelement path="${external.src.dir}" />
</path>

<target name="prog.tests" depends="prog.build" echo="true">
    <java classname="org.ikayzo.sdl.PackageTests" failonerror="true" dir="${basedir}/test" fork="true">
        <arg value="-d" />
        <arg value="${basedir}/test" />
    </java>
</target>
```

**KD**
The KD equivalent is shorter, more readable, and type-aware:

```text
path prog.src elements=["${src.dir}" "${res.dir}" "${external.src.dir}"]

target prog.tests depends=prog.build echo=true {
    java org.ikayzo.sdl.PackageTests failonerror=true dir="${basedir}/test" fork=true \
        args=["-d" "${basedir}/test"]
}
```

## Q: Why doesn't KD use ISO standards for types such as date/time and duration?

It does. Because clarity and readability (by humans and computers) is a design goal, KD
has its own format, but it also supports ISO-8601 representations.

### Zoned Date/Time

| [**ISO-8601**](https://en.wikipedia.org/wiki/ISO_8601) | [**KD**](ki-data.md#duration) |
| --- | --- |
| 2021-05-21T05:30:15Z | 2021/5/21 @5:30:15-Z |
| 2021-05-21T05:30:15-08:00 | 2021/5/21 @5:30-8 |
| 2021-05-21T05:30:00-08:00[America/Los_Angeles] | 2021/5/21 @5:30-US/PST |

Notes:
1. Ki ZonedDateTime's can also optionally include zero padding.
2. The space between date and time is optional.
3. Ki DateTime types have nanosecond resolution.
4. Ki ZoneIDs can be specified by offsets or short common names called [KiTZ](ki-time-zones.md), which are unambiguous thanks to the inclusion of a country code.

### Duration

| [**ISO-8601**](https://en.wikipedia.org/wiki/ISO_8601) | [**KD**](https://github.com/kixi-io/Ki.Docs/wiki/Ki.Core-JVM/) |
| --- | --- |
| PT5H | 5h |
| PT29H30M15.412S | 1day:5:30:15.412 |

Note: 
1. Ki Durations have nanosecond resolution.
2. The time unit is required for durations including a `day` component. All others are optional (i.e. `1day:5:30:15.412` or `1day:5h:30min:15.412s`)

## Q: Why isn't there some kind of header that specifies the encoding of KD files?

Because this isn't 1990. Everything in KD is [UTF-8](https://en.wikipedia.org/wiki/UTF-8). UTF-8 can encode every code point in Unicode and is used by 95+% of everything. If you aren't using UTF-8 and you aren't working on the internal representation of Strings in memory for some type of runtime system you are doing it wrong.

## Q: Can I use KD with older versions of Java (pre Java 11)?

KD-JVM uses new language features introduced in Java 11. The included binaries will only work with the Java 11 JVM or later.

## Q: Can I use KD with older versions of C#?

KD-.NET requires C# 5.0. There are currently no plans to provide support for earlier versions, but if you would like to contribute a backport, by all means, please contribute.

## Q: I have a question about KD. Who can help me?

<span style="color:rgb(20,90,20)">We are happy to help! :smile:</span> Please post questions here:

| Library | Description | Issues | Discussion |
| ------- | ----------- | ------ | ---------- |
| [KD](https://github.com/kixi-io/Ki.KD-JVM) | The KD language and library | [Issues](https://github.com/kixi-io/Ki.KD-JVM/issues) | [Discussions](https://github.com/orgs/kixi-io/discussions) |
| [Core](https://github.com/kixi-io/Ki.Core-JVM) | Ki.Types &amp; utilities | [Issues](https://github.com/kixi-io/Ki.Core-JVM/issues) | [Discussions](https://github.com/orgs/kixi-io/discussions) |

Bugs and feature requests can be posted directly to [GitHub Issues](https://github.com/kixi-io/Ki.Docs/issues).

## Q: Are the KD libraries thread-safe?

The Tag data structure is not thread-safe in any of the official implementations. In general, libraries should not make assumptions about how threads work in client apps. It is up to you to handle thread safety in a way most appropriate to your application. For Java, Kotlin and C#, if multiple threads have access to a tree of Tags it's a good idea to synchronize on the root Tag.

## Q: If tags are nested to very deep levels, doesn't the lack of a closing name cause confusion?

It is our experience that for most purposes KD doesn't need to be nested as deeply as XML (because of the value lists, anonymous tags, list and map literals, etc.) If you need deeply nested KD tags you can add a simple comment to the close bracket to avoid confusion.

```text
tag1 {
    tag2 {
        tag3 {
        }
    } # tag2
} # tag1
```

## Q: Does the order of values, attributes, or children in a Tag matter?

The order of values and children is significant. Duplicates are allowed and the order in which they are listed is maintained in the KOM (KD Object Model). The order of attributes is not significant. Only one instance of an attribute key is allowed in a given tag. 

## Q: Why doesn't KD use squiggly brackets like JSON for maps?

Because squiggly brackets are used for tag bodies. KD uses square brackets for lists and maps (à la Swift) with key/value pairs separated by `=`.

## Q: Why don't you add \<insert suggested literal type\>?

We are trying to keep KD simple by including only very commonly needed literal types. Others may be added in future versions based on demand.

## Q: How mature is KD? 

[KD for the JVM (Kotlin, Java, etc.)](https://github.com/kixi-io/Ki.KD-JVM) is in beta as of August, 2020. SDL, KD's predecessor, was released in 2005. It is in use on a number of large projects ranging from open source frameworks to systems developed for global investment banks. The project is actively maintained by multiple contributors.

## Q: Have you heard of \<insert related language\>? Why wasn't it good enough?

If it is listed in the references section of the [Language Guide](ki-data.md#references), then yes, we have heard of it. Similar declarative languages like YAML met some but not all of our design goals. YAML code such as:

```yaml
!!map {
  ? !!str "---"
  : !!str "foo",
  ? !!str "...",
  : !!str "bar"
}
```

...does not seem very natural in the modern language landscape. We feel KD's type inference from "natural" looking literals similar to those found in Swift, Java, Kotlin, Dart and C# are preferable. KD's maps (attributes) and lists look similar to analogous constructs in popular scripting languages.

Additionally, KD was designed to cleanly map to and from XML. This is not true for all the languages in our references list.

<a name="contribution"></a>

## Q: I want to contribute to KD. Who do I talk to?

<span style="color:rgb(20,90,20)">Great!</span> You can [start a discussion](https://github.com/orgs/kixi-io/discussions) in the forum, post an issue or feature request to [GitHub Issues](https://github.com/kixi-io/Ki.Docs/issues), or submit a pull request to a [repo](https://github.com/kixi-io).
