# AVRO Handling of CIM Compound Types

## What Are Compound Types in CIM/OWL?

In a CIMTool OWL file, compound types are marked with:
```xml
<uml:hasStereotype rdf:resource="http://langdale.com.au/2005/UML#compound"/>
```

These are **value objects** that:
- Have no independent identity (e.g. no mRID)
- Are always embedded within their parent object
- Cannot exist independently
- Examples in our SAR profile: StreetAddress, StreetDetail, TownDetail

## Three Approaches Compared

### Approach 1: Inline Record (Anonymous Type)
```json
{
  "name": "Location",
  "type": "record",
  "fields": [
    {
      "name": "mainAddress",
      "type": {
        "type": "record",
        "name": "StreetAddress_inline",
        "fields": [...]
      }
    }
  ]
}
```
**Use when**: Type is only used once, never reused

---

### Approach 2: Named Types (Separate Definitions) [RECOMMENDED]
```json
[
  {
    "type": "record",
    "name": "StreetAddress",
    "fields": [...]
  },
  {
    "type": "record", 
    "name": "Location",
    "fields": [
      {
        "name": "mainAddress",
        "type": "StreetAddress"  // References the named type
      }
    ]
  }
]
```
**Use when**: Type might be reused, matches OWL structure

---

### Approach 3: By Reference (Foreign Key)
```json
{
  "name": "Location",
  "type": "record",
  "fields": [
    {
      "name": "mainAddressRef",
      "type": "string"  // Just an mRID reference
    }
  ]
}
```
**Use when**: Object has identity and exists independently. (NOT applicable for compound types)

---

## Key Decision Factors

| Factor | Inline Record | Named Types | By Reference |
|--------|--------------|-------------|--------------|
| **Reusability** | ❌ Must duplicate | ✅ Define once, use many | ✅ Reference anywhere |
| **Message Size** | 🔴 Larger (full embed) | 🔴 Larger (full embed) | 🟢 Smaller (just IDs) |
| **Self-contained** | ✅ Everything in one place | ✅ Everything in one place | ❌ Requires lookup |
| **CIM Compound Semantics** | ✅ Matches | ✅ Matches perfectly | ❌ Doesn't match |
| **Query Complexity** | 🟢 Simple | 🟢 Simple | 🔴 Requires joins |
| **Maintenance** | 🔴 Change in many places | 🟢 Change once | 🟢 Change schema separately |

---

## Practical Example

### Location Definition:
```
Location (concrete)
  ├─ direction: string [min=1]
  ├─ mainAddress: StreetAddress [min=1, compound]
      ├─ postalCode: string [optional]
      ├─ streetDetail: StreetDetail [min=1, compound]
      │   ├─ buildingName: string
      │   ├─ code: string
      │   └─ name: string
      └─ townDetail: TownDetail [optional, compound]
          ├─ code: string
          ├─ country: string
          └─ name: string
```

### Recommended AVRO (Named Types):
```json
[
  {
    "type": "record",
    "name": "StreetDetail",
    "namespace": "eu.cim4.example",
    "fields": [
      {"name": "buildingName", "type": ["null", "string"], "default": null},
      {"name": "code", "type": ["null", "string"], "default": null},
      {"name": "name", "type": ["null", "string"], "default": null}
    ]
  },
  {
    "type": "record",
    "name": "TownDetail", 
    "namespace": "eu.cim4.example",
    "fields": [
      {"name": "code", "type": ["null", "string"], "default": null},
      {"name": "country", "type": ["null", "string"], "default": null},
      {"name": "name", "type": ["null", "string"], "default": null}
    ]
  },
  {
    "type": "record",
    "name": "StreetAddress",
    "namespace": "eu.cim4.example",
    "fields": [
      {"name": "postalCode", "type": ["null", "string"], "default": null},
      {"name": "streetDetail", "type": "eu.cim4.example.StreetDetail"},
      {"name": "townDetail", "type": ["null", "eu.cim4.example.TownDetail"], "default": null}
    ]
  },
  {
    "type": "record",
    "name": "Location",
    "namespace": "eu.cim4.example",
    "fields": [
      {"name": "mRID", "type": "string"},
      {"name": "direction", "type": "string"},
      {"name": "mainAddress", "type": "eu.cim4.example.StreetAddress"}
    ]
  }
]
```

## For Optional Fields

When a field is **optional** (min cardinality = 0), we utilize a union in the Avro schema:

```json
{
    "name": "townDetail", 
    "type": ["null", "TownDetail"], 
    "default": null
}
```

This means:
- The field can be `null` OR a `TownDetail` object
- Default is `null` (field is optional)
- First type in union should be `null` for optional fields

When a field is **required** (min cardinality = 1):

```json
{
    "name": "streetDetail", 
    "type": "StreetDetail", 
}
```

This means:
- The field MUST be present
- No union with null
- Must always have a value

## Recommendation for Our Case Study

**Utilization of Approach 2 (Named Types)** because:

1. ✅ Our compound types (StreetAddress, StreetDetail, TownDetail) are true CIM compounds
2. ✅ Named types can be reused if needed elsewhere
3. ✅ Matches the OWL structure semantics
4. ✅ Standard practice in other CIM message profiles
5. ✅ Easier to maintain and evolve

**Order of Definition Matters:**
- Define leaf types first (StreetDetail, TownDetail)
- Then types that use them (StreetAddress)
- Then types that use those (Location)
- This is because AVRO requires types to be defined before they're referenced
