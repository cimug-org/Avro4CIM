```JSON
[
     {
          "type": "enum",
          "name": "UnitMultiplier",
          "namespace": "eu.cim4.ap_voc.securityanalysisresult.domain",
          "doc": "The unit multipliers defined for the CIM. When applied to unit symbols, the unit symbol is treated as a derived unit. Regardless of the contents of the unit symbol text, the unit symbol shall be treated as if it were a single-character unit symbol. Unit symbols should not contain multipliers, and it should be left to the multiplier to define the multiple for an entire data type.For example, if a unit symbol is "m2Pers" and the multiplier is "k", then the value is k(m**2/s), and the multiplier applies to the entire final value, not to any individual part of the value. This can be conceptualized by substituting a derived unit symbol for the unit type. If one imagines that the symbol "�" represents the derived unit "m2Pers", then applying the multiplier "k" can be conceptualized simply as "k�".For example, the SI unit for mass is "kg" and not "g". If the unit symbol is defined as "kg", then the multiplier is applied to "kg" as a whole and does not replace the "k" in front of the "g". In this case, the multiplier of "m" would be used with the unit symbol of "kg" to represent one gram. As a text string, this violates the instructions in IEC 80000-1. However, because the unit symbol in CIM is treated as a derived unit instead of as an SI unit, it makes more sense to conceptualize the "kg" as if it were replaced by one of the proposed replacements for the SI mass symbol. If one imagines that the "kg" were replaced by a symbol "�", then it is easier to conceptualize the multiplier "m" as creating the proper unit "m�", and not the forbidden unit "mkg".",
          "modelReference": "http://iec.ch/TC57/CIM100#UnitMultiplier",
          "symbols": [
               "k",
               "m",
               "none"
          ]
     },
     {
          "type": "enum",
          "name": "UnitSymbol",
          "namespace": "eu.cim4.ap_voc.securityanalysisresult.domain",
          "doc": "The derived units defined for usage in the CIM. In some cases, the derived unit is equal to an SI unit. Whenever possible, the standard derived symbol is used instead of the formula for the derived unit. For example, the unit symbol Farad is defined as "F" instead of "CPerV". In cases where a standard symbol does not exist for a derived unit, the formula for the unit is used as the unit symbol. For example, density does not have a standard symbol and so it is represented as "kgPerm3". With the exception of the "kg", which is an SI unit, the unit symbols do not contain multipliers and therefore represent the base derived unit to which a multiplier can be applied as a whole.Every unit symbol is treated as an unparseable text as if it were a single-letter symbol. The meaning of each unit symbol is defined by the accompanying descriptive text and not by the text contents of the unit symbol.To allow the widest possible range of serializations without requiring special character handling, several substitutions are made which deviate from the format described in IEC 80000-1. The division symbol "/" is replaced by the letters "Per". Exponents are written in plain text after the unit as "m3" instead of being formatted as "m" with a superscript of 3 or introducing a symbol as in "m^3". The degree symbol "�" is replaced with the letters "deg". Any clarification of the meaning for a substitution is included in the description for the unit symbol.Non-SI units are included in list of unit symbols to allow sources of data to be correctly labelled with their non-SI units (for example, a GPS sensor that is reporting numbers that represent feet instead of meters). This allows software to use the unit symbol information correctly convert and scale the raw data of those sources into SI-based units.The integer values are used for harmonization with IEC 61850.",
          "modelReference": "http://iec.ch/TC57/CIM100#UnitSymbol",
          "symbols": [
               "A",
               "V",
               "VA",
               "VAr",
               "W",
               "deg",
               "none"
          ]
     },
     {
          "type": "record",
          "name": "BaseCasePowerFlowResult",
          "namespace": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult",
          "doc": "Base case power flow result for a given terminal.",
          "modelReference": "https://cim4.eu/ns/nc#BaseCasePowerFlowResult",
          "fields": [
               {
                    "name": "absoluteValue",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Absolute value from a power flow calculation on a given terminal related to a given operational limit. For instance, if the operational limit is 1000 A and the current flow is 1100 A the absoluteValue is reported as 1100 A.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.absoluteValue"
               },
               {
                    "name": "atTime",
                    "type": {"type": "long", "logicalType": "timestamp-millis"},
                    "doc": "The date and time of the scenario time that was studied and at which the limit violation occurred.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.atTime"
               },
               {
                    "name": "isViolation",
                    "type": "boolean",
                    "doc": "True if the power flow result is violating the associated operational limit. False if it is not violating the associated operational limits.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.isViolation"
               },
               {
                    "name": "value",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "The value of the limit violation in percent related to the value of the operational limit that is violated. For instance, if the operational limit is 1000 A and the current flow is 1100 A the value is reported as 110 %.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.value"
               },
               {
                    "name": "valueA",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Current from a power flow calculation on a given terminal. The value shall be a positive value or zero.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueA"
               },
               {
                    "name": "valueAngle",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Voltage angle value from a power flow calculation on a given terminal.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueAngle"
               },
               {
                    "name": "valueV",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Voltage value from a power flow calculation on a given terminal. The attribute shall be a positive value.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueV"
               },
               {
                    "name": "valueVA",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Apparent power value from a power flow calculation on a given terminal.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueVA"
               },
               {
                    "name": "valueVAR",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Reactive power value from a power flow calculation on a given terminal.Load sign convention is used, i.e. positive sign means flow out from a TopologicalNode (bus) into the conducting equipment.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueVAR"
               },
               {
                    "name": "valueW",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Active power value from a power flow calculation on a given terminal.Load sign convention is used, i.e. positive sign means flow out from a TopologicalNode (bus) into the conducting equipment.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueW"
               },
               {
                    "name": "ACDCTerminal",
                    "type": "string",
                    "doc": "ACDC terminal where the powerflow result is located. Note that the value of this field is the identifier (e.g. mRID) used to reference the ACDCTerminal external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.ACDCTerminal"
               },
               {
                    "name": "OperationalLimit",
                    "type": ["null", "string"],
                    "default": null,
                    "doc": "The operational limit that has this limit violation. Note that the value of this field is the identifier (e.g. mRID) used to reference the OperationalLimit external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.OperationalLimit"
               },
               {
                    "name": "ReportedByRegion",
                    "type": ["null", "string"],
                    "default": null,
                    "doc": "The region which reports this limit violation. Note that the value of this field is the identifier (e.g. mRID) used to reference the Region external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.ReportedByRegion"
               }
          ]
     },
     {
          "type": "record",
          "name": "ContingencyPowerFlowResult",
          "namespace": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult",
          "doc": "Contingency power flow result on a given terminal for a given contingency.",
          "modelReference": "https://cim4.eu/ns/nc#ContingencyPowerFlowResult",
          "fields": [
               {
                    "name": "absoluteValue",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Absolute value from a power flow calculation on a given terminal related to a given operational limit. For instance, if the operational limit is 1000 A and the current flow is 1100 A the absoluteValue is reported as 1100 A.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.absoluteValue"
               },
               {
                    "name": "atTime",
                    "type": {"type": "long", "logicalType": "timestamp-millis"},
                    "doc": "The date and time of the scenario time that was studied and at which the limit violation occurred.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.atTime"
               },
               {
                    "name": "isViolation",
                    "type": "boolean",
                    "doc": "True if the power flow result is violating the associated operational limit. False if it is not violating the associated operational limits.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.isViolation"
               },
               {
                    "name": "value",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "The value of the limit violation in percent related to the value of the operational limit that is violated. For instance, if the operational limit is 1000 A and the current flow is 1100 A the value is reported as 110 %.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.value"
               },
               {
                    "name": "valueA",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Current from a power flow calculation on a given terminal. The value shall be a positive value or zero.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueA"
               },
               {
                    "name": "valueAngle",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Voltage angle value from a power flow calculation on a given terminal.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueAngle"
               },
               {
                    "name": "valueV",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Voltage value from a power flow calculation on a given terminal. The attribute shall be a positive value.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueV"
               },
               {
                    "name": "valueVA",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Apparent power value from a power flow calculation on a given terminal.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueVA"
               },
               {
                    "name": "valueVAR",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Reactive power value from a power flow calculation on a given terminal.Load sign convention is used, i.e. positive sign means flow out from a TopologicalNode (bus) into the conducting equipment.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueVAR"
               },
               {
                    "name": "valueW",
                    "type": ["null", "float"],
                    "default": null,
                    "doc": "Active power value from a power flow calculation on a given terminal.Load sign convention is used, i.e. positive sign means flow out from a TopologicalNode (bus) into the conducting equipment.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.valueW"
               },
               {
                    "name": "ACDCTerminal",
                    "type": "string",
                    "doc": "ACDC terminal where the powerflow result is located. Note that the value of this field is the identifier (e.g. mRID) used to reference the ACDCTerminal external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.ACDCTerminal"
               },
               {
                    "name": "OperationalLimit",
                    "type": ["null", "string"],
                    "default": null,
                    "doc": "The operational limit that has this limit violation. Note that the value of this field is the identifier (e.g. mRID) used to reference the OperationalLimit external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.OperationalLimit"
               },
               {
                    "name": "ReportedByRegion",
                    "type": ["null", "string"],
                    "default": null,
                    "doc": "The region which reports this limit violation. Note that the value of this field is the identifier (e.g. mRID) used to reference the Region external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.ReportedByRegion"
               },
               {
                    "name": "Contingency",
                    "type": "string",
                    "doc": "The contingency that has this power flow result. Note that the value of this field is the identifier (e.g. mRID) used to reference the Contingency external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#ContingencyPowerFlowResult.Contingency"
               }
          ]
     },
     {
          "type": "record",
          "name": "RemedialActionApplied",
          "namespace": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult",
          "doc": "Remedial Action or Remedial Action Stage that has been applied to the power flow case which has the associated power flow result.",
          "modelReference": "https://cim4.eu/ns/nc#RemedialActionApplied",
          "fields": [
               {
                    "name": "mRID",
                    "type": "string",
                    "doc": "Master resource identifier issued by a model authority. The mRID is unique within an exchange context. Global uniqueness is easily achieved by using a UUID, as specified in RFC 4122, for the mRID. The use of UUID is strongly recommended.For CIMXML data files in RDF syntax conforming to IEC 61970-552, the mRID is mapped to rdf:ID or rdf:about attributes that identify CIM object elements.",
                    "modelReference": "https://cim4.eu/ns/nc#RemedialActionApplied.mRID"
               },
               {
                    "name": "PowerFlowResult",
                    "type": [
                         "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult",
                         "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult"
                    ],
                    "doc": "Power flow result that is obtained when the remedial action is applied.",
                    "modelReference": "https://cim4.eu/ns/nc#RemedialActionApplied.PowerFlowResult"
               },
               {
                    "name": "RemedialAction",
                    "type": "string",
                    "doc": "Remedial action that is applied. Note that the value of this field is the identifier (e.g. mRID) used to reference the RemedialAction external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#RemedialActionApplied.RemedialAction"
               },
               {
                    "name": "StageForRemedialActionScheme",
                    "type": ["null", "string"],
                    "default": null,
                    "doc": "The stage of the remedial action scheme that is applied. Note that the value of this field is the identifier (e.g. mRID) used to reference the Stage external to this profile.",
                    "modelReference": "https://cim4.eu/ns/nc#RemedialActionApplied.StageForRemedialActionScheme"
               }
          ]
     },
     {
          "copyright": "Copyright 2025 UCAIug SPDX-License-Identifier: Apache-2.0",
          "type": "record",
          "name": "SecurityAnalysisResult",
          "namespace": "eu.cim4.ap_voc.securityanalysisresult",
          "doc": "The Security Analysis Result (SAR) profile is a profile to exchange a security analysis result.The security analysis result includes each limit violation detected for each assessed element and for a given contingency. The limit violation has a direct association to operational limit and contingency. The association to the operational limit provides information on the following:- The terminal (the end of the equipment) where the limit is defined- The equipment to which the limit is related- The type of the limit e.g. PATL, TATL, etc including the relevant time phase and other conditionsThe association to the contingency provides information which contingency was simulated when this limit violation was detected.This profile is not intended to replace the Topology (TP) and State Variables (SV) profiles. Its intention is to exchange a power flow result that is relevant for security optimization, either through violation or through a loading threshold. Systems should not use this profile for dumping a full database. The modeling is optimized to have the minimum size in addition to a well defined value definition (e.g. active power, apparent power, etc.).Recommendation: If the terminals are connected with zero impedance, it is recommended to include only one terminal with a voltage (e.g. the terminal of a BusbarSection). The connection between Contingency and Remedial Action is given by the Remedial Action Profile. The connection between AssessedElement and PowerFlowResult is given by the OperationalLimit.",
          "fields": [
               {
                    "name": "RemedialActionApplied",
                    "type": {
                        "type": "array",
                        "items": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.RemedialActionApplied"
                    },
                    "doc": "Remedial Action or Remedial Action Stage that has been applied to the power flow case which has the associated power flow result.",
                    "minCardDoc": "[min cardinality = 1] Application level validation will be required to ensure the array contains at least 1 item."
               }
          ]
     }
]

```
