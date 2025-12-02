# SecurityAnalysisResult Example Payloads

This directory contains example JSON payloads for the SecurityAnalysisResult Apache Avro schema based on the updated schema structure where only `RemedialActionApplied` appears at the container level.

## Schema Structure Changes

The updated schema reflects the topological analysis results:
- **Container field**: Only `RemedialActionApplied` array (minimum cardinality = 1)
- **PowerFlowResult**: Union type (not array) with cardinality 1..1
- **Union members**: BaseCasePowerFlowResult or ContingencyPowerFlowResult

## Example Files

### 01_minimal_valid.json
**Purpose:** Minimal valid message structure

**Content:**
- Empty RemedialActionApplied array

**Note:** While this is structurally valid Avro, it violates the application-level constraint that requires at least 1 RemedialActionApplied item (minCardinality=1).

---

### 02_single_remedial_action.json
**Purpose:** Single remedial action with base case power flow result

**Content:**
- 1 RemedialActionApplied record
- PowerFlowResult uses BaseCasePowerFlowResult type
- Demonstrates proper UUID format for all references
- Shows ENTSO-E EIC code for Region

**Key Features:**
- Base case result (no contingency)
- Current measurement (valueA = 850.0 A)
- Non-violation scenario (85% loading)
- Proper Avro union JSON encoding

---

### 03_complete_analysis.json
**Purpose:** Complete security analysis with multiple remedial actions and scenarios

**Content:**
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

---

### 04_multi_region_union_demo.json
**Purpose:** Demonstrates Avro union type encoding with cross-border analysis

**Content:**
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
- Preventive: FR-ES interconnector at 1850 MW (74% loading)
- Curative: DE-FR interconnector at 2400 MW (96% loading after contingency)

**Key Features:**
- Demonstrates both union members
- Shows preventive vs curative actions
- Cross-border multi-region analysis
- ENTSO-E EIC codes for regions

## Field Reference

### Required Fields (RemedialActionApplied)
- `mRID` - UUID format recommended
- `PowerFlowResult` - Union type, exactly one required
- `RemedialAction` - UUID reference to external RemedialAction

### Required Fields (BaseCasePowerFlowResult / ContingencyPowerFlowResult)
- `atTime` - timestamp-millis (long integer)
- `isViolation` - boolean
- `ACDCTerminal` - UUID reference

### Optional Fields (PowerFlowResult)
- `absoluteValue` - float
- `value` - float (percentage)
- `valueA`, `valueAngle`, `valueV`, `valueVA`, `valueVAR`, `valueW` - float measurements
- `OperationalLimit` - UUID reference (optional)
- `ReportedByRegion` - EIC code or UUID (optional)

### ContingencyPowerFlowResult Specific
- `Contingency` - UUID reference (required for ContingencyPowerFlowResult)

### RemedialActionApplied Optional
- `StageForRemedialActionScheme` - UUID reference (optional)

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

**Using Python (fastavro):**
```python
from fastavro import reader, parse_schema
import json

# Load schema
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

**Using Java (Apache Avro):**
```bash
# Download avro-tools
wget https://repo1.maven.org/maven2/org/apache/avro/avro-tools/1.11.3/avro-tools-1.11.3.jar

# Validate (converts JSON to Avro binary, will fail if invalid)
java -jar avro-tools-1.11.3.jar fromjson --schema-file SecurityAnalysisResult.avsc 03_complete_analysis.json > /dev/null && echo "Valid!"
```

### Application-Level Validation

The following constraints must be validated at the application level:

1. **Minimum cardinality**: RemedialActionApplied array must contain at least 1 item
2. **PowerFlowResult type**: Must be exactly one of BaseCasePowerFlowResult or ContingencyPowerFlowResult
3. **Contingency field**: Required when PowerFlowResult is ContingencyPowerFlowResult type
4. **Measurement consistency**: If OperationalLimit provided, appropriate value fields must be present based on limit type
5. **EIC format**: Region codes should follow ENTSO-E EIC format when used

## Common Patterns

### Creating a Base Case Result
```json
{
  "RemedialActionApplied": [{
    "mRID": "uuid-here",
    "PowerFlowResult": {
      "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult": {
        "atTime": 1704067200000,
        "isViolation": false,
        "valueA": 850.0,
        "ACDCTerminal": "terminal-uuid"
      }
    },
    "RemedialAction": "action-uuid",
    "StageForRemedialActionScheme": null
  }]
}
```

### Creating a Contingency Result
```json
{
  "RemedialActionApplied": [{
    "mRID": "uuid-here",
    "PowerFlowResult": {
      "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult": {
        "atTime": 1704067200000,
        "isViolation": true,
        "valueA": 1450.0,
        "ACDCTerminal": "terminal-uuid",
        "Contingency": "contingency-uuid"
      }
    },
    "RemedialAction": "action-uuid",
    "StageForRemedialActionScheme": null
  }]
}
```

## References

- [Apache Avro Documentation](https://avro.apache.org/docs/)
- [ENTSO-E EIC Codes](https://www.entsoe.eu/data/energy-identification-codes-eic/)
- [RFC 4122 UUID Specification](https://www.rfc-editor.org/rfc/rfc4122)
- [ISO 8601 Timestamp Format](https://en.wikipedia.org/wiki/ISO_8601)
