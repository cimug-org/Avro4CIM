# SecurityAnalysisResult Example Payloads

This directory contains example JSON payloads for the SecurityAnalysisResult Apache Avro schema. Two schema versions are available:
- **BASE schema** (`SecurityAnalysisResult.avsc`) - Simple, lightweight schema with 7 types
- **EXTENDED schema** (`SecurityAnalysisResult_EXTENDED.avsc`) - Comprehensive schema with 24 types

## Schema Structure

### BASE Schema Structure

The BASE schema uses a simplified structure:
- **Container fields**: `header` and `RemedialActionApplied` array
- **OperationalLimit**: String UUID reference
- **7 types total**: Header, 2 enums, 4 records

**Container Structure:**
```json
{
  "header": {
    "profProfile": "string",
    "identifier": "string",
    "isVersionOf": "string or null",
    "version": "string or null",
    "startDate": "string",
    "schemaRef": "string"
  },
  "RemedialActionApplied": [...]
}
```

### EXTENDED Schema Structure

The EXTENDED schema includes additional context and nested objects:
- **Container fields**: `header`, `Location`, `RemedialActionApplied`, and `Substation` arrays
- **OperationalLimit**: Nested object with detailed limit information
- **24 types total**: Includes equipment types, location types, and operational limit details

**Container Structure:**
```json
{
  "header": {...},
  "Location": [...],
  "RemedialActionApplied": [...],
  "Substation": [...]
}
```

### Common Structure Elements

Both schemas share:
- **PowerFlowResult**: Union type (not array) with cardinality 1..1
- **Union members**: BaseCasePowerFlowResult or ContingencyPowerFlowResult
- **RemedialActionApplied**: Array with minimum cardinality = 0 (schema-level), though application logic typically requires ≥1 item

## Example Files

### BASE Schema Examples

#### 01_minimal_valid.json
**Purpose:** Minimal valid message structure for BASE schema

**Content:**
- Required `header` field with all mandatory header fields
- Empty `RemedialActionApplied` array

**Note:** While this is structurally valid Avro, it represents an empty result set. Application-level logic may require at least 1 RemedialActionApplied item.

**Structure:**
```json
{
  "header": {
    "profProfile": "https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4",
    "identifier": "sar-minimal-base-001",
    "isVersionOf": null,
    "version": null,
    "startDate": "2024-01-01T00:00:00Z",
    "schemaRef": "https://schemas.cim4.eu/avro/SecurityAnalysisResult/2.4.0"
  },
  "RemedialActionApplied": []
}
```

---

#### 02_single_remedial_action.json
**Purpose:** Single remedial action with base case power flow result

**Content:**
- Complete `header` with metadata
- 1 RemedialActionApplied record
- PowerFlowResult uses BaseCasePowerFlowResult type
- Demonstrates proper UUID format for all references
- Shows ENTSO-E EIC code for Region
- OperationalLimit as string UUID (BASE schema format)

**Key Features:**
- Base case result (no contingency)
- Current measurement (valueA = 850.0 A)
- Non-violation scenario (85% loading)
- Proper Avro union JSON encoding

---

#### 03_complete_analysis.json
**Purpose:** Complete security analysis with multiple remedial actions and scenarios

**Content:**
- Complete `header` with versioning information
- 4 RemedialActionApplied records showing:
  1. Base case - normal operation (85% loading)
  2. Base case - violation (105% loading)
  3. Contingency - critical violation (145% loading)
  4. Contingency with stage - resolved (92% loading)

**Scenario:**
Comprehensive analysis demonstrating:
- Base case violations
- Contingency analysis
- Remedial action staging (StageForRemedialActionScheme)
- Multiple regions (Germany, France)
- Both union types (BaseCasePowerFlowResult and ContingencyPowerFlowResult)
- OperationalLimit as string UUID references

---

#### 04_multi_region_union_demo.json
**Purpose:** Demonstrates Avro union type encoding with cross-border analysis

**Content:**
- Complete `header` with cross-border analysis metadata
- 2 RemedialActionApplied records
- First: PowerFlowResult as BaseCasePowerFlowResult (preventive action)
- Second: PowerFlowResult as ContingencyPowerFlowResult (curative action with stage)

**Avro Union JSON Format:**
```json
"PowerFlowResult": {
  "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult": {
    /* field values */
  }
}
```

The fully qualified type name acts as the discriminator for union types.

**Scenario:**
- Preventive: FR region at 1850 MW (74% loading)
- Curative: DE region at 2400 MW (96% loading after contingency)

**Key Features:**
- Demonstrates both union members
- Shows preventive vs curative actions
- Cross-border multi-region analysis
- ENTSO-E EIC codes for regions
- Alternative measurements (valueW instead of valueA)

---

### EXTENDED Schema Examples

The EXTENDED schema includes all BASE schema features plus additional context. All EXTENDED examples use the `_EXTENDED.json` suffix.

#### 01_minimal_valid_EXTENDED.json
**Purpose:** Minimal valid message structure for EXTENDED schema

**Content:**
- Required `header` field
- Empty `Location` array
- Empty `RemedialActionApplied` array
- Empty `Substation` array

**Key Differences from BASE:**
- Adds `Location` array field (empty)
- Adds `Substation` array field (empty)

---

#### 02_single_remedial_action_EXTENDED.json
**Purpose:** Single remedial action with EXTENDED schema features

**Content:**
- Same base structure as 02_single_remedial_action.json
- Empty `Location` array
- Empty `Substation` array
- OperationalLimit as **nested object** (not string)

**Key Differences from BASE:**
- OperationalLimit structure:
  ```json
  "OperationalLimit": {
    "mRID": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "OperationalLimitSet": null,
    "OperationalLimitType": null
  }
  ```

---

#### 03_complete_analysis_EXTENDED.json
**Purpose:** Complete security analysis with EXTENDED schema features

**Content:**
- Same 4 remedial action scenarios as BASE version
- Empty `Location` array
- Empty `Substation` array
- All OperationalLimit references as nested objects

**Key Differences from BASE:**
- Nested OperationalLimit objects throughout
- Additional container fields (Location, Substation)

---

#### 04_multi_region_union_demo_EXTENDED.json
**Purpose:** Cross-border analysis with EXTENDED schema features

**Content:**
- Same 2 remedial action scenarios as BASE version
- Empty `Location` array
- Empty `Substation` array
- OperationalLimit as nested objects

**Potential Extensions:**
This example uses empty Location and Substation arrays for simplicity. In a comprehensive deployment, these could include:
- **Location**: Physical addresses of substations or equipment
- **Substation**: Equipment topology (Breakers, Disconnectors, etc.)

---

## Field Reference

### Header Fields (Required in both BASE and EXTENDED)

**Required Fields:**
- `profProfile` - Profile URI (e.g., "https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4")
- `identifier` - Unique message identifier (UUID format recommended)
- `startDate` - Analysis timestamp (ISO 8601 format)
- `schemaRef` - Schema reference URI

**Optional Fields:**
- `isVersionOf` - Reference to base case or previous version (UUID or URI)
- `version` - Message version string

### RemedialActionApplied Fields

**Required Fields:**
- `mRID` - UUID format recommended
- `PowerFlowResult` - Union type, exactly one required
- `RemedialAction` - UUID reference to external RemedialAction

**Optional Fields:**
- `StageForRemedialActionScheme` - UUID reference for staged remedial actions

### PowerFlowResult Fields (BaseCasePowerFlowResult / ContingencyPowerFlowResult)

**Required Fields:**
- `atTime` - timestamp-millis (long integer)
- `isViolation` - boolean
- `ACDCTerminal` - UUID reference

**Optional Fields:**
- `absoluteValue` - float
- `value` - float (percentage)
- `valueA`, `valueAngle`, `valueV`, `valueVA`, `valueVAR`, `valueW` - float measurements
- `ReportedByRegion` - EIC code or UUID (optional)

**OperationalLimit:**
- **BASE schema**: String UUID reference (optional)
- **EXTENDED schema**: Nested object with mRID, OperationalLimitSet, OperationalLimitType (optional)

### ContingencyPowerFlowResult Specific
- `Contingency` - UUID reference (required for ContingencyPowerFlowResult)

### EXTENDED Schema Additional Fields

**Location Array:**
- Array of Location objects with address information
- Can be empty array for basic examples

**Substation Array:**
- Array of Substation objects with equipment topology
- Each substation contains an Equipments array
- Equipment types: Breaker, Cut, DisconnectingCircuitBreaker, Disconnector, GroundDisconnector, Recloser
- Can be empty array for basic examples

## Data Format Notes

### Timestamps
The `atTime` field uses Avro's `timestamp-millis` logical type:
- Format: long integer (milliseconds since Unix epoch)
- Example: `1704067200000` = 2024-01-01T00:00:00Z

**Python conversion:**
```python
from datetime import datetime
timestamp_ms = int(datetime(2024, 1, 1).timestamp() * 1000)
```

**Java conversion:**
```java
import java.time.Instant;
long timestampMs = Instant.parse("2024-01-01T00:00:00Z").toEpochMilli();
```

### UUIDs
All identifiers use RFC 4122 UUID format:
- Format: 8-4-4-4-12 hexadecimal characters
- Example: `550e8400-e29b-41d4-a716-446655440000`

**Python generation:**
```python
import uuid
str(uuid.uuid4())
```

**Java generation:**
```java
import java.util.UUID;
UUID.randomUUID().toString();
```

### ENTSO-E EIC Codes
Region references may use ENTSO-E Energy Identification Codes:
- Format: `XY-ZZZZZ-C` where XY is type code, ZZZZZ is identifier, C is checksum
- Example: `10Y-DE-ENTSOE-1` (Germany), `10Y-FR-ENTSOE-1` (France)
- Type code `10Y` indicates control area

## Validation

### Avro Schema Validation

**Using Python (fastavro) - BASE Schema:**
```python
from fastavro import reader, parse_schema
import json

# Load BASE schema
with open('SecurityAnalysisResult.avsc') as f:
    schema = json.load(f)

# Find SecurityAnalysisResult record
container_schema = next(s for s in schema if s.get('name') == 'SecurityAnalysisResult')

# Validate message
with open('03_complete_analysis.json') as f:
    message = json.load(f)
    
# fastavro will validate during write
from fastavro import writer
from io import BytesIO
output = BytesIO()
writer(output, container_schema, [message])
print("Valid!")
```

**Using Python (fastavro) - EXTENDED Schema:**
```python
from fastavro import reader, parse_schema
import json

# Load EXTENDED schema
with open('SecurityAnalysisResult_EXTENDED.avsc') as f:
    schema = json.load(f)

# Find SecurityAnalysisResult record
container_schema = next(s for s in schema if s.get('name') == 'SecurityAnalysisResult')

# Validate EXTENDED message
with open('03_complete_analysis_EXTENDED.json') as f:
    message = json.load(f)
    
# fastavro will validate during write
from fastavro import writer
from io import BytesIO
output = BytesIO()
writer(output, container_schema, [message])
print("Valid!")
```

**Using Java (Apache Avro):**
```bash
# Download avro-tools
wget https://repo1.maven.org/maven2/org/apache/avro/avro-tools/1.11.3/avro-tools-1.11.3.jar

# Validate BASE schema payload
java -jar avro-tools-1.11.3.jar fromjson --schema-file SecurityAnalysisResult.avsc 03_complete_analysis.json > /dev/null && echo "Valid!"

# Validate EXTENDED schema payload
java -jar avro-tools-1.11.3.jar fromjson --schema-file SecurityAnalysisResult_EXTENDED.avsc 03_complete_analysis_EXTENDED.json > /dev/null && echo "Valid!"
```

### Application-Level Validation

The following constraints must be validated at the application level:

1. **Minimum cardinality**: While the Avro schema allows empty arrays, application logic typically requires at least 1 RemedialActionApplied item for meaningful results
2. **PowerFlowResult type**: Must be exactly one of BaseCasePowerFlowResult or ContingencyPowerFlowResult
3. **Contingency field**: Required when PowerFlowResult is ContingencyPowerFlowResult type
4. **Header completeness**: All required header fields must be non-null and properly formatted
5. **Measurement consistency**: If OperationalLimit provided, appropriate value fields should be present based on limit type
6. **EIC format**: Region codes should follow ENTSO-E EIC format when used
7. **EXTENDED schema**: When using EXTENDED schema, Location and Substation arrays must be present (can be empty)

## Common Patterns

### Creating a BASE Schema Message with Header
```json
{
  "header": {
    "profProfile": "https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4",
    "identifier": "uuid-here",
    "isVersionOf": null,
    "version": "1.0",
    "startDate": "2024-01-01T00:00:00Z",
    "schemaRef": "https://schemas.cim4.eu/avro/SecurityAnalysisResult/2.4.0"
  },
  "RemedialActionApplied": [{
    "mRID": "uuid-here",
    "PowerFlowResult": {
      "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult": {
        "atTime": 1704067200000,
        "isViolation": false,
        "valueA": 850.0,
        "ACDCTerminal": "terminal-uuid",
        "OperationalLimit": "limit-uuid"
      }
    },
    "RemedialAction": "action-uuid",
    "StageForRemedialActionScheme": null
  }]
}
```

### Creating an EXTENDED Schema Message with Header
```json
{
  "header": {
    "profProfile": "https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4",
    "identifier": "uuid-here",
    "isVersionOf": null,
    "version": "1.0",
    "startDate": "2024-01-01T00:00:00Z",
    "schemaRef": "https://schemas.cim4.eu/avro/SecurityAnalysisResult/2.4.0"
  },
  "Location": [],
  "RemedialActionApplied": [{
    "mRID": "uuid-here",
    "PowerFlowResult": {
      "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult": {
        "atTime": 1704067200000,
        "isViolation": false,
        "valueA": 850.0,
        "ACDCTerminal": "terminal-uuid",
        "OperationalLimit": {
          "mRID": "limit-uuid",
          "OperationalLimitSet": null,
          "OperationalLimitType": null
        }
      }
    },
    "RemedialAction": "action-uuid",
    "StageForRemedialActionScheme": null
  }],
  "Substation": []
}
```

### Creating a Contingency Result (BASE Schema)
```json
{
  "header": {
    "profProfile": "https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4",
    "identifier": "uuid-here",
    "isVersionOf": null,
    "version": null,
    "startDate": "2024-01-01T00:00:00Z",
    "schemaRef": "https://schemas.cim4.eu/avro/SecurityAnalysisResult/2.4.0"
  },
  "RemedialActionApplied": [{
    "mRID": "uuid-here",
    "PowerFlowResult": {
      "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult": {
        "atTime": 1704067200000,
        "isViolation": true,
        "valueA": 1450.0,
        "ACDCTerminal": "terminal-uuid",
        "OperationalLimit": "limit-uuid",
        "Contingency": "contingency-uuid"
      }
    },
    "RemedialAction": "action-uuid",
    "StageForRemedialActionScheme": null
  }]
}
```

## Schema Comparison

### When to Use BASE Schema
- Simple messaging requirements
- Lightweight data exchange
- String references are sufficient
- Focus on power flow results only
- No need for equipment or location details

### When to Use EXTENDED Schema
- Comprehensive data requirements
- Need equipment topology information
- Location data is important
- Detailed operational limit information needed
- Integration with asset management systems
- Rich data model preferred

### Key Differences

| Feature | BASE Schema | EXTENDED Schema |
|---------|-------------|-----------------|
| **Container Fields** | 2 (header, RemedialActionApplied) | 4 (header, Location, RemedialActionApplied, Substation) |
| **Total Types** | 7 | 24 |
| **OperationalLimit** | String UUID | Nested object |
| **Location Field** | No | Yes (array) |
| **Substation Field** | No | Yes (array) |
| **Equipment Types** | None | 6 types (Breaker, Cut, etc.) |
| **Complexity** | Simple | Comprehensive |

## File Inventory

### BASE Schema Payloads
- `01_minimal_valid.json` - Empty arrays, minimal structure
- `02_single_remedial_action.json` - Single base case result
- `03_complete_analysis.json` - 4 scenarios (base + contingency)
- `04_multi_region_union_demo.json` - Cross-border analysis

### EXTENDED Schema Payloads
- `01_minimal_valid_EXTENDED.json` - Empty arrays with EXTENDED fields
- `02_single_remedial_action_EXTENDED.json` - Single result with nested OperationalLimit
- `03_complete_analysis_EXTENDED.json` - 4 scenarios with EXTENDED features
- `04_multi_region_union_demo_EXTENDED.json` - Cross-border with EXTENDED features

## References

- [Apache Avro Documentation](https://avro.apache.org/docs/)
- [ENTSO-E EIC Codes](https://www.entsoe.eu/data/energy-identification-codes-eic/)
- [RFC 4122 UUID Specification](https://www.rfc-editor.org/rfc/rfc4122)
- [ISO 8601 Timestamp Format](https://en.wikipedia.org/wiki/ISO_8601)
- [CIM Standards](https://www.iec.ch/smartgrid/standards/)