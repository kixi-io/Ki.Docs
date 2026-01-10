---
title: Ki Types
nav_order: 4
---

# Ki Types

## Introduction

The Ki Type System (KTS) defines portable types used by [Ki Declarative (KD)](Ki-Data-(KD)), including
[KD Schema](KD-Schema), and other Ki libraries. It is a modern type system with nullable and non-nullable types,
generics and units of measure. KTS was modeled after type systems in languages such as Kotlin, Swift and C#.

KTS currently supports 27 literal types including data structures with type parameters:

### Base Types

| Type | Description | Examples | Notes |
| ---- | ----------- | -------- | ----- |
| String | Unicode text | "hello", "ท้องฟ้า", "대양" | See [String](#String) |
| Char | Unicode character | 'A', 'Ж', '桜' | |
| Int | 32b signed integer | 123, 424_235_412, 0xFF | Optional underscores for legibility |
| Long | 64b signed integer | 123L, 424_235_412L | |
| Float | 32b signed float | 123.43F, 123.43f | |
| Double | 64b signed float | 123.43, 123.43D, 123d, 5.421_523 | Optional underscores for legibility |
| Dec | 128b+ signed decimal | 123.44BD, 123.44bd | See notes below<sup>*</sup> |
| Bool | boolean | true, false, on, off | |
| URL | URL | https://www.nasa.gov, ldap://ds.example.com:389 | |
| Date | simple date | 2005/12/05, 1592/4/8 | See [Date](#Date) |
| LocalDateTime | local date & time | 2021/12/05@05:21:23.53 | See [LocalDateTime](#LocalDateTime) |
| ZonedDateTime | DateTime w/ zone | 2021/12/05@05:21:23.53+03 | See [ZonedDateTime](#ZonedDateTime) |
| KiTZDateTime | DateTime w/ named zone | 2021/12/05@05:21:23.53-US/PST | See [KiTZDateTime](#KiTZDateTime) |
| Duration | length of time |  12:74:42, 23days:05:21:23.53, 85ms | See [Duration](#Duration) |
| Version | Version descriptor | 4.2.5, 5.2-beta, 2.8.5-alpha-2 | See [Version](#Version) |
| Blob | Bytes encoded in base64 | .blob(sdf789GSfsb2+3324sf2) | See [Blob](#Blob) |
| GeoPoint | GPS coordinates | .geo(37.7749, -122.4194) | See [GeoPoint](#GeoPoint) |
| Email | Email address | kai@rockets.com, foo.noa@reptilezoo.org | |
| Grid<T\> | 2D Grid | .grid(1 2 3; 40 50 60), .grid\<Int\>(3 6 9) | See [Grid](#Grid) |
| Coordinate | Grid location | .coordinate(x=2, y=5), .coordinate(c="A", r=10) | See [Grid & Coordinate](#Grid) |
| Quantity\<U\> | An amount in Units | 23cm, -15mL, 2.5m³, 5.5e(-7)m | see [Quantity](#Quantity) |
| Range\<T\> | Range for comparables | 2..5, 2.0..<3.5, 2.0<.._ | See [Range](#Range) |
| List\<T\> | Ordered list | [1 2 3], [4, 5, 6], ['a' 'b' 'c'] | Commas optional; See [List](#List) |
| Map\<K, V\>  | Set of key=value pairs |  [name="Rufus" color="rust"], ['a'=1, 'c'=3] | Commas optional; See [Map](#Map) |
| Call | Function call | rgb(120, 140, 20, alpha=.5) | Commas optional; See [Call](#Call) | 
| Snip | Embed content | .snip(../myFile.kd), .snip("https:/kixi.io/KDExample.kd") | See [Snip](#Snip) |
| nil | Absence of a value | nil, null | |

_<sup>*</sup>For platforms that do not support this level of precision, the implementation should provide a conformant number type.

### Super Types

| Type | Description | Base Types |
| --- | --- | --- |
| Number | A scalar | Int, Long, Float, Double, Decimal |
| Any | Any value | All |

Suffix a ? make a base or supertype nullable. For example: Int? means "Int or nil"

> :bulb: Supertypes are used by libraries that interact with KTS such as KD Schema (KDS). 

<a name="String"></a>
## String

A `String` represents [unicode](https://home.unicode.org/) text. KTS Strings can contain
any unicode character. External representations should always use
[UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoding.

Strings support the following escape characters:

| Char | Description |
| ---- | ----------- |
| \t | Tab |
| \b | Backspace |
| \n | New Line |
| \r | Carriage Return |
| \u | Unicode Escape |
| \' | Single Quote |
| \" | Double Quote |
| \\ | Backslash |

<a name="DateTime"></a>
## DateTime Literals
KTS supports three DateTime types. `LocalDateTime`, `ZonedDateTime` and `KiTZDateTime` provide nanosecond
accuracy.

| Type | Description |
| ---- | ----------- |
| Date | A simple date with year/month/day |
| LocalDateTime | A local (relative) DateTime with year/month/day@hour:minute(:second(.fractional_second)?)? |
| ZonedDateTime | A DateTime with a time zone offset (see below)|
| KiTZDateTime | A DateTime with a named time zone (see below)|

_Note: A time-of-day that isn't attached to a date can be represented by using a
[Duration](#Duration) treated as an offset from midnight._

<a name="Date"></a>
### Date
A Ki `Date` type is a calendar date with a year, month and day (no time component).

The canonical representation is year/month/day. Examples:
* 2005/12/05
* 2020/09/18
* 2020/9/18

<a name="LocalDateTime"></a>
### LocalDateTime
`LocalDateTime` literals don't specify a time zone. They are to be interpreted as local
(relative to the observer) time.

    Canonical Form: yyyy/mm/dd @hh:mm(:ss(.fractional)?)?

| Example  | Description |
| ------------- | ------------- |
| 2005/09/05 @05:08:03.532 | Date and time with all components zero padded |

<a name="ZonedDateTime"></a>
### ZonedDateTime
`ZonedDateTime` literals specify a date, time and zone. Zones can be specified with -Z,
-UTC, [+/-]offset, or -[KiTZ](https://github.com/kixi-io/Ki.Docs/wiki/Ki-Time-Zone-Specification):

    Canonical Form: yyyy/mm/dd @hh:mm(:ss(.fractional)?)?(-Z | [+/-]offset)

| Example  | Description |
| ------------- | ------------- |
| 2005/12/05@05:21:23.532-Z | UTC (no offset) | |
| 2005/12/05@05:21:23.532+2 | UTC+02 offset |
| 2005/12/05@05:21:23.532-2 | UTC-02 offset |
| 2005/12/05@05:21:23.532+2:30 | UTC+02:30 offset |

<a name="KiTZDateTime"></a>
### KiTZDateTime
KiTZDateTime literals specify a date, time and KiTZ (named time zone). Zones can be specified with -[KiTZ](https://github.com/kixi-io/Ki.Docs/wiki/Ki-Time-Zone-Specification):

    Format: yyyy/mm/dd @hh:mm(:ss(.fractional)?)-KiTZ

| Example  | Description |
| ------------- | ------------- |
| 2005/12/05@05:21:23.532-JP/JST | Ki Time Zone (KiTZ) `JP/JST`: Japan Standard Time  |
| 2005/12/05@05:21:23.532-US/PST | Ki Time Zone (KiTZ) `US/PST`: US Pacific Standard Time  |
| 2005/12/05@05:21:23.532-DE/CET | Ki Time Zone (KiTZ) `DE/CET`: Germany Central European Time |

<a name="Duration"></a>
### Duration

KTS Duration literals represent a length of time, which may be negative. Duration literals
are useful for expressing the duration of an event, intervals, or chronological distances
from a reference point.

#### Canonical Form Examples
| Example                   | Description                                               |
| ------------------------- | --------------------------------------------------------- |
| 03:00:00                  | 3 hours |
| 00:12:00                  | 12 minutes |
| 00:00:42                  | 42 seconds | 
| 00:12:32.423              | 12 minutes, 32 seconds, 423 milliseconds |
| 00:01:32.000002584        | 12 minutes, 32 seconds, 2,584 nanoseconds |
| 30days:15:23:04.023       | 30 days, 15 hours, 23 mins, 4 secs, 23 milliseconds |

<a name="Version"></a>
## Version

The KTS Version type uses a simple schema based on
[Semantic Versioning 2](https://semver.org). All numeric components must be zero or
positive integers. 

    Format: `major('.'minor('.'micro)?)?('-'qualifier(('-')?qualiferNumber)?)?`

#### Version Components
| Component | Role | Description |
| --------- | ---- |----------- |
| major | Major feature release, possible breaking changes | positive integer |
| minor | Minor feature release, no breaking changes | positive integer |
| micro | Bug fixes and performance enhancement | positive integer |
| qualifier | Text label, e.g. "alpha", "beta" or "RC" | Unicode string |
| qualifierNumber | A positive integer (requires a qualifier) | positive integer |

#### Comparing Versions
For the purpose of comparison and inclusion in ranges, the sort order compares numeric
components and qualifier, if present, ignoring case. Versions that have qualifiers are
sorted below versions that are otherwise equal without a qualifier (e.g. 5.2-alpha is
lower than 5.2). Qualifiers are sorted by alphabetical ordering (case insensitive), and
finally the qualifier number, if present, is compared.

#### Canonical Form Examples
1. 5.2.7
2. 5.0.0-beta-2
3. 5.2.0-alpha
4. 5.2.7-rc

<a name="GeoPoint"></a>
### GeoPoint

GeoPoint represents geographic coordinates (GPS location) on Earth. It stores latitude, longitude, and an optional altitude with high precision using Dec (BigDecimal).

    Format: .geo(latitude, longitude(, altitude)?)

#### Coordinate System
- **Latitude**: -90.0 to +90.0 degrees (south to north)
- **Longitude**: -180.0 to +180.0 degrees (west to east)  
- **Altitude**: Optional, in meters above WGS84 ellipsoid

#### GeoPoint Examples
| Example | Description |
| ------- | ----------- |
| .geo(37.7749, -122.4194) | San Francisco (latitude, longitude) |
| .geo(35.6762, 139.6503, 40.0) | Tokyo with altitude (40 meters) |
| .geo(-33.8688, 151.2093) | Sydney |
| .geo(0.0, 0.0) | Null Island (origin) |
| .geo(51.5074, -0.1278) | London |
| .geo(48.8566, 2.3522) | Paris |

#### GeoPoint Precision
Coordinates are stored as Dec (BigDecimal) for maximum precision. The default formatting uses 6 decimal places, which provides approximately 0.1 meter accuracy.

#### GeoPoint in KD Tags
```
# Location-based data
office "Headquarters" location=.geo(37.7749, -122.4194)

# Route with waypoints
route name="City Tour" {
    waypoint "Start" coords=.geo(37.7749, -122.4194)
    waypoint "Golden Gate" coords=.geo(37.8199, -122.4783)
    waypoint "Alcatraz View" coords=.geo(37.8267, -122.4233, 15.0)
}

# Geo-fenced regions
regions {
    region "Downtown" center=.geo(37.7849, -122.4094) radius=2.5km
    region "Airport" center=.geo(37.6213, -122.3790) radius=5km
}
```

#### GeoPoint API Methods
The Ki API provides several utility methods for GeoPoint:

```kotlin
val sf = GeoPoint.of(37.7749, -122.4194)
val tokyo = GeoPoint.of(35.6762, 139.6503, 40.0)

// Distance calculation (Haversine formula)
val distance = sf.distanceTo(tokyo)  // Returns distance in kilometers

// Bearing calculation
val bearing = sf.bearingTo(tokyo)    // Returns bearing in degrees (0-360)

// Destination from point
val dest = sf.destination(100.0, 45.0)  // 100km at 45° bearing

// Hemisphere checks
sf.isNorthern   // true
sf.isWestern    // true
tokyo.isEastern // true

// Altitude handling
val withAlt = sf.withAltitude(100.0)
val noAlt = tokyo.withoutAltitude()

// Geographic center of multiple points
val center = GeoPoint.center(listOf(sf, tokyo))

// Bounding box
val (sw, ne) = GeoPoint.boundingBox(listOf(sf, tokyo))

// Formatting options
sf.toString()           // .geo(37.774900, -122.419400)
sf.toCompactString()    // .geo(37.7749, -122.4194)
sf.toDecimalDegrees()   // 37.774900°N, 122.419400°W
sf.toDMS()              // 37°46'29.6"N, 122°25'9.8"W
```

<a name="Quantity"></a>
## Quantity

A Quantity is an amount of a Unit. KD supports all SI base axes and several popular derived axes:

**SI Base Axes**
1. Time (via Duration)
2. Length 
3. Mass
4. Temperature (Celsius &amp; Kelvin)
5. SubstanceAmount
6. Current
7. Luminosity

**SI Derived Axes**
1. Area
2. Volume
3. Speed
4. Density

*Other derived axes are likely to be added in future versions.*

You specify them using standard
[SI symbols](https://physics.nist.gov/cuu/Units/units.html). There is an exception for
Liter (L). Due to a conflict with the "L" suffix for long literals, liter is written with
"LT" or "ℓ" (e.g. 5LT or 5ℓ).

By default Quantities use the high precision Dec type for their value. You can override
this behavior by adding a :i, :L, :d or :f suffix after the unit type for
Int, Long, Double or Float.

#### Canonical Form Examples
1. 23cm
2. 51.4m3 or 51.4m³
3. 97LT or 97ℓ
4. 1000kg or 1_000kg
5. 142.24km:d # This forces the value to be stored as a Double rather than a Dec

<a name="Range"></a>
### Range

Ranges can be created for all KTS base types (numbers, chars, Strings, etc.) They can be
inclusive or exclusive on either side. They can also be open on either side. Here are some
examples:

| Example              | Description                                                 |
| -------------------- | ----------------------------------------------------------- |
| 1..5                 | `Range<Int>` 1 - 5 (inclusive)                              |
| 5.0<..<15.0          | `Range<Double>` > 5, < 15 (exclusive)                       |
| 2<..17               | `Range<Int>` > 2, <= 17 (exclusive-left)                    |
| 6..<12               | `Range<Int>` >= 6, < 12, (exclusive-right)                  |
| 6.._                 | `Range<Int>` >= 6  (open-right)                             |
| _..100               | `Range<Int>` <= 100 or less (open-left)                     |
| 8<.._                | `Range<Int>` > 8, (exclusive-left, open-right)              |
| 'a'..'f'             | `Range<Char>` >= 'a', <='f' (inclusive)                     |
| 7.2.5-beta-2.._      | `Range<Version>` >= 7.2.5-beta-2 (inclusive, open-right)    |

<a name="List"></a>
### List

KD Lists are written in square brackets with space and/or comma separated values. Examples:
* `[1, 2, 3, 4] # List of Ints`
* `[5 6 7 8] # Commas are optional`
* `['a' 'b' 'c'] # List of Chars`
* `[[1 2] [3 4]] # List of Lists`
* `friends ["Pedro", "Rika", "Naisha"] type="closest" # Tag with a List value and an attribute`

<a name="Map"></a>
### Map

KD maps are written in square brackets with entry pairs separated by `=`. Examples:
* `[Spanish="hola", Fijian="Bula"] # Keys using naked (quoteless) strings`
* `[Spanish="hola" Fijian="Bula"] # Commas are optional.`
* `['a'="Ant", 'b'="Bird"] # Keys and values can be any KD type, including Lists and Maps.`

<a name="Blob"></a>
###  Blob

A KTS `Blob` is a byte array, normally represented as a byte array or encoded with 
[Base64](https://en.wikipedia.org/wiki/Base64) encoding.

Canonical Form: .blob(base64_chars)

<a name="Call"></a>
### Call

A KD `Call` is a data representations of a function invocation. It supports positional and named parameters. Examples:
* `rgb(200, 100, 120)`
* `rgb(200 100 120) # Commas are optional.`
* `rgb(200, 100, 120, alpha=.5) # positional and named parameters`

<a name="Snip"></a>
### Snip

This update adds new types and literals. Most notably, the `.snip(path)` literal. Snip allows KD files to import content from local files or URLs (with security precautions). The imported file's root will replace the `.snip(path)` with the imported content.

We also improved the exception handling and classes. Finally, we ensured there are no known bugs. Between Ki.Core-JVM and Ki.KD-JVM, which are the reference libraries for all compliant implementations, **we have over 1,400 unit and integration tests**.

---

 A Snip is a reference to an external KD document that gets expanded inline during parsing.
 
 Snips enable modular KD documents by allowing one document to include content from another.
 When a KD document containing a snip is parsed, the referenced file is loaded and its
 root tag is inserted in place of the snip.
 
 ## Ki Literal Format
 ```
 .snip(path/to/file)              # Relative path (auto-appends .kd)
 .snip(../components/button)      # Parent directory reference
 .snip("path with spaces/file")   # Quoted for special characters
 .snip(config.yaml)               # Explicit extension (no .kd added)
 .snip(https://example.com/comp)  # URL reference
 .snip(../toggles, expand=true)   # Insert only children of root tag
 ```
 
 ## Path Resolution
 - Paths are always relative to the containing KD file
 - Use forward slashes (`/`) regardless of platform
 - The `.kd` extension is auto-appended if no extension is present
 - Absolute paths are supported but may pose security risks
 
 ## Unquoted Path Characters
 Unquoted paths may only contain:
 - Alphanumeric characters (a-z, A-Z, 0-9)
 - Forward slash (`/`)
 - Hyphen (`-`)
 - Underscore (`_`)
 - Period (`.`)
 - Colon (`:`) - for URL schemes like `https://`
 
 Paths containing spaces, parentheses, or other special characters must be quoted.
 
 ## Expand Mode
 By default, the entire root tag from the snipped file is inserted. With `expand=true`,
 only the children of the root tag are inserted (the root tag itself is discarded).
 
 ## Example
 **File: main.kd**
 ```
 app {
     header {
         .snip(components/navbar)
     }
     content {
         .snip(pages/home, expand=true)
     }
 }
 ```
 
 **File: components/navbar.kd**
 ```
 navbar {
     logo "MyApp"
     menu { item "Home"; item "About" }
 }
 ```
 
 **Result after expansion:**
 ```
 app {
     header {
         navbar {
             logo "MyApp"
             menu { item "Home"; item "About" }
         }
     }
     content {
         # Children of pages/home.kd root inserted here
     }
 }
 ```

