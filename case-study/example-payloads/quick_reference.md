# Quick Reference - SecurityAnalysisResult Messages

## Message Structure

```json
{
  "RemedialActionApplied": [
    {
      "mRID": "uuid",
      "PowerFlowResult": {
        "TypeName": { /* fields */ }
      },
      "RemedialAction": "uuid",
      "StageForRemedialActionScheme": "uuid" | null
    }
  ]
}
```

## PowerFlowResult Union Types

### BaseCasePowerFlowResult
```json
"PowerFlowResult": {
  "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult": {
    "atTime": 1704067200000,
    "isViolation": false,
    "ACDCTerminal": "uuid",
    "valueA": 850.0,
    /* optional fields */
  }
}
```

### ContingencyPowerFlowResult
```json
"PowerFlowResult": {
  "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult": {
    "atTime": 1704067200000,
    "isViolation": true,
    "ACDCTerminal": "uuid",
    "Contingency": "uuid",
    "valueA": 1450.0,
    /* optional fields */
  }
}
```

## Required vs Optional

### RemedialActionApplied
| Field | Required | Type | Format |
|-------|----------|------|--------|
| mRID | ✓ | string | UUID |
| PowerFlowResult | ✓ | union | See types above |
| RemedialAction | ✓ | string | UUID |
| StageForRemedialActionScheme | ✗ | string | UUID or null |

### PowerFlowResult (both types)
| Field | Required | Type | Notes |
|-------|----------|------|-------|
| atTime | ✓ | long | timestamp-millis |
| isViolation | ✓ | boolean | true/false |
| ACDCTerminal | ✓ | string | UUID |
| absoluteValue | ✗ | float | null or value |
| value | ✗ | float | percentage |
| valueA | ✗ | float | current (A) |
| valueAngle | ✗ | float | angle (deg) |
| valueV | ✗ | float | voltage (V) |
| valueVA | ✗ | float | apparent power (VA) |
| valueVAR | ✗ | float | reactive power (VAr) |
| valueW | ✗ | float | active power (W) |
| OperationalLimit | ✗ | string | UUID or null |
| ReportedByRegion | ✗ | string | EIC code or null |

### ContingencyPowerFlowResult Additional
| Field | Required | Type | Format |
|-------|----------|------|--------|
| Contingency | ✓ | string | UUID |

## Common Commands

### Validate with Python
```bash
python validate_examples.py
```

### Validate with Java
```bash
java -jar avro-tools.jar fromjson --schema-file SecurityAnalysisResult.avsc message.json
```

### Generate UUID
```bash
# Python
python3 -c "import uuid; print(uuid.uuid4())"

# Command line (Linux/Mac)
uuidgen | tr '[:upper:]' '[:lower:]'
```

### Generate Timestamp
```bash
# Python (for 2024-01-01)
python3 -c "from datetime import datetime; print(int(datetime(2024,1,1).timestamp()*1000))"

# Result: 1704067200000
```

## Validation Checklist

- [ ] Container has `RemedialActionApplied` array with at least 1 item
- [ ] Each RemedialActionApplied has mRID, PowerFlowResult, RemedialAction
- [ ] PowerFlowResult uses proper union format with type name wrapper
- [ ] PowerFlowResult has atTime, isViolation, ACDCTerminal
- [ ] ContingencyPowerFlowResult includes Contingency field
- [ ] All UUIDs follow RFC 4122 format (8-4-4-4-12)
- [ ] Timestamps are milliseconds (long), not ISO strings
- [ ] Region codes use ENTSO-E EIC format (optional)

## Quick Fixes

**Empty array error:**
- Ensure RemedialActionApplied array has at least 1 item

**Union type error:**
- Wrap PowerFlowResult object with fully qualified type name

**Timestamp error:**
- Use milliseconds since epoch (1704067200000), not string

**Required field error:**
- Check atTime, isViolation, ACDCTerminal are present
- Check Contingency is present for ContingencyPowerFlowResult type
