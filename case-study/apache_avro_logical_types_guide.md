# Apache Avro Logical Types - Complete Guide

## What Are Logical Types?

**Logical types** are a way to add semantic meaning to Avro's primitive types. They allow you to represent specialized data types (like dates, timestamps, decimals) using the underlying primitive binary format, while giving code generators hints about how to interpret the data.

### The Concept

```
Primitive Type (storage) + Logical Type (interpretation) = Meaningful Data
```

**Example:**
```json
{
  "type": "long",              // Stored as 64-bit integer
  "logicalType": "timestamp-millis"  // Interpreted as milliseconds since epoch
}
```

The data is **stored** as a `long` (8 bytes), but **interpreted** as a timestamp.

---

## Why Use Logical Types?

### Without Logical Types

```json
{
  "name": "birthDate",
  "type": "long",
  "doc": "Birth date as milliseconds since epoch"
}
```

**Problems:**
- No type safety - could be any long value
- Generators produce `long` in code - not `Date` or `Instant`
- Developer must remember the convention
- Easy to make mistakes (seconds vs milliseconds)

### With Logical Types

```json
{
  "name": "birthDate",
  "type": "long",
  "logicalType": "timestamp-millis"
}
```

**Benefits:**
- Clear semantic meaning
- Code generators can produce proper types (`java.time.Instant`, `datetime`)
- Type safety in generated code
- Self-documenting schema

---

## Standard Logical Types

Avro defines several standard logical types:

### 1. Decimal (Arbitrary Precision)

**Purpose:** Exact decimal numbers (financial calculations, measurements)

**Storage:** `bytes` or `fixed`

**Attributes:**
- `precision`: Total number of digits
- `scale`: Number of digits after decimal point

**Schema:**
```json
{
  "name": "amount",
  "type": "bytes",
  "logicalType": "decimal",
  "precision": 10,
  "scale": 2
}
```

**Example values:**
- `12345.67` (precision=7, scale=2)
- `999999.99` (precision=8, scale=2)

**Why use it:**
- Exact precision (no floating-point errors)
- Essential for money/financial data
- Better than `float`/`double` for measurements

**Code generation:**
```java
// Java
private BigDecimal amount;

// Python
amount: Decimal
```

**Alternative with fixed:**
```json
{
  "name": "amount",
  "type": {
    "type": "fixed",
    "name": "DecimalAmount",
    "size": 8
  },
  "logicalType": "decimal",
  "precision": 10,
  "scale": 2
}
```

### 2. Date (Days Since Epoch)

**Purpose:** Calendar dates (no time component)

**Storage:** `int` (days since Unix epoch: 1970-01-01)

**Schema:**
```json
{
  "name": "birthDate",
  "type": "int",
  "logicalType": "date"
}
```

**Example values:**
- `0` = 1970-01-01
- `19723` = 2024-01-01
- `-1` = 1969-12-31

**Why use it:**
- Compact (4 bytes)
- Natural for dates without time
- No timezone issues
- Easy date arithmetic (add/subtract days)

**Code generation:**
```java
// Java
private java.time.LocalDate birthDate;

// Python
birthDate: datetime.date
```

### 3. Time (Millisecond Precision)

**Purpose:** Time of day (no date component)

**Storage:** `int` (milliseconds since midnight)

**Schema:**
```json
{
  "name": "alarmTime",
  "type": "int",
  "logicalType": "time-millis"
}
```

**Example values:**
- `0` = 00:00:00.000 (midnight)
- `3661000` = 01:01:01.000
- `86399999` = 23:59:59.999

**Why use it:**
- Compact (4 bytes)
- Millisecond precision sufficient for most use cases
- Natural for "time of day" without date

**Code generation:**
```java
// Java
private java.time.LocalTime alarmTime;

// Python
alarmTime: datetime.time
```

### 4. Time (Microsecond Precision)

**Purpose:** Time of day with microsecond precision

**Storage:** `long` (microseconds since midnight)

**Schema:**
```json
{
  "name": "preciseTime",
  "type": "long",
  "logicalType": "time-micros"
}
```

**Example values:**
- `0` = 00:00:00.000000
- `3661000000` = 01:01:01.000000
- `86399999999` = 23:59:59.999999

**Why use it:**
- Higher precision than time-millis
- Needed for high-frequency trading, scientific data
- 8 bytes (vs 4 for time-millis)

### 5. Timestamp (Millisecond Precision)

**Purpose:** Instant in time (date + time)

**Storage:** `long` (milliseconds since Unix epoch: 1970-01-01T00:00:00.000Z)

**Schema:**
```json
{
  "name": "createdAt",
  "type": "long",
  "logicalType": "timestamp-millis"
}
```

**Example values:**
- `0` = 1970-01-01T00:00:00.000Z
- `1704067200000` = 2024-01-01T00:00:00.000Z
- `1735689600000` = 2025-01-01T00:00:00.000Z

**Why use it:**
- Most common timestamp type
- Millisecond precision sufficient for most applications
- Compact (8 bytes)
- Easy arithmetic (add/subtract milliseconds)
- Universal - works across all timezones

**Code generation:**
```java
// Java
private java.time.Instant createdAt;

// Python
createdAt: datetime.datetime
```

**This is what you COULD have used instead of ISO 8601 strings!**

### 6. Timestamp (Microsecond Precision)

**Purpose:** Instant in time with microsecond precision

**Storage:** `long` (microseconds since Unix epoch)

**Schema:**
```json
{
  "name": "preciseTimestamp",
  "type": "long",
  "logicalType": "timestamp-micros"
}
```

**Example values:**
- `0` = 1970-01-01T00:00:00.000000Z
- `1704067200000000` = 2024-01-01T00:00:00.000000Z

**Why use it:**
- Higher precision than timestamp-millis
- Needed for financial systems, scientific applications
- Still 8 bytes (same as timestamp-millis)

### 7. Duration (Fixed Duration)

**Purpose:** Fixed-length time duration

**Storage:** `fixed` of size 12 (3 x 4-byte integers)

**Format:** 
- 4 bytes: months (unsigned)
- 4 bytes: days (unsigned)
- 4 bytes: milliseconds (unsigned)

**Schema:**
```json
{
  "name": "elapsed",
  "type": {
    "type": "fixed",
    "name": "Duration",
    "size": 12
  },
  "logicalType": "duration"
}
```

**Why use it:**
- Represents calendar-aware durations
- Accounts for variable month/year lengths
- Less common - most use timestamp arithmetic instead

### 8. UUID

**Purpose:** Universally unique identifier

**Storage:** `string` (36-character string) or `fixed` of size 16 (binary)

**Schema (as string):**
```json
{
  "name": "id",
  "type": "string",
  "logicalType": "uuid"
}
```

**Schema (as fixed - more efficient):**
```json
{
  "name": "id",
  "type": {
    "type": "fixed",
    "name": "UUID",
    "size": 16
  },
  "logicalType": "uuid"
}
```

**Example value (string format):**
```
"550e8400-e29b-41d4-a716-446655440000"
```

**Why use it:**
- Type safety for UUIDs
- Code generators produce UUID types
- Fixed format is more compact (16 vs 36 bytes)

**Code generation:**
```java
// Java
private java.util.UUID id;

// Python
id: uuid.UUID
```

---

## Logical Types Summary Table

| Logical Type | Base Type | Size | Precision | Use Case | Example |
|--------------|-----------|------|-----------|----------|---------|
| `decimal` | `bytes` or `fixed` | Variable | Configurable | Money, exact math | 12345.67 |
| `date` | `int` | 4 bytes | Days | Calendar dates | 2024-01-01 |
| `time-millis` | `int` | 4 bytes | Milliseconds | Time of day | 13:45:30.500 |
| `time-micros` | `long` | 8 bytes | Microseconds | Precise time | 13:45:30.500123 |
| `timestamp-millis` | `long` | 8 bytes | Milliseconds | Timestamps | 2024-01-01T13:45:30.500Z |
| `timestamp-micros` | `long` | 8 bytes | Microseconds | Precise timestamps | 2024-01-01T13:45:30.500123Z |
| `duration` | `fixed[12]` | 12 bytes | Months/days/ms | Calendar durations | 1 month, 2 days, 3 hours |
| `uuid` | `string` or `fixed[16]` | 36 or 16 bytes | N/A | Unique IDs | 550e8400-e29b-... |

---

## When to Use Logical Types

### Use Logical Types When:

✓ **Financial Data**
```json
{
  "name": "price",
  "type": "bytes",
  "logicalType": "decimal",
  "precision": 10,
  "scale": 2
}
```

✓ **Timestamps/Dates**
```json
{
  "name": "eventTime",
  "type": "long",
  "logicalType": "timestamp-millis"
}
```

✓ **UUIDs**
```json
{
  "name": "correlationId",
  "type": "string",
  "logicalType": "uuid"
}
```

✓ **Dates Without Time**
```json
{
  "name": "birthDate",
  "type": "int",
  "logicalType": "date"
}
```

### Don't Use Logical Types When:

✗ **You need human-readable timestamps** → Use ISO 8601 strings instead
```json
{
  "name": "atTime",
  "type": "string",
  "doc": "ISO 8601 timestamp: 2024-01-01T00:00:00Z"
}
```

✗ **Backward compatibility with systems expecting raw types**
```json
{
  "name": "timestamp",
  "type": "long",
  "doc": "Milliseconds since epoch (no logical type for legacy compatibility)"
}
```

✗ **Simple measurements that don't need type safety**
```json
{
  "name": "temperature",
  "type": "float",
  "doc": "Temperature in Celsius"
}
```

---

## Your Use Case: ISO 8601 vs timestamp-millis

You chose ISO 8601 strings for `atTime`. Let's compare:

### Option A: ISO 8601 String (Your Current Choice)

```json
{
  "name": "atTime",
  "type": "string",
  "doc": "ISO 8601 timestamp: 2024-01-01T00:00:00Z"
}
```

**Data:**
```json
"atTime": "2024-01-01T00:00:00Z"
```

**Pros:**
- Human-readable
- Includes timezone info
- Variable precision (seconds, millis, micros)
- Easy debugging
- Language-agnostic parsing

**Cons:**
- Larger size (~24 bytes vs 8 bytes)
- Parsing overhead
- No automatic type generation
- String validation needed

### Option B: timestamp-millis Logical Type

```json
{
  "name": "atTime",
  "type": "long",
  "logicalType": "timestamp-millis"
}
```

**Data:**
```json
"atTime": 1704067200000
```

**Pros:**
- Compact (8 bytes)
- Fast comparison/arithmetic
- Type-safe code generation (Instant, datetime)
- No parsing needed
- Standard Avro practice

**Cons:**
- Not human-readable
- Fixed millisecond precision
- No timezone in data (assumes UTC)
- Harder to debug raw data

### Recommendation

**For our CIM use case:**

The choice of **ISO 8601 strings would be good** if:
- Human readability is important
- You need to match CIM XML/RDF conventions
- Different timezones need to be represented
- Variable precision is needed

**But `timestamp-millis` would be better** if:
- WE prioritize compact binary format
- All timestamps are UTC
- Millisecond precision is sufficient
- We want type-safe generated code

**Most Avro users choose timestamp-millis** for efficiency, but ISO 8601 is valid for interoperability.

## Code Generation Examples

### Java

**Schema with logical types:**
```json
{
  "type": "record",
  "name": "Event",
  "fields": [
    {"name": "id", "type": "string", "logicalType": "uuid"},
    {"name": "timestamp", "type": "long", "logicalType": "timestamp-millis"},
    {"name": "date", "type": "int", "logicalType": "date"},
    {"name": "amount", "type": "bytes", "logicalType": "decimal", "precision": 10, "scale": 2}
  ]
}
```

**Generated Java class:**
```java
public class Event {
    private java.util.UUID id;
    private java.time.Instant timestamp;
    private java.time.LocalDate date;
    private java.math.BigDecimal amount;
    
    // Getters/setters...
}
```

**Without logical types:**
```java
public class Event {
    private String id;           // Just a string
    private long timestamp;      // Just a long
    private int date;            // Just an int
    private ByteBuffer amount;   // Just bytes
    
    // Getters/setters...
}
```

### Python

**With logical types:**
```python
from datetime import datetime, date
from decimal import Decimal
from uuid import UUID

event = {
    'id': UUID('550e8400-e29b-41d4-a716-446655440000'),
    'timestamp': datetime(2024, 1, 1, 0, 0, 0),
    'date': date(2024, 1, 1),
    'amount': Decimal('12345.67')
}
```

**Without logical types:**
```python
event = {
    'id': '550e8400-e29b-41d4-a716-446655440000',  # String
    'timestamp': 1704067200000,                     # Long
    'date': 19723,                                  # Int
    'amount': b'\x00\x12\xd6\x87'                   # Bytes
}
```

## Custom Logical Types

You can define custom logical types (not portable across all Avro implementations):

```json
{
  "name": "ipAddress",
  "type": "string",
  "logicalType": "ipv4"
}
```

**Note:** Custom logical types are application-specific and may not be recognized by all tools.

## Best Practices

### 1. Use Standard Logical Types When Possible

- Prefer standard types (date, timestamp-millis, decimal, uuid)

- Avoid custom logical types unless necessary

### 2. Document the Logical Type Meaning

```json
{
  "name": "createdAt",
  "type": "long",
  "logicalType": "timestamp-millis",
  "doc": "Creation timestamp in UTC as milliseconds since epoch"
}
```

### 3. Choose Appropriate Precision

- Most applications: `timestamp-millis` (±292 million years)
- High-frequency systems: `timestamp-micros` (±292,000 years)
- Financial: `decimal` with appropriate scale

### 4. Consider Backwards Compatibility

Adding a logical type to an existing field is usually safe (readers ignore unknown logical types), but changing the logical type is breaking.

### 5. Test Code Generation

Verify that your code generator properly supports the logical types you use.


## Summary

### What Are Logical Types?

Semantic layer on top of primitive types that tells code generators how to interpret the data.

### When to Use Them?

- Timestamps/dates → `timestamp-millis`, `date`
- Financial/precise numbers → `decimal`
- UUIDs → `uuid`
- When you want type-safe generated code

### When NOT to Use Them?

- Human readability is priority → Use strings
- Interoperability with non-Avro systems → Use strings
- Legacy compatibility → Use raw primitives

### For Our SAR Profile Schema

We could represent as ISO 8601 strings which would also be valid, but the preferred choice utilized in our builder for this case study is:

```json
{
  "name": "atTime",
  "type": "long",
  "logicalType": "timestamp-millis"
}
```

**Both approaches are correct** but the current opinion is that the use of logical types for more nuanced semantic clarity in the context of serialization and deserialization of streamed data is preferable in the context of Apache Avro. Additionally, for more compact, type-safe timestamps like this, we believe human readability isn't critical.