---
title: KD or JSON?
nav_order: 7
---

# KD vs JSON Comparison

A comprehensive comparison of KD (Ki Data) format and JSON, highlighting the strengths and use cases of each.

---

## KD Advantages Over JSON

### 1. Rich Literal Types

KD has native support for 27 literal types that JSON must encode as strings:

| Type | KD Example | JSON Equivalent |
|------|------------|-----------------|
| Date | `2024/3/15` | `"2024-03-15"` |
| DateTime | `2024/3/15@14:30:00` | `"2024-03-15T14:30:00"` |
| Zoned DateTime | `2024/3/15@14:30:00-US/PST` | `"2024-03-15T14:30:00-08:00"` |
| Duration | `5h`, `2:30:00`, `3days` | `"PT5H"` or `18000` |
| Quantity | `5cm`, `100kg`, `9.109e(-31)kg` | `{"value": 5, "unit": "cm"}` |
| Currency | `$100`, `€50.25`, `0.5BTC` | `{"amount": 100, "currency": "USD"}` |
| URL | `<https://example.com>` | `"https://example.com"` |
| Blob | `.blob(SGVsbG8=)` | `"SGVsbG8="` |
| GeoPoint | `.geo(37.7749, -122.4194)` | `{"lat": 37.7749, "lon": -122.4194}` |
| Coordinate | `.coordinate(x=4, y=7)` | `{"x": 4, "y": 7}` |
| Grid | `.grid(1 2 3 \n 4 5 6)` | `[[1,2,3],[4,5,6]]` |
| Version | `2.1.0` | `"2.1.0"` |
| Range | `1..10`, `"a".."z"` | `{"start": 1, "end": 10}` |
| Email | `user@example.com` | `"user@example.com"` |
| Char | `'a'` | `"a"` |

### 2. Document Structure

```kd
# KD: Named tags with namespaces and annotations
@deprecated
@since(2.0)
ui:button name="Submit" enabled=true {
    icon "check"
    tooltip "Click to submit"
}
```

```json
// JSON: Requires verbose object nesting
{
  "_annotations": ["deprecated", {"since": "2.0"}],
  "_namespace": "ui",
  "_tag": "button",
  "name": "Submit",
  "enabled": true,
  "children": [
    {"_tag": "icon", "value": "check"},
    {"_tag": "tooltip", "value": "Click to submit"}
  ]
}
```

### 3. Human Readability

| Feature | KD | JSON |
|---------|-----|------|
| Quotes on keys | Not required | Required |
| Commas between elements | Not required | Required |
| Comments | ✅ `# comment` and `/* block */` | ❌ Not allowed |
| Trailing commas | N/A | ❌ Syntax error |
| Number readability | `1_000_000` | `1000000` |

**Example - Configuration file:**

```kd
# KD Configuration
server {
    host "localhost"
    port 8080
    timeout 30s
    
    # Database settings
    database {
        url <jdbc:postgresql://localhost/mydb>
        pool_size 10
        max_wait 5s
    }
}
```

```json
{
  "server": {
    "host": "localhost",
    "port": 8080,
    "timeout": "30s",
    "database": {
      "url": "jdbc:postgresql://localhost/mydb",
      "pool_size": 10,
      "max_wait": "5s"
    }
  }
}
```

### 4. Modularity (Snip System)

KD supports modular document composition:

```kd
# main.kd
app {
    .snip(shared/database)        # Include entire file
    .snip(shared/logging)
    .snip(pages/home, expand=true) # Include only children
    
    server { port 8080 }
}
```

Features:
- ✅ Relative path resolution
- ✅ Circular reference detection
- ✅ Caching of resolved snips
- ✅ Security constraints (URL restrictions, path limits)
- ✅ Configurable nesting depth

### 5. Type Safety & Precision

```kd
# Explicit numeric types
count 100        # Int (default)
bigCount 100L    # Long
price 99.99bd    # BigDecimal (arbitrary precision)
ratio 3.14f      # Float
rate 2.718d      # Double

# Scientific notation with units
electron_mass 9.109e(-31)kg
earth_sun_dist 1.496e(8)km

# Unit conversion built-in
distance 100cm   # Can convert to: 1000mm, 1m, 0.001km
```

### 6. Additional Features

- **Namespaced identifiers**: `html:div`, `svg:rect`
- **Annotations**: `@readonly @validate(min=0, max=100)`
- **Anonymous tags**: Clean config-file syntax
- **Grids**: Native 2D data support with row validation

---

## JSON Advantages Over KD

### 1. Ubiquity & Ecosystem

| Aspect | JSON | KD |
|--------|------|-----|
| Language support | Every language | Kotlin/JVM (primary) |
| Browser native | ✅ `JSON.parse()` | ❌ Requires library |
| Database support | MongoDB, PostgreSQL, MySQL | Limited |
| API standards | REST, GraphQL, OpenAPI | Emerging |
| IDE support | Universal | Limited |

### 2. Simplicity

JSON has only **6 data types**:
- `string`
- `number`
- `boolean`
- `null`
- `array`
- `object`

This means:
- ✅ Minimal learning curve
- ✅ Easy to implement parsers

Note: KD.Light is underway. It will address the issues mentioned above when a lean version of KD is required (e.g. embedded code).

### 3. Standardization

| Standard | Description |
|----------|-------------|
| [RFC 8259](https://tools.ietf.org/html/rfc8259) | JSON specification |
| [JSON Schema](https://json-schema.org/) | Validation & documentation - Note: KD Schema is underway. |
| [JSON Patch](https://tools.ietf.org/html/rfc6902) | Document modification |
| [JSON Pointer](https://tools.ietf.org/html/rfc6901) | Value addressing |
| [JSON-LD](https://json-ld.org/) | Linked data |


### 5. Performance

- **Smaller parser implementations**: ~100 lines vs ~2000+ for KD
- **Faster simple parsing**: No date/unit/type inference
- **Mature streaming parsers**: Jackson, Gson, simdjson
- **SIMD optimizations**: simdjson achieves GB/s parsing

### 6. Tooling

| Tool Category | JSON Options |
|---------------|--------------|
| Validators | jsonlint, ajv, json-schema-validator |
| Formatters | jq, prettier, json_pp |
| Diff tools | json-diff, deepdiff |
| Query languages | jq, JSONPath, JMESPath |
| Editors | VS Code, WebStorm, any text editor |
| Online tools | jsonformatter.org, jsoneditoronline.org |

---

## Summary Comparison

| Feature | KD | JSON |
|---------|:--:|:----:|
| **Literal types** | 23+ | 6 |
| **Dates/times** | Native | String |
| **Units/quantities** | Native | Not supported |
| **Comments** | ✅ | ❌ |
| **Modularity (snips)** | ✅ | ❌ |
| **Namespaces** | ✅ | ❌ |
| **Annotations** | ✅ | ❌ |
| **Ecosystem maturity** | Emerging | Universal |
| **Browser native** | ❌ | ✅ |
| **Schema validation** | Limited | JSON Schema |
| **Streaming parsers** | Limited | Mature |
| **Human editing** | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Machine parsing** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Learning curve** | Moderate | Minimal |

---

## When to Use Each

### Favor KD When:
- 📝 Human readability is a priority (complex data, config files, documentation)
- 📅 Working with dates, times, durations extensively
- 📏 Scientific/engineering data with units
- 💰 Financial data with currencies
- ◽️ Space efficiency without sacrificing readability is important
- 🧩 Modular document composition needed
- 🏷️ Semantic markup with annotations/namespaces
- 📊 Grid/tabular data is common
- 🎼 Composing documents to manage document size and complexity is important

### Favor JSON When:
- 🌐 Web APIs and browser interop required
- 🔌 Integrating with existing JSON-based systems
- 📚 Team familiarity is important. KD is stable but emerging.
- ⚡ Maximum parsing performance needed. Note: KD.Light is underway. It has a faster parser.
- 🔧 Rich tooling ecosystem required
- 📋 JSON Schema validation needed - Note: KD Schema is underway
- 🗄️ Storing in JSON-native databases

---

## References

- [KD Specification](Ki-Data-%28KD%29)
- [KD Repos](https://github.com/kixi-io)
- [JSON Specification (RFC 8259)](https://tools.ietf.org/html/rfc8259)
- [JSON Schema](https://json-schema.org/)