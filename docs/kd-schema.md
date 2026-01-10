---
title: KD Schema
nav_order: 5
---

# KD Schema

![KD Schema Diagram](https://raw.githubusercontent.com/kixi-io/Ki.Docs/refs/heads/master/KD_Schema_Diagram.png)

## Introduction
KD Schema (KDS) defines KD document structures. It is simple to express and easy to
use. Here is a Kotlin example of defining a schema and applying it to a document in three
lines:
```
val schema = Schema.make(KD.read("tag Insect name=String flying=Bool"))
val document = KD.read("""Insect name="ant" flying=false""")
schema.apply(document) // Throws an error if attributes are missing or types are wrong
```

KDS is expressive. This is schema for a tag with a list of SI length
measurements: `tag Lengths [Length]`, which matches `Lengths [8mm 12cm 1m]` but not 
`Lengths [2mg 4mg]`

KDS allows you to validate the structure of a tag tree and inject default values. This is
a Customer `tag` with a `name` and `premium` attribute. The premium attribute defaults
to `true`.
```
tag Customer name=String premium=true

# A Customers tag has one or more Customer children
Customers {
    Customer 0.._
}
```

This is a Kotlin example demonstrating how to validate a tag structure and inject values:
```
var schemaTag = KD.read("""
    tag Customer name=String premium=true

    # A Customers tag has one or more Customer children
    @Root tag Customers {
        Customer 0.._
    }
""")
var schema = Schema.make(schemaTag)

var doc = KD.read("""
Customers {
    # The premium attribute is optional because its default value is specified.
    Customer name="Hajime" 
    Customer name="Jason" premium=false
}
""")

schema.apply(doc)
println(doc)
```

...the output is:
```
Customers {
    Customer premium=true name="Hajime"
    Customer premium=false name="Jason"
}
```
## Types & Unit Axes
KD Schema allows you to constrain a type by providing a [KTS Type](Ki-Types), SI Unit Axis
or default value. For default values the type is inferred.

**Examples**
```
# The insect attribute is required, and must be a Bool
tag Bug insect=Boolean

# The insect attribute is required, and must be a Bool or nil
tag Bug insect=Boolean_N

# The insect attribute is optional, and defaults to true
tag Bug insect=true
```

You can specify type parameters for Ranges, Lists and Maps.

**Examples**
```
# The attribute names is a List<String>
tag Group names=[String]

# The attribute players is a Map<String, Int> (name, score)
tag Game players=[String, Int]

# The size is a range of Ints
board size=Range.Int

# The size is a range of SI unit axis Length
board size=Range.Length

# The size is a range of SI unit axis Length from 5cm to 20cm
board size=5cm..20cm
```

## Matchers & Default Types
Matchers allow you to put more specific constraints on a value or attribute. Matchers can
be regular expressions, a list of options or a range of values (including SI units). They
can also include a default value, which makes them optional.

**Examples**
```
# name must end with "widget"
tag Widget name=[regex=".* widget"]

# color must be red, green or blue (default=red, String type is inferred)
tag Widget color=[options=[red, green, blue] default=red]

# size must be 5cm to 10cm (inclusive). There is no default, so the attribute is required.
tag Widget size=[range=5cm..10cm]

# Widget tag with a name Regex and options for color and size
tag Widget name=[regex=".* widget"] color=[options=[red, green, blue] default=red] \
               size=[range=5..10 default=7]
```

## Child Tags
KD schema allows you to easily specify which child types a tag can contain, and how
many. Child tags need to be specified above the parent tag. The root of the tag hierarchy
is marked with the @Root annotation.

**Example**
```
tag Potion color=String amount=Volume

@Root
tag Potions collection="General" {
  Potion 1..0 # Must contain 1 or more Potion tags
}
```

## Tag Behavior Attributes
A tag's schema may allow any number of values (similar to varargs in many languages) by
adding the attribute `kd:varVals=Type`

**Example**

```tag Stuff kd:varVals=String # Contains any number of String values```

...which allows for tags such as:

```Stuff "apples" "hats" "lemurs"```

Similarly, a tag can allow for any attribute by using `kd:anyAtts=true`.

## Conventions
Tag names are uppercase. Attribute names are lowercase.