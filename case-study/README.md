# Case Study: CSA 2.4 Security Analysis Result (SAR) Profile

## Table of Contents
- [1. UML Profiles Overview](#1-uml-profiles-overview)
  - [1.1. Validation Capabilities](#11-uml-diagram-of-the-csa-24-sar-profile)
  - [1.2. Namespace Support](#12-sar-profile-generate-apache-avro-schema) 
  - [1.3. Inheritance and Type Extension](#13-uml-diagram-of-the-csa-24-sar-extended-profile) 
  - [1.4. Schema Modularity and Reuse](#14-extended-sar-profile-generate-apache-avro-schema)  
- [2. CIMTool Workflow for Generating Artifacts](#2-cimtool-workflow-for-generating-artifacts)
- [3. Overview of Apache Avro](#3-overview-of-apache-avro)
- [4. The Purpose of an Apache Avro Schema (.avsc)](#4-the-purpose-of-an-apache-avro-schema-avsc)
- [5. Apache Avro Schema Limitations -vs- JSON and XSD Schema Specifications](#5-apache-avro-schema-limitations--vs--json-and-xsd-schema-specifications)
  - [5.1. Validation Capabilities](#51-validation-capabilities)
  - [5.2. Namespace Support](#52-namespace-support) 
  - [5.3. Inheritance and Type Extension](#53-inheritance-and-type-extension) 
  - [5.4. Schema Modularity and Reuse](#54-schema-modularity-and-reuse)  
  - [5.5. Human Readability and Tooling](#55-human-readability-and-tooling)  
  - [5.6. Type System Compared to JSON Schema](#56-type-system-compared-to-json-schema) 
- [6. Mapping CIM Profiles  to Apache Avro Schema: Impacts to Formal Mapping Specifications](#6-mapping-cim-profiles--to-apache-avro-schema-impacts-to-formal-mapping-specifications)
- [7. Design Decisions](#7-design-decisions)
  - [7.1. Single or Multiple Avro Schemas](#71-single-or-multiple-avro-schemas)
  - [7.2. Namespacing](#72-namespacing)
  - [7.3. Canonical Model Traceability Within Avro Schemas](#73-canonical-model-traceability-within-avro-schemas)
  - [7.4. Class Inheritance: Flattened Record Structure](#74-class-inheritance-flattened-record-structure)
  - [7.5. Cross Profile Associations: Reference Strategy](#75-cross-profile-associations-reference-strategy)
  - [7.6. Mapping CIM Compound Types](#76-mapping-cim-compound-types)
  - [7.7. Date Datatypes and Apache Avro Logical Types](#77-date-datatypes-and-apache-avro-logical-types)
  - [7.8. Cardinality Mapping](#78-cardinality-mapping)
  - [7.9. Document Element Root Fields](#79-document-element-root-fields)
- [8. Example SAR Profile JSON Payloads](#8-example-sar-profile-json-payloads)
  - [8.1. SecurityAnalysisResult.avsc](#81-securityanalysisresultavsc)
  - [8.2. SecurityAnalysisResult_EXTENDED.avsc](#82-securityanalysisresult_extendedavsc)
  - [8.3. Profile Constraint Rules](#83-profile-constraint-rules)
  - [8.4. Complex Constraint Validation](#84-complex-constraint-validation)
  - [8.5. Notes on Example Payloads](#85-notes-on-example-payloads)
- [9. SecurityAnalysisResult.avsc Usage](#9-securityanalysisresultavsc-usage)
  - [9.1. Code Generation](#91-code-generation)
  - [9.2. Writing Messages](#92-writing-messages)
  - [9.3. Reading Messages](#93-reading-messages)
- [10. Schema Evolution](#10-schema-evolution)
  - [10.1. Backward Compatible Changes](#101-backward-compatible-changes)
  - [10.2. Breaking Changes](#102-breaking-changes)
  - [10.3. Best Practices](#103-best-practices)
- [11. Validation](#11-validation)
- [12. Tools](#12-tools)
- [13. References](#13-references)
- [14. License](#14-license)


## 1. UML Profiles Overview

For this CIM mapping to Apache Avro schema case study the CSA 2.4 Security Analysis Result Profile was targeted. It does not comprehensively represent all CIM types that can be included in a profile. For example Compound data types defined within the CIM are not defined as part of the SAR profile. Despite this, the initial Avro schema builder outlined here includes support for all types that are currently identified as potentially relevant in an Avro schema. A special extended variant of the profile was used to proof out and test the generation of a more complex profile definition that included Compound types and additional edge cases.

### 1.1. UML Diagram of the CSA 2.4 SAR Profile

![](https://img.plantuml.biz/plantuml/svg/hLVRJkD847ttLxJ8mqYB4W84jA08ZUF4cf56B4eG-x3QXyQk92tQRgkx3PCT-FUjQZl1bpWxZDOMYbrrwjIrA-6piLpRF96ULHcpYZqgQrN2Og4XiaAbdlU9gOoUs4hpM41gsIEFdbPQvMbC5aOakUGMsCndfvU3xsZB4PeOppo9DEFuo2OxYq19fLaldD3TxqqlfsV1Z9ny4J6mI79Zc8XKQhor4mWTIIYU8SdiXYLWroesP7A_5eQHoZ2x1iQjrU8nnqWAWdVtZjRVgh-YTY7-0JQiG7ojrKiPtEXYT3cwg2ZDciVgHQGhPdKEv9fqAGcMQ90g2rLtzCv-U8nzQkMz7DADghOcMluEvwHHEWPT3uMnKAMsb9sxgvJTyWGmUC5bAN2PdPxjYB18qLCAMDPJb2tgcRnvpIgvkDljceL1UFNQ2wzcqndnx8fPYUl27mr7GJtMeRkseTcD19mm6LCa5ZGekybW7_mBS8pziAP6jxzg-qf0VIdhCFjk8JKqsovtd2FZE6DOYbIuyMyLOpnksDxSiDuifFkr9yLgRN7uDri__gEncaVsGmvuQKJCeRjAvWr8gLwstBCqJq1pOlavNdWtS45VXU7iuhKt7OP-SEaLd7WCjFbGj40cPQZd8VA5OJ0C9jKv68zdMz2kZ-q9QgFUwAYviryhYjcDZbSgjp5KDDw-0_cJKpAug1scKVc6G3ThxR5ympZzE6QyS6NL2zTniPNoB0FD92nn1fHEyLajdQ9Savwpu8Oj8Cgri1i_vN9ZX9c3oQMz9MsZ--nEgwMGSDrXGn3Q2oRZ4OBMrcRc-kI4P_rd5GqYaOm6a9_ijpqv6unw7k1gfxYe_dum3Uv19m9n7g_DSqt7watwGTr3ymat4E3VdNh1Oui957wOIDU0xWrRQlo-QMYzWhxBTZo-3T15uQEbVEttkomfe02xXmiGqcZJJAUYwiYcWlYLX0wO_tTEtSRU0TwZumFAuM3msn7hzs_gC7PwLAl5THaTLU2Fh5mXN7uJYR3D6ZRr7OLiPj1WwQXEWmhd_BZKe5n9t2OFNEQ00QCjj-mlJF3qxnhEtek405FyoKAFLQKm3q995pqY9af9u6aLy5oot-4IeLz2RNw7y14Iv5eZX7BeXAKhIMpkSmeh3RXoEx0FY7jGql8LT4B8hP_XoaE_he3To3b2vu1r5Czm4FedSRPXUsRqwFZGTDKs1OvNRQjyZAerOHJIKNlmTbFRfdU0zrvNsUUGAMqXdcpci2gLo5w-L4z28MfERWdSFx35xCyX0Tp-qYz-ORS8gjeF467hfO3OsqTJwYfRfjBePFvrYfajy0ivNH7rVbkgpJvvjNhDKcFbcg9T2N_TIwD4hZqroV_WOq57BLIwZbn4QtmlPy91QYFX9Cs65h3EFdx6Brlw1ypx5m00)

### 1.2. SAR Profile Generate Apache Avro Schema

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


### 1.3. UML Diagram of the CSA 2.4 SAR EXTENDED Profile

![](https://img.plantuml.biz/plantuml/svg/hHdRRjiuzbrVGIGF0soGD6cJOXi4HHrVPeBff4MSoJmize6MQ9iRYjIIL59lst-VSoXA5YNATOgDKTdakT_8SzGFnb9jormaLXN4Ah9GrggSP5opr78ba-GNeWhyHLQQPfn9InOqVLnfLSeCAGpFIq4j8zCHENjxyWLeAQ3c9Z6FN1OKVfBhXcAiXDBaS7QDd-F3oTl9w_4KXVp6CqPIGOqXF5NI96ktmNKgMFMRvONTa9pPjSeCajolkI48CiIk6Q7MQhueGPEAmLQyOpkgmJS0dVF_CNBQGXbfhPwzSkSNvs_FBrf0iwQPUkPoHQmk6Of6cCID9H18PLbR7Ax9ruIGNtfsBq_n0w0-95t-axr1YD8Pqw4gXCoKj7xlZVlpUxSqPw0yfy9lEClENZUCm4dfesJWrZFOcuUMV4-wNd9wzyCrdOrdxxuTpjvD9rFaqpYpfTV5wFpqV1pgUkh-QjNiHZ38C96X9fPffkocO37qtnW5tNT30dPnhyLSW77nVfX-iKmQp6rdE-IHSJmpjkIIk_JlEUFggaFx_ZqvdCxmy-rGyDNQGl9lQZcZXR6QflP50gWdGiupz-SvRvWGwhdcFPLbpZH5BllnPLi2f_IxsMnwyUrmSZeRZTyc5Kye0srUf2r34g_gcndmcyt6f-FhTXvSNKrhf2QE_GpgOtt7KdEr7pZ5DAcJx0Ei5Mn1abYrxfb29nT1fPtx5I2wHEpdx7SmOfHbX5QYh7gcEgkEhB8ec2Q2BI51b9RmMoqTm9-XYQEWXinPMcfkDoD9nSPmSyTCAUmjGZkfGJvhjUI2NGxG80eU6gQWAI2jhItCvQjNj3X-Kkb9olFp4rQ-YjE-UdroVfWmEFTHBmpEQ3mPtpETSy1BQ93i0Oo4GVgAeGNoQshO6FvzLizGqs109OTmkMOWtf2bXike0qqgzaEqLrUtOn1HoUW1lnqV3r4wXJIZZn26ZW8Qp4qjo-RFtARhMW6FMuiAaRoe2WTPZvNCIhXJvMhwtv8NETwaePXcl-l966d4eLkWansXydf8lDkV8BotAcEmjfJBlj7edVa6gYO7_wXIfooYrm2hY7OHVesUSIvUMxI0oNGJaLw31jAhXV4p6RQ3_CT9DaoSxE1xSBG3U7foyey3SdpyFaGZPmU1BowzTiu3PTVK88KHJDpXHuu4aokxdW5ENRHdmEgZsoN_0jbd_qPZszaDv2XamatgOoXNO-ZOIcxZXLINGoLz5vrtRRrqudRWXsgehjp0w03FosbRrOOZtw5otQD0XiiLFSxbdEad3dLm2RU74epAsj48bahdL8ZDpJ3Qhh8AYhvP35NxS14EcRIps8TFXb2thPW8tuEZo2OQVa1KLGKDtFZY28p-rRKf-wP513kI77qib7IJWKY-v2pxjFWFx7QTreL7qo6_kvb0YEVG5ycLFy5QFHxvo7DkV-SoIx2Hsov9jxEh-551TVwWXAKhXkirj0cj9HmhhPKgF3aLVjUnHr11hQY_tGQCZ0ik8B2ui1vgq-hfCA31mU6evk_Qk8yghSm8Gz3pNEMm1hi-dPg1TEiXvpD82kYjD7Pc3i2OzMiKs_ExYs7D7DNwj3NSMRoVUZqgdobnULtif4T-SDJTUg2YPCs9ho8OSsPzCKGW-uW8xfCky1xQv07szww33dEpYpEfmxNDrvqeGpcyC4h0ZE2yKIUnpzughM9U-QaG8k6BeXZpCZ8PZ5JZwwOf1kS63NJ47XU7dpB8KTRbGzEK5L2tWarA7p8t842BQDftJhlCTWUiHSJDZNGJOfjvv4A30Q_KSD9G3F0ZChbidMBRAsq__zTfsEskEaVlGESMENyz9CWvq11egGtDgF3iFSyHOK8jmvzj83SFN8cEBze8JvxxPvWuiJ_eytSO8q0Pbre32jemY2PN0hcvxmbRmJ5eXd0V0C-Vp8CeA7PKm7mCSfq3Xp7l0FMEKRphsKxKlv1d7otIFnySxUfyWx6oMq4lOnKx4IC7thQZxhEl3kicrctkTwnG6kR1wyqTMtaWSFUBTfSUw-5-KFEnBEzmYBHbySgADRAeLMUWZVPENJqwDB4wMs3BZXqyBMWARKIOujs6gpjxWs9tB1NAx3cbDyYxnlGQgUAEyUaQvgCMFpsgNksEvIpZL8og5-Sz1cte2nX17Ge8M19Zus6jSRHLTRr-CawBYHhRRwTof8WWaxkxJKh_lA2Ebrov13l_1nbpFFfcIeUOytGDQs-PGmbArErqOrgXxGebODZI90qwB01nhF82nuYc939_XNHl4wWKqsrvDSkfVWptwjiac1t2RhXMeRKNqqHGkIehuojcxUEyKDVgom04fW8nXaal2ZQLvPUr2m8GQZHXCDw9feMlbWvkrRDieFVRnVRwQ4prL6rBFNB4K5hvu5S6GoKLKQr3bDTxuBo9uYnABZBOS_zN4qD8X_os54hf-j1EcxwjXoU7E4DFlO2pBneopPOy7arJBeT1qERT3dF5r-jH09osQounWExADD6N6HZASe7PqfyK8-yCVAIt5AJp8hJpsc6BwIJ5hhNOIzDkrGzVRTKh6-WV1ZfWDGY1XeDb_XeSxlaqS-PaKUV5tX9NPZStvxP-lEzQ1HMfaS6yJ07DwitBSj5q1lZk0S4-QRYSdxXC-_Jzjo5_mhpn1zmw70zuNOsOuTx0JFc8ZnmVKad_SgBBUBx95qficLRh0OOZV3ouH_LpfF7hBtnaRDMMb7nW4ACS9hs_0G00)

### 1.4. EXTENDED SAR Profile Generate Apache Avro Schema

The EXTENDED version includes additional classes for demonstration purposes (Location, OperationalLimit, OperationalLimitSet, OperationalLimitType with compound types). See [SecurityAnalysisResult_EXTENDED.avsc](./case-study/example-schemas/SecurityAnalysisResult_EXTENDED.md).

## 2. CIMTool Workflow for Generating Artifacts

The standard workflow for generating artifacts used by **CIMTool** appears below. This reflects those that appear as part of the CIMTool project in this GitHub repository:

![](https://img.plantuml.biz/plantuml/svg/fLVVR-8u47xFNp5g7xP5eWhzmNMhQhKKajsUe5IbT-r3fSe40xucTcGxKFPqzxVVs92Ga0IrElfGy7pZ-URpU3BvhZIHEeb2HuiOj82nq5f4C66I1fe9xipPX0ADOhs2YEgvc2Z7-G8_gQImf40cVoLAWvvJc0l9VWckIGWg9W7ZCvWA2MEgDPLcIJCqISedL1hZaN6w3mdNVmpwCA5JaeGQc89_JzrZHxqn7XD98k1YGXMDeUMwrO4NL6eM_F-mPuMmal2tD6nk-OO4RpCf4ZxfYX3Z7FhsaqEeEPc8fUL1GaLpbeUrc-H8x2A_fv18RhFV-SLv-pnlxKWfbdL67mNVs5pxoNCDGg8K_Eq0V86ku86acguNgGAbLo4r-JFEx453TMwUYvPo682ErfAD4qshZ2CKdmgzYYkjFod1XAiiLLAaguzp5hnnYYcrSCrcrHahBKcWFv7LzJNPE7tx1eUzC_ENpx1etvDj3R0YynhaZWerg9mYHKITEWLKffJ7auXAOZJvc4fqws2YVNKzxwjR56e7ibUhEcobN7NWtRgfHnPKsm7L2rS4PjfrRmSzeeaX-J7jSWuc85wYcswlgDqEPAzsTTXAxUh0fMgh0nQasm7LIrS4RgKJKMo4-TG5JPrgjSlPzmjN1QnMhGfPbgmALTGhXzWZrXQLANKlMOIAB3xJeQ-lCoyJq6jxRNmK5RGg0FQALOsiLAiQMfAh6bRGgm2f5omFsxOqV31AzIcvg75PbrNNxTxijBFKkB-HbL3LBQm4AybJXXIxLchUqwysa9RpJ-K0ecL27KSpZSBW1NqK8eGd8T-ceLXUWPb_SEBhuDI48r5d8GMCy5j47CSCVuJFqEsWqJ2u0GvXOQFnmez66EGTlCxn0H1bzpZqBlrB_y8E6W1JPhmmMkktvkbhzyV06vsr5cxhzVvXw3TFtOLx-lh2xxgdPujs-tKmQhd6sFFMNqueYUt6NWTpAl8OFlLXo3MgCrjPAlTIs6YzxCpNf0pmqE_wPrs_G6f4WqGolUfm4guKKmzKuUnt8fPXwlRgt_QzKk3dKVyHRZaUCpCJbuo8jl7j4ku5puF-ElApaSFlz6vwbn-9_1w5gLjrv1-Kc-SHdU0KgDaKd-dAHhQC7d54LLDB9qkzy-V34AUQ2__SQt-40Lce0FShrq2wtQXhI7aTQ4esc-lcjqmckBB7R8hgXQT7uKsO5lB0oK2PMvfZmIrLy8NZV8fplC943fmCcZaR7iRHvdsJi13l0T8cbdQJ8EscihHFtbLuVE0uNE2ektcN4LDJJG2DnlhihXeDC7K6Hvu97qkcC7KPp9b6xOyj4kbeoYtKJy8G0i85joyPo1qYSniipgHXG9jRKR6VWsyUOEYKI6NrjaCDwn6EdkWuAtReOq4aP4RBt8nN2F56_KBpsv9BhtxnaEg9PUMuGmpB5RP5lwwy4gLXhB5N_yAIDQzXzi2GNe3z2ZjI9IsMxXZO6i0V5dQ7f0mC_mCsLoD-k885KsoCEM3ZGlxDHD4zYTZotKd2L3mSvIeTVXyDxqffv6_4khJgrBKfwi9TjFPXekF4xZLYzZvljh57i6Hw3fWGsA4obu7p7R-Rzt77-Gy0)

- The UML model is imported into CIMTool and is referred to as the "schema" representing the full canonical model. The schema formats supported by CIMTool include either `.xmi` (XML Metadata Interchange) or EA Project files of the following types: `.eap`, `.eapx`, `.qea`, `.qeap`, and `.feap` 
- The OWL ontology is the formal profile definition. The format is in OWL and is derived from the canonical CIM using the CIMTool user interface.
- For the purposes of XML transformation, a corresponding internal XML format is created from OWL that is optimized for XSLT transformation into other downstream formats. This XML internal format is not typically visible to an end-user but is used internally as input into the XSLT transformation process to produce final artifacts.
- For this case study the desired generated artifact is an Apache Avro schema i.e. an `.avsc` file that is a JSON representation of the profile.

CIMTool utilizes what is known as "builders" to generate these various artifacts based on the OWL profile definition. As highlighted above, one flavor of supported builder it an XSLT builder. XSLT (Extensible Stylesheet Language Transformations) is an XML-based language used to transform the structure and content of an XML document into another format, such as HTML, JSON, PlantUML, plain text, PDF, etc. XSLT operates on the principle of transforming an input document's structure into a new result structure. For further detail refer to CIMTool's [CIMTool Builders Library](https://cimtool-builders.ucaiug.io/) companion website.

## 3. Overview of Apache Avro

**Apache Avro** is a binary data serialization system designed for efficient, compact data exchange and long-term storage in big data environments. Created by Doug Cutting (creator of Hadoop) in 2009, Avro's primary purpose is to serialize structured data into a compact binary format that is significantly smaller and faster to process than text-based formats like JSON or XML. It is the de facto standard for data serialization in Apache Kafka, Hadoop, and Spark ecosystems, where millions or billions of records need to be stored, transmitted, and processed efficiently.

Avro's key distinguishing feature is **schema evolution** - the ability to read old data with new schemas and vice versa. This makes it ideal for long-term data archival and systems that evolve over time, as you can add or remove fields without breaking existing readers or writers. Avro also supports RPC (remote procedure calls) and generates type-safe code in multiple programming languages (Java, Python, C++, etc.), making it suitable for both data storage and inter-service communication.

## 4. The Purpose of an Apache Avro Schema (.avsc)

An **Avro schema** (`.avsc` file) is a JSON document that defines the structure, types, and metadata of your data. Its primary purposes are: 

1. **data validation** - ensuring data conforms to the expected structure when serialized/deserialized; 
2. **code generation** - automatically creating type-safe classes in your programming language;
3. **schema evolution** - enabling readers and writers to negotiate compatible versions when schemas change over time; and 
4. **self-documentation** - serving as the contract between data producers and consumers.

Unlike XML Schema or JSON Schema which focus primarily on validation, Avro schemas are tightly integrated with the binary serialization format itself. The schema can be embedded directly into Avro data files (making them self-describing) or registered in a Schema Registry for efficient reference by ID. This design enables Avro's compact binary format - field names don't need to be stored in the data because the schema provides the structure, resulting in significant space savings compared to self-describing formats like JSON or XML.

## 5. Apache Avro Schema Limitations -vs- JSON and XSD Schema Specifications

In view of the primary objectives and purposes just outlined, Apache Avro schemas do not entirely align with more traditional schema specifications and therefore do not support the full breadth of features one might be accustomed to. The following outlines specific limitations:

### 5.1. **Validation Capabilities** 
Avro schemas provide only basic type validation (is this field an integer? a string? an array?), whereas XSD supports rich validation rules including regular expression patterns, numeric ranges, string length constraints, enumerations, and complex cross-field validation. For example, XSD can enforce that an email field matches a specific pattern or that an age value falls between 0 and 150, while Avro simply validates that the field is a string or integer respectively. This means Avro schemas cannot express business rules or data constraints beyond structural type correctness.

### 5.2. **Namespace Support** 
Avro only supports namespaces at the type level (records, enums), not at the field level within records. XSD, by contrast, allows every element and attribute to have its own namespace, enabling schema mixing where different fields come from different XML namespaces (e.g., `<cim:Terminal>`, `<nc:isViolation>`). This is critical for standards like CIM where fields originate from different specification namespaces, but when mapping to Avro, all fields must be flattened into a single namespace with their origins documented only in comments.

### 5.3. **Inheritance and Type Extension** 
XSD supports full object-oriented inheritance through extension and restriction mechanisms, allowing complex types to inherit from base types and add or constrain fields (e.g., `<xs:complexType>` extending a base `PowerFlowResult` type). Avro has no inheritance support - all record types are flat and standalone. When mapping object-oriented models like CIM to Avro, inheritance hierarchies must be flattened by copying all parent class fields into each child record, or represented using union types to allow multiple concrete implementations. This results in schema duplication and makes it difficult to maintain DRY (Don't Repeat Yourself) principles when multiple types share common fields.

### 5.4. **Schema Modularity and Reuse** 
XSD has sophisticated mechanisms for schema composition including imports, includes, type extension, restriction, and substitution groups, allowing complex schemas to be built from reusable components across multiple files. Avro's support for multi-file schemas is more limited - while you can define types in separate files, there's no formal import mechanism, and schema composition is primarily achieved by embedding type definitions or using a Schema Registry. This makes XSD better suited for large, modular, standards-based schema libraries.

### 5.5. **Human Readability and Tooling** 
XSD schemas are designed to validate human-readable XML documents, with extensive IDE support, visual schema designers, and debugging tools built over 20+ years of enterprise adoption. Avro's binary format is not human-readable without deserialization tools, making debugging harder, and its primary focus on programmatic access means fewer visual design tools exist. Additionally, XSD integrates seamlessly with XML technologies (XSLT, XPath, XML databases), while Avro requires specialized tools for inspection and processing.

### 5.6. **Type System Compared to JSON Schema** 

While both Avro and JSON Schema use JSON syntax for schema definition, their type systems differ significantly. Avro has a richer set of primitive types (int, long, float, double, bytes, fixed) compared to JSON Schema's simpler number and integer types, making Avro more precise for numeric data representation. However, JSON Schema excels at validation with built-in format validators (email, uri, date-time, ipv4, etc.) and pattern matching via regular expressions, features that Avro completely lacks. Avro compensates somewhat with "logical types" (decimal, date, timestamp-millis, uuid) that add semantic meaning to primitive types and enable proper code generation (e.g., generating `java.time.Instant` instead of `long`), but these provide type interpretation rather than validation. JSON Schema also supports conditional schemas (if/then/else), dependencies between properties, and the `additionalProperties` constraint, giving it more flexibility for validation scenarios, while Avro's strength lies in its integration with binary serialization and schema evolution rather than runtime validation.

## 6. Mapping CIM Profiles  to Apache Avro Schema: Impacts to Formal Mapping Specifications

It is normal protocol to develop IEC standards-based mapping specifications for specific syntactical formats (e.g. XSD Schema, JSON schema, etc). Such specifications provide directives on how to precisely map classes, attributes, associations, cardinalities and constructs such as unions and XOR, to specific syntactical mappings. The **IEC 62361-100** and **IEC 62361-104** are examples of such specifications. It is a long term goal of this case study to produce a corresponding IEC specification for mapping to Apache Avro schema for use in developing profiling tools that generate IEC compliant Avro schemas derived from the CIM.

For the purposes of defining such a specification for Apache Avro, the limitations outlined in the previous section highlight areas requiring alternative aproaches and/or work arounds. Such limitations were taken into consideration as part of this case study.   

## 7. Design Decisions

The following subsections describe the more signficant design decisions that went into the implementation of the **CIMTool** Avro schema builder including solution approaches addressing some of the previously mentioned limitations.

### 7.1. Single or Multiple Avro Schemas

**Decision: Single Schema File Approach**

For this case study, all Avro type definitions are contained in a single `.avsc` file as a JSON array.

**Key Advantages:**
- **Single source of truth** - All types defined together in one location
- **Atomic versioning** - The entire profile versions as a cohesive unit
- **Simpler deployment** - One file to distribute and manage
- **Schema Registry friendly** - Register once with all types included
- **No dependency management** - All types available without imports
- **Cohesive profile** - SecurityAnalysisResult types form a tightly coupled set

**Structure:**
The schema file contains an array of type definitions, ordered by dependency:
```json
[
  { 
    "type": "enum", 
    "name": "UnitMultiplier", 
    // additional definition here
    },
  { "type": "enum", 
    "name": "UnitSymbol", 
    // additional definition here
  },
  { "type": "record", 
    "name": "BaseCasePowerFlowResult", 
    // additional definition here
  },
  { "type": "record", 
    "name": "ContingencyPowerFlowResult", 
    // additional definition here
  },
  { "type": "record", 
    "name": "RemedialActionApplied", 
    // additional definition here
  },
  { "type": "record", 
    "name": "SecurityAnalysisResult", 
    // additional definition here
  }
]
```

**Alternative Approach:**
Multiple files (one type per file) offer better modularity for reusable type libraries across projects, but add complexity in dependency management and deployment. For CIM profiles where types are derived as unique subsets specific to each profile, the single-file approach provides the optimal balance of simplicity and functionality. For IEC-style CIM profiles where each profile is a self-contained message definition, the single-file approach is the recommended pattern.

### 7.2. Namespacing

**Decision: Hierarchical Namespace Structure with Sub-packages**

For this case study, Avro namespaces follow a hierarchical structure based on reverse-DNS naming convention with logical sub-packages to organize different categories of types.

**Key Advantages:**
- **Clear type organization** - Types grouped by their semantic purpose
- **Avoids naming collisions** - Fully qualified names prevent conflicts
- **Code generation friendly** - Maps naturally to Java packages and Python modules
- **Standards alignment** - Follows Apache Avro best practices
- **Extensibility** - Easy to add new type categories without restructuring
- **Readable qualified names** - Clear provenance of each type

**Namespace Structure:**
The schema derives unique namespaces in the following manner:

```
Profile baseURI:   https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4#
                   ↓
Extract domain:    ap-voc.cim4.eu
                   ↓
Reverse DNS:       eu.cim4.ap_voc
                   ↓
Add profile name:  eu.cim4.ap_voc + securityanalysisresult (lowercase)
                   ↓
Base namespace:    eu.cim4.ap_voc.securityanalysisresult
                   ↓
Add CIM package:   eu.cim4.ap_voc.securityanalysisresult + .domain
                   eu.cim4.ap_voc.securityanalysisresult + .extsecurityanalysisresult

Final namespaces:
  Root:            eu.cim4.ap_voc.securityanalysisresult
  Domain types:    eu.cim4.ap_voc.securityanalysisresult.domain
  Extension types: eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult
```

**Type Organization:**
- **Domain derived namespace elements** (`eu.cim4.ap_voc`): The first namespace elements are derived from the domain defined in the profile URI (i.e. `https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4#`).
- **Profile namespace element** (`.securityanalysisresult`): Next, a profile-specific namespace element is then appended to the namespace. By convention, the document container/envelope is used (i.e. SecurityAnalysisResult) and thus produces a 'root' namespace unique solely to the profile.
- **Package namespace element** (`.domain` or `.extsecurityanalysisresult`): Finally, for each enumeration or class defined in the Avro schema, a namespace element derived from the CIM UML package that contains it is appended.

Applying the above conventions produces the following fully qualified type names:
```
eu.cim4.ap_voc.securityanalysisresult.domain.UnitMultiplier
eu.cim4.ap_voc.securityanalysisresult.domain.UnitSymbol
eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult
eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult
eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.RemedialActionApplied
eu.cim4.ap_voc.securityanalysisresult.SecurityAnalysisResult
```

**Alternative Approach:**
A flat namespace (all types in `eu.cim4.ap_voc.securityanalysisresult`) would be simpler but loses the organizational benefits of sub-packages. For profiles with multiple type categories (domain types, extensions, compound types), the hierarchical approach provides better structure and maintainability. Avro's limitation of type-level namespaces (not field-level like XML) means that field origins must be documented through the custom `modelReference` annotation described next rather than field-level namespace prefixes.

### 7.3. Canonical Model Traceability Within Avro Schemas

Traceability back to the canonical CIM is essential in Apache Avro schema mappings. The IEC 62361-100 "CIM profile to XML Schema mapping" specification addresses this through W3C semantic annotations, which provide extension attributes for WSDL and XSD schemas; however, no comparable mechanism exists for Avro schemas.

For the Avro schemas generated by **CIMTool** tracability has been implemented using a custom `modelReference` annotation as a conceptual derivation of W3C semantic annotations. These `modelReference` annotations identify the origin of each type, attribute, or association within the Avro schema and have a value expressed as an absolute URI that corresponds to their namespace URI in the CIM. 

Finally, important to note is that Apache Avro treats unknown attributes as metadata and preserves them in the schema without affecting serialization or deserialization. This allows the `modelReference` annotations to provide documentation and traceability while being safely ignored by Avro's binary encoding and decoding processes.

**Usage Examples:**

The `modelReference` annotation appears at different levels within the Avro schema:

**Enum Type Level:**
```json
{
  "type": "enum",
  "name": "UnitMultiplier",
  "namespace": "eu.cim4.ap_voc.securityanalysisresult.domain",
  "modelReference": "http://iec.ch/TC57/CIM100#UnitMultiplier",
  "symbols": ["k", "m", "none"]
}
```

**Record Type Level:**
```json
{
  "type": "record",
  "name": "BaseCasePowerFlowResult",
  "namespace": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult",
  "modelReference": "https://cim4.eu/ns/nc#BaseCasePowerFlowResult",
  "fields": [
    // additional definition here
  ]
}
```

**Primitive Field (Attribute):**
```json
{
  "name": "isViolation",
  "type": "boolean",
  "doc": "True if the power flow result is violating the associated operational limit.",
  "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.isViolation"
}
```

**Association Field (Reference):**
```json
{
  "name": "ACDCTerminal",
  "type": "string",
  "doc": "ACDC terminal where the powerflow result is located.",
  "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.ACDCTerminal"
}
```

**Union Field (Optional Association):**
```json
{
  "name": "OperationalLimit",
  "type": ["null", "string"],
  "default": null,
  "doc": "The operational limit that has this limit violation.",
  "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.OperationalLimit"
}
```

### 7.4. Class Inheritance: Flattened Record Structure

**Decision: Combined Flattening and Union Pattern**

For this case study, CIM class inheritance is handled using a two-part strategy: 

1. parent class fields are flattened by duplicating them into each concrete child record type, and 
2. polymorphic associations to the abstract parent are represented using Avro union types. 

This approach addresses Avro's lack of native inheritance while preserving type distinctions for polymorphic relationships.

**Key Advantages:**
- **No Avro inheritance complexity** - Avoids Avro's lack of native inheritance support
- **Self-contained records** - Each concrete record type is complete and standalone
- **Preserved polymorphism** - Union types maintain the ability to reference either concrete subclass
- **Type safety maintained** - Each concrete type is fully specified with proper union discrimination
- **Clear field provenance** - All fields explicitly defined with their modelReference
- **Simpler code generation** - Generated classes are flat but can still handle polymorphic fields

**Inheritance Structure:**

In the CIM model, `PowerFlowResult` is an abstract parent class with common fields, and two concrete subclasses:

```
PowerFlowResult (abstract)
  ├─ Common fields: atTime, isViolation, absoluteValue, value, valueA, valueAngle,
  │                  valueV, valueVA, valueVAR, valueW, ACDCTerminal,
  │                  OperationalLimit, ReportedByRegion
  │
  ├─ BaseCasePowerFlowResult (concrete)
  │    └─ Inherits all PowerFlowResult fields
  │
  └─ ContingencyPowerFlowResult (concrete)
       ├─ Inherits all PowerFlowResult fields
       └─ Adds: Contingency (required association)
```

**Part 1: Field Flattening**

The abstract parent (`PowerFlowResult`) is not defined as an Avro type. Instead, all parent fields are copied into each concrete child:

```json
{
  "type": "record",
  "name": "BaseCasePowerFlowResult",
  "namespace": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult",
  "fields": [
    {
      "name": "absoluteValue", 
      "type": ["null", "float"], 
      // additional definition here
    },
    {
      "name": "atTime", 
      "type": {"type": "long", "logicalType": "timestamp-millis"}, 
      // additional definition here
    },
    {
      "name": "isViolation", 
      "type": "boolean", 
      // additional definition here
    },
    // all remaining PowerFlowResult fields duplicated here
  ]
}

{
  "type": "record",
  "name": "ContingencyPowerFlowResult",
  "namespace": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult",
  "fields": [
   {
      "name": "absoluteValue", 
      "type": ["null", "float"], 
      // additional definition here
    },
    {
      "name": "atTime", 
      "type": {"type": "long", "logicalType": "timestamp-millis"}, 
      // additional definition here
    },
    {
      "name": "isViolation", 
      "type": "boolean", 
      // additional definition here
    },
    // additional definition here all remaining PowerFlowResult fields duplicated here
    {
      "name": "Contingency", 
      "type": "string", 
      // additional definition here
    }  // Additional Association field
  ]
}
```

**Part 2: Polymorphic Association via Union**

When another class (like `RemedialActionApplied`) has an association to the abstract `PowerFlowResult` class, this is represented as an Avro union of the concrete types:

```json
{
  "type": "record",
  "name": "RemedialActionApplied",
  "namespace": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult",
  "fields": [
    {
      "name": "mRID", 
      "type": "string", 
      // additional definition here
    },
    {
      "name": "PowerFlowResult",
      "type": [
        "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult",
        "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult"
      ],
      "doc": "Power flow result (either base case or contingency)",
      "modelReference": "https://cim4.eu/ns/nc#RemedialActionApplied.PowerFlowResult"
    }
  ]
}
```

**Field Count:**
- PowerFlowResult (parent): 13 fields
- BaseCasePowerFlowResult: 13 fields (all from parent)
- ContingencyPowerFlowResult: 14 fields (13 from parent + 1 additional)

**Why This Combined Approach:**

The flattening strategy alone would not support polymorphic associations - there would be no way to represent "a PowerFlowResult that could be either subclass." The union type alone would not eliminate the need for the abstract parent definition in Avro. By combining both techniques:

1. **Flattening** makes each concrete type complete and self-contained
2. **Unions** enable polymorphic associations without requiring an abstract parent type in Avro

This pattern is the standard approach for CIM to Avro mappings and is consistent with how other schema-based serialization formats (like Protocol Buffers with `oneof`) handle inheritance and polymorphism.

### 7.5. Cross Profile Associations: Reference Strategy

**Decision: String-Based Identifiers Reference Pattern**

For this case study, associations to CIM objects that reference abstract classes with no parent classes, child classes, fields, or associations are represented as string identifiers (mRIDs) rather than embedding nested objects. This approach is specifically applied to cross-profile references where the referenced class serves only as a placeholder in the profile. Associations to classes with defined properties within the profile follow different patterns (such as embedded compound types or polymorphic unions as discussed in other sections). The string-based reference approach treats these minimal class references as external pointers, maintaining clean separation between the message content and objects defined in other CIM profiles.

**Key Advantages:**
- **Smaller message sizes** - Only identifiers transmitted, not full object graphs
- **No circular reference issues** - Prevents infinite recursion in bidirectional associations
- **Clean model separation** - Results messages reference but don't contain network model
- **Follows CIM messaging patterns** - Standard approach in CIM exchange profiles
- **Schema simplicity** - No complex nested structures to maintain
- **Efficient serialization** - Minimal data duplication across messages

**Profile Constraints:**

The OWL profile declares abstract reference classes but defines no properties for them:

```xml
<rdf:Description rdf:about="#ACDCTerminal">
  <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#Class"/>
  <rdfs:subClassOf rdf:resource="http://iec.ch/TC57/CIM100#ACDCTerminal"/>
</rdf:Description>

<rdf:Description rdf:about="#Contingency">
  <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#Class"/>
  <rdfs:subClassOf rdf:resource="https://cim4.eu/ns/nc#Contingency"/>
</rdf:Description>
```

These classes are declared as part of the profile but have:
- **No properties defined** - Only the class itself is referenced
- **No parent class properties inherited** - They serve as placeholders
- **"Limited to OWL" principle** - Since no properties are specified, only the reference (mRID) is utilied.

**Reference Classes in the SecurityAnalysisResult Profile:**
- `ACDCTerminal` - Terminal where power flow measurements occur
- `Contingency` - Contingency scenario being analyzed
- `OperationalLimit` - Limit being monitored or violated
- `Region` - Geographic or organizational region reporting results
- `RemedialAction` - Remedial action being applied
- `Stage` - Stage of a remedial action scheme

**Avro Schema Representation:**

References are mapped to simple string fields containing the mRID:

```json
{
  "name": "ACDCTerminal",
  "type": "string",
  "doc": "ACDC terminal where the powerflow result is located. Note that the value of this field is the identifier (e.g. mRID) used to reference the ACDCTerminal external to this profile.",
  "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.ACDCTerminal"
}
```

For optional references, a union with null is used:

```json
{
  "name": "OperationalLimit",
  "type": ["null", "string"],
  "default": null,
  "doc": "The operational limit that has this limit violation. Note that the value of this field is the identifier (e.g. mRID) used to reference the OperationalLimit external to this profile.",
  "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.OperationalLimit"
}
```

**Example Message Fragment:**

```json
{
  "BaseCasePowerFlowResult": [
    {
      "atTime": 1704067200000,
      "isViolation": true,
      "ACDCTerminal": "550e8400-e29b-41d4-a716-446655440000",
      "OperationalLimit": "123e4567-e89b-12d3-a456-426614174000",
      "ReportedByRegion": "fa3e25f7-1d9b-e2d3-c456-4267141740e2"
    }
  ]
}
```

The mRID strings point to objects defined in separate network model messages or maintained in a network model database.

**Alternative Approach:**

Embedding complete nested objects would provide self-contained messages but at significant cost:

- **Message bloat** - Each terminal/limit/contingency fully embedded
- **Data duplication** - Same objects repeated across multiple results
- **Circular references** - Bidirectional associations create infinite nesting
- **Schema complexity** - Deep nested structures difficult to maintain
- **Not CIM standard** - Deviates from established CIM messaging patterns

The string reference pattern is universally used in CIM exchange profiles (XML, RDF, JSON-LD) and is the appropriate choice for Avro mappings. External systems resolve references by correlating mRIDs across message types or querying a network model repository.

### 7.6. Mapping CIM Compound Types

In the UML, compound types appear as follows:

![Compound Types](https://img.plantuml.biz/plantuml/svg/bLLDRzim3BtxLn38eOTXWTCcmLe4HHF7MdVP1hItOGVLPXAXiXH8zDfixN-VQCTfNBl5PbralKVoyKFsbIVfg2nqf755G1QSMcy8SkKm8sLD59s0tV8Eraxc2Wt1dSpkrywM9cS3hufIIq98vp2Q3X-3hGpKmaV-Nfcj35jO72mIgws3WLZm-ZXOJfUdISf1hbMEa6dfFQZC6XzjgWFbCetr7eijxQ10sjZSiydjHdbWo0Dj42IHKtTbK58xU0bVkPqty9U0tgZV2F4HPUwSVTYB6q_6q_7a2FGRcTi7PTP0haGEpPZXWrKmC9RmE1o_mvy8uAIJzohc5Q0kagq-uHaZrkNesb80X3MqFpkhdltPLrbW4A-atXzKcOrE3uv2aR9xWw6iet1sqyxaENfTfKftjrsfICJvqq2SfykK_HoAUQHhCX_7uwIjDQwUHXhjDCATrJah8NHeQRV5Flerog3zVQJbhhzgVINWlkpha3uI6i-pNUKUviXNFdDSAQEgyUyKOpPxPNjv2ODKy7eQQBuyOVXtJPpKb0KwoQx-pJc-65IEpuL89q-3PIpcoJIgVOPHTVw_92ERi3yXn9a8_eH8ucHntAlPB6r8XrftkznbXH6gRkSBuW-TZQ9wfcx88T8ypnqtOJPBRB5bTO7CumOVG4kpBkKQBvZBu_aZ7Wv7FsjmQnVsyGqaVBzqOlD-mm1IwHaA1VMl89QqH99AN-m5rcyj6ja7qv1k3_k6yjmaU0JrPPUz9J0BF7AxNio4czsVLVrbVZDZ_EQ-E_jBcQnZ_zpko3kIPTsipsZ2DEhUM6tAQGzdo8k3Ng79-ISJ_GK0)

**Decision: Named Record Types with Nested Message Data**

Compound types in CIM represent value objects with no independent identity that always appear nested within their parent objects in message instances. While the base SecurityAnalysisResult profile does not include compound types, the EXTENDED version demonstrates their mapping to Avro for validation and edge case testing. When compound types are present in a profile, they are defined in the Avro schema as separate named record types (not anonymous inline types), but in actual JSON messages the compound data appears nested within the containing parent object rather than referenced by identifier. For further context on the design pattern, refer to the [AVRO Handling of CIM Compound Types](compound_types_guide.md) guide.

**Key Advantages:**
- **Semantic correctness** - Value objects embedded as part of their parent
- **No identity management** - Compound types have no mRID, appropriately modeled as nested structures
- **Type reusability** - Named record types can be referenced by multiple parent classes
- **Schema clarity** - Clear distinction between entities (with mRID) and value objects (embedded)
- **Data integrity** - Compound data travels with its parent, ensuring consistency
- **Standard CIM pattern** - Consistent with compound type treatment across all CIM serializations

**Compound Type Characteristics:**

In a CIMTool OWL file, compound types are marked with the compound stereotype:

```xml
<rdf:Description rdf:about="#StreetAddress">
  <uml:hasStereotype rdf:resource="http://langdale.com.au/2005/UML#compound"/>
  <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#Class"/>
  <rdfs:subClassOf rdf:resource="http://iec.ch/TC57/CIM100#StreetAddress"/>
</rdf:Description>
```

These are **value objects** that:
- Have no independent identity (no mRID)
- Are always embedded within their parent object
- Cannot exist independently or be referenced by other objects
- Represent structured data that is intrinsic to the parent

**Example Compound Type Hierarchy (from EXTENDED profile):**

The EXTENDED profile includes:
```
Location (concrete class with mRID)
  └─ mainAddress: StreetAddress (compound, required)
      ├─ postalCode: string (optional)
      ├─ streetDetail: StreetDetail (compound, required)
      │   ├─ buildingName: string
      │   ├─ code: string
      │   └─ name: string
      └─ townDetail: TownDetail (compound, optional)
          ├─ code: string
          ├─ country: string
          └─ name: string
```

**Avro Mapping:**

Compound types are defined as separate named record types, then embedded by reference in their parent:

```json
[
  {
    "type": "record",
    "name": "StreetDetail",
    "namespace": "eu.cim4.ap_voc.securityanalysisresult.location",
    "fields": [
      {"name": "buildingName", "type": ["null", "string"], "default": null},
      {"name": "code", "type": ["null", "string"], "default": null},
      {"name": "name", "type": ["null", "string"], "default": null}
    ]
  },
  {
    "type": "record",
    "name": "TownDetail",
    "namespace": "eu.cim4.ap_voc.securityanalysisresult.location",
    "fields": [
      {"name": "code", "type": ["null", "string"], "default": null},
      {"name": "country", "type": ["null", "string"], "default": null},
      {"name": "name", "type": ["null", "string"], "default": null}
    ]
  },
  {
    "type": "record",
    "name": "StreetAddress",
    "namespace": "eu.cim4.ap_voc.securityanalysisresult.location",
    "fields": [
      {"name": "postalCode", "type": ["null", "string"], "default": null},
      {"name": "streetDetail", "type": "StreetDetail"},
      {"name": "townDetail", "type": ["null", "TownDetail"], "default": null}
    ]
  },
  {
    "type": "record",
    "name": "Location",
    "namespace": "eu.cim4.ap_voc.securityanalysisresult.location",
    "fields": [
      {"name": "mRID", "type": "string"},
      {"name": "direction", "type": "string"},
      {"name": "mainAddress", "type": "StreetAddress"}
    ]
  }
]
```

**Key Mapping Rules:**
- **No mRID field** - Compound types have no identity
- **Named record types** - Defined once, reusable across multiple parents
- **Embedded by type reference** - Parent field references the compound record type name
- **Cardinality via unions** - Optional compounds use `["null", "CompoundType"]`
- **Definition order** - Compound types defined before their containing types

**Example Message Fragment:**

```json
{
  "Location": [
    {
      "mRID": "location-001",
      "direction": "North side of intersection",
      "mainAddress": {
        "postalCode": "94102",
        "streetDetail": {
          "buildingName": "Power Station Alpha",
          "code": "BLDG-123",
          "name": "Main Street"
        },
        "townDetail": {
          "code": "SF",
          "country": "USA",
          "name": "San Francisco"
        }
      }
    }
  ]
}
```

**Alternative Approach:**

Inline anonymous record types could be used instead of named types, but this prevents reusability and increases schema verbosity. By defining compound types as named records, they can be referenced by multiple parent classes if needed, maintaining DRY principles. This approach is demonstrated in the EXTENDED profile to validate the XSLT transformation's capability to handle compound types when they appear in future CIM to Avro profile mappings.

### 7.7. Date Datatypes and Apache Avro Logical Types

**Decision: timestamp-millis Logical Type**

For this case study, date and time values are represented using Avro's `timestamp-millis` logical type rather than ISO 8601 strings. This provides compact binary encoding, type-safe code generation, and efficient timestamp operations while maintaining millisecond precision sufficient for power system analysis scenarios.

**Key Advantages:**
- **Compact representation** - 8 bytes vs ~24 bytes for ISO 8601 strings
- **Type-safe code generation** - Generates proper date/time types (Instant, datetime) in target languages
- **Fast comparisons** - Numeric comparison instead of string parsing
- **Efficient arithmetic** - Direct millisecond addition/subtraction
- **No parsing overhead** - Binary encoding eliminates deserialization parsing
- **Standard Avro practice** - Widely supported across Avro implementations

**Logical Type Concept:**

Avro logical types add semantic meaning to primitive types without changing the binary encoding. They provide hints to code generators about how to interpret the underlying primitive data:

```
Primitive Type (storage) + Logical Type (interpretation) = Meaningful Data
     long (8 bytes)      +   timestamp-millis          =   Timestamp
```

**timestamp-millis Specification:**
- **Underlying type:** `long` (64-bit signed integer)
- **Interpretation:** Milliseconds since Unix epoch (1970-01-01T00:00:00.000Z)
- **Precision:** Millisecond (0.001 seconds)
- **Range:** ±292 million years from epoch
- **Timezone:** Values represent instants in UTC

**Avro Schema Definition:**

```json
{
  "name": "atTime",
  "type": {
    "type": "long",
    "logicalType": "timestamp-millis"
  },
  "doc": "The date and time of the scenario time that was studied and at which the limit violation occurred.",
  "modelReference": "https://cim4.eu/ns/nc#PowerFlowResult.atTime"
}
```

**Example Values:**

| Timestamp Value | ISO 8601 Equivalent | Description |
|----------------|---------------------|-------------|
| `0` | 1970-01-01T00:00:00.000Z | Unix epoch |
| `1704067200000` | 2024-01-01T00:00:00.000Z | New Year 2024 |
| `1704070800000` | 2024-01-01T01:00:00.000Z | One hour later |
| `1735689600000` | 2025-01-01T00:00:00.000Z | New Year 2025 |

**JSON Message Example:**

```json
{
  "BaseCasePowerFlowResult": [
    {
      "atTime": 1704067200000,
      "isViolation": true,
      "absoluteValue": 1150.0,
      "ACDCTerminal": "550e8400-e29b-41d4-a716-446655440000"
    }
  ]
}
```

**Code Generation:**

The logical type enables proper type generation in target languages:

**Java:**
```java
public class BaseCasePowerFlowResult {
    private java.time.Instant atTime;  // Not long!
    private boolean isViolation;
    // additional definition here
}
```

**Python:**
```python
from datetime import datetime

result = {
    'atTime': datetime(2024, 1, 1, 0, 0, 0),  # Automatically converted
    'isViolation': True,
    'absoluteValue': 1150.0
}
```

**Other Standard Logical Types:**

While this profile uses only `timestamp-millis`, Avro supports several other logical types:

| Logical Type | Base Type | Use Case |
|--------------|-----------|----------|
| `date` | `int` | Calendar dates (days since epoch) |
| `time-millis` | `int` | Time of day (milliseconds since midnight) |
| `time-micros` | `long` | Time of day (microseconds since midnight) |
| `timestamp-micros` | `long` | Timestamp with microsecond precision |
| `decimal` | `bytes` or `fixed` | Arbitrary precision decimals |
| `uuid` | `string` or `fixed[16]` | Universally unique identifiers |

**Alternative Approach:**

ISO 8601 strings (`"2024-01-01T00:00:00Z"`) could be used instead, offering:
- **Human readability** - Easy to debug and inspect
- **Variable precision** - Can represent seconds, millis, micros, nanos
- **Timezone information** - Can include offset in string format

However, for binary streaming protocols like Kafka where efficiency matters, the logical type approach is preferred. ISO 8601 strings are more suitable for:
- REST APIs where readability is prioritized
- Interchange with systems not supporting Avro
- Configuration files and human-edited data

**Logical Types Comprehensive Guide:**

For detailed information on all Avro logical types, see the [Apache Avro Logical Types - Complete Guide](apache_avro_logical_types_guide.md).

### 7.8. Cardinality Mapping

**Decision: Union-Based Optional Fields**

For this case study, OWL cardinality constraints are mapped to Avro type unions to distinguish required from optional fields. Fields with minimum cardinality of 1 are represented as their direct type, while fields with minimum cardinality of 0 or unspecified are represented as unions with null, enabling proper optional field semantics.

**Key Advantages:**
- **Clear optionality** - Type system enforces required vs optional distinction
- **Explicit defaults** - Optional fields can specify default null values
- **Schema evolution friendly** - Adding optional fields is backward compatible
- **Type safety** - Code generators produce proper nullable types
- **Validation support** - Schema validators can check required field presence
- **Self-documenting** - Schema clearly shows which fields are mandatory

**Cardinality Mapping Rules:**

**Rule 1: Required Fields (minCardinality = 1)**

OWL constraint:
```xml
<owl:minCardinality rdf:datatype="http://www.w3.org/2001/XMLSchema#int">1</owl:minCardinality>
<owl:onProperty rdf:resource="https://cim4.eu/ns/nc#PowerFlowResult.isViolation"/>
```

Avro mapping:
```json
{
  "name": "isViolation",
  "type": "boolean",
  "doc": "True if the power flow result is violating the associated operational limit."
}
```

**Rule 2: Optional Fields (minCardinality = 0 or unspecified)**

OWL constraint (implicit - no minCardinality specified):
```xml
<owl:onProperty rdf:resource="https://cim4.eu/ns/nc#PowerFlowResult.absoluteValue"/>
```

Avro mapping:
```json
{
  "name": "absoluteValue",
  "type": ["null", "float"],
  "default": null,
  "doc": "Absolute value from a power flow calculation on a given terminal."
}
```

**Cardinality Summary Table:**

| OWL Constraint | Avro Type | Default | Description |
|----------------|-----------|---------|-------------|
| `minCardinality = 1` | `"type": "TypeName"` | None | Required, must be present |
| `minCardinality = 0` | `"type": ["null", "TypeName"]` | `null` | Optional, can be omitted |
| No constraint | `"type": ["null", "TypeName"]` | `null` | Optional, can be omitted |
| `minCardinality = 1`, Array | `"type": {"type": "array", "items": "T"}` | None | Required array (can be empty) |
| `minCardinality = 0`, Array | `"type": ["null", {"type": "array", "items": "T"}]` | `"default": []` | Optional array |

**maxCardinality Handling:**

Avro does not provide native support for maximum cardinality constraints. The maxCardinality constraints from OWL are handled differently depending on the context:

**Rule 3: Single-Valued Fields (maxCardinality = 1)**

When a field has `maxCardinality = 1`, it is represented as a single value (not an array):

OWL constraint:
```xml
<owl:maxCardinality rdf:datatype="http://www.w3.org/2001/XMLSchema#int">1</owl:maxCardinality>
<owl:onProperty rdf:resource="https://cim4.eu/ns/nc#PowerFlowResult.ACDCTerminal"/>
```

Avro mapping:
```json
{
  "name": "ACDCTerminal",
  "type": "string"
}
```

**Rule 4: Multi-Valued Fields (maxCardinality > 1 or unbounded)**

When a field can have multiple values (maxCardinality > 1 or no maxCardinality specified), it is represented as an Avro array:

OWL constraint (unbounded):
```xml
<owl:onProperty rdf:resource="https://cim4.eu/ns/nc#SecurityAnalysisResult.BaseCasePowerFlowResult"/>
```

Avro mapping:
```json
{
  "name": "BaseCasePowerFlowResult",
  "type": {
    "type": "array",
    "items": "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult"
  }
}
```

**Rule 5: Message Root Cardinality**

For some message root classes (e.g. `RemedialActionApplied`) in CIM profiles cardinality may be defined with cardinality such as:
```xml
<uml:hasMinCardinality>1</uml:hasMinCardinality>
<uml:hasMaxCardinality>1</uml:hasMaxCardinality>
```

This constraint (exactly one root element) is enforced by the message structure itself - the Avro schema defines a single container record with arrays for each concrete class. For SecurityAnalysisResult, the container structure ensures proper message boundaries.

**Validation Limitations:**

Avro schemas cannot enforce specific maximum array lengths (e.g., maxCardinality = 5). These constraints must be validated at the application level:

```java
// Application-level validation example
if (result.getOperationalLimits().size() > MAX_LIMITS) {
    throw new ValidationException("Exceeds maximum cardinality");
}
```

**Specific maxCardinality Values (e.g., maxCardinality = 4):**

When an OWL profile specifies a specific maxCardinality value greater than 1 (e.g., 2, 3, 5), Avro has no native schema construct to enforce this limit. The field is still represented as an unbounded array:

OWL constraint:
```xml
<owl:minCardinality rdf:datatype="http://www.w3.org/2001/XMLSchema#int">2</owl:minCardinality>
<owl:maxCardinality rdf:datatype="http://www.w3.org/2001/XMLSchema#int">4</owl:maxCardinality>
<owl:onProperty rdf:resource="http://iec.ch/TC57/CIM100#EquipmentContainer.Equipments"/>
```

Example Avro mapping (same as unbounded) but with custom cardinality annotations (i.e. `minCardDoc`, `maxCardDoc`) to document when in field description:
```json
{
    "name": "Equipments",
    "type": {
        "type": "array",
        "items": [
              "eu.cim4.ap_voc.securityanalysisresult.wires.Breaker",
              "eu.cim4.ap_voc.securityanalysisresult.wires.Cut",
              "eu.cim4.ap_voc.securityanalysisresult.wires.DisconnectingCircuitBreaker",
              "eu.cim4.ap_voc.securityanalysisresult.wires.Disconnector",
              "eu.cim4.ap_voc.securityanalysisresult.wires.GroundDisconnector",
              "eu.cim4.ap_voc.securityanalysisresult.wires.Recloser"
        ]
    },
    "doc": "Contained equipment.",
    "modelReference": "http://iec.ch/TC57/CIM100#EquipmentContainer.Equipments",
    "minCardDoc": "[min cardinality = 2] Application level validation will be required to ensure the array contains at least 2 items.",
    "maxCardDoc": "[max cardinality = 4] Application level validation will be required to ensure the array contains at most 4 items."
}
```

The specific maximum value (4 in this example) cannot be expressed in the Avro schema itself. Implementation options include:

1. **Application-level validation:**
```java
if (object.getSomeProperty().size() < 2 || object.getSomeProperty().size() > 4) {
    throw new CardinalityViolationException("EquipmentContainer.Equipments association must have between 2 and 4 devices.");
}
```

2. **External validation framework:**
- JSON Schema validators with custom rules
- Apache Avro validators with custom plugins
- Business logic validation layers

While this approach requires external validation, it maintains compatibility with Avro's design philosophy of schema evolution and keeps the schema simple and declarative. Most real-world systems implement such business rule validations in their service layers rather than in serialization schemas.

**Combined Cardinality Examples:**

**Required single-valued (min=1, max=1):**
```json
{"name": "mRID", "type": "string"}
```

**Optional single-valued (min=0, max=1):**
```json
{"name": "OperationalLimit", "type": ["null", "string"], "default": null}
```

**Required multi-valued (min=1, max=unbounded):**
```json
{
  "name": "BaseCasePowerFlowResult",
  "type": {"type": "array", "items": "BaseCasePowerFlowResult"}
}
```

**Optional multi-valued (min=0, max=unbounded):**
```json
{
  "name": "OptionalResults",
  "type": ["null", {"type": "array", "items": "ResultType"}],
  "default": null
}
```

Note: In practice, most CIM profiles use unbounded maxCardinality for collections, relying on business rules rather than schema constraints to limit array sizes.

**Examples from SecurityAnalysisResult Schema:**

**Required Primitive Field:**
```json
{
  "name": "isViolation",
  "type": "boolean"
}
```

**Optional Primitive Field:**
```json
{
  "name": "value",
  "type": ["null", "float"],
  "default": null
}
```

**Required Reference:**
```json
{
  "name": "ACDCTerminal",
  "type": "string"
}
```

**Optional Reference:**
```json
{
  "name": "ReportedByRegion",
  "type": ["null", "string"],
  "default": null
}
```

**Required Association (via union of concrete types):**
```json
{
  "name": "PowerFlowResult",
  "type": [
    "BaseCasePowerFlowResult",
    "ContingencyPowerFlowResult"
  ]
}
```

**Union Type Ordering:**

For optional fields using unions, `null` should always appear first in the union type array:

**Correct:**
```json
"type": ["null", "string"]
```

**Incorrect:**
```json
"type": ["string", "null"]
```

This convention ensures consistent default value behavior and aligns with Avro best practices.

**Message Examples:**

**Required field present:**
```json
{
  "atTime": 1704067200000,
  "isViolation": true,
  "ACDCTerminal": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Optional field with value:**
```json
{
  "atTime": 1704067200000,
  "isViolation": false,
  "absoluteValue": 950.0,
  "value": 95.0,
  "ACDCTerminal": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Optional field omitted (equivalent to null):**
```json
{
  "atTime": 1704067200000,
  "isViolation": false,
  "ACDCTerminal": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Optional field explicitly null:**
```json
{
  "atTime": 1704067200000,
  "isViolation": false,
  "absoluteValue": null,
  "value": null,
  "ACDCTerminal": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Alternative Approach:**

All fields could be made optional by default (always using unions with null), but this loses important semantic information from the CIM model about which fields are truly required. By preserving cardinality constraints in the Avro schema, we:
- Maintain model fidelity with the OWL profile
- Enable validation at serialization/deserialization
- Provide better documentation for schema consumers
- Generate more type-safe code with proper non-null types

Some serialization formats (like Protocol Buffers v3) make all fields implicitly optional, but Avro's union-based approach provides explicit control while maintaining schema evolution compatibility.

### 7.9. Document Element Root Fields

**Decision: Topological Analysis to Identify True Message Roots**

Within the Avro schemas, the document container (top-level message record) includes only those Root classes that are never referenced by other classes in the profile. This ensures a clean, logical message structure where referenced types appear as nested fields rather than redundant top-level arrays.

**Key Advantages:**
- **Correct message topology** - Only true roots appear at the top level
- **Avoids redundancy** - Referenced types don't appear both nested and at top level
- **Self-documenting structure** - Message structure reflects the actual data relationships
- **Automatic identification** - Topological analysis removes manual decision-making
- **Handles inheritance** - Properly accounts for references through abstract classes
- **Scalable approach** - Works for any profile complexity without manual intervention


Container structure:
```json
{
  "type": "record",
  "name": "SecurityAnalysisResult",
  "namespace": "eu.cim4.ap_voc.securityanalysisresult",
  "fields": [
    {
      "name": "RemedialActionApplied",
      "type": {"type": "array", "items": "RemedialActionApplied"}
    }
  ]
}
```

The topological analysis approach provides a generic, scalable solution that works correctly for any CIM profile structure without requiring manual intervention or profile-specific customization. The algorithm automatically adapts to changes in the profile as classes and associations are added or removed.

## 8. Example SAR Profile JSON Payloads

### 8.1. SecurityAnalysisResult JSON

The following examples demonstrate valid JSON messages for the SecurityAnalysisResult schema. Complete examples with validation documentation are available in the [`/example-payloads`](https://github.com/cimug-org/Avro4CIM/tree/main/case-study/example-payloads) directory.

**Example 1: Minimal Valid Message**
```json
{
  "RemedialActionApplied": []
}
```

**Example 2: Complete Security Analysis Payload**
```json
{
  "RemedialActionApplied": [
    {
      "mRID": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
      "PowerFlowResult": {
        "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult": {
          "absoluteValue": 850.0,
          "atTime": 1704067200000,
          "isViolation": false,
          "value": 85.0,
          "valueA": 850.0,
          "valueAngle": null,
          "valueV": null,
          "valueVA": null,
          "valueVAR": null,
          "valueW": null,
          "ACDCTerminal": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
          "OperationalLimit": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
          "ReportedByRegion": "10Y-DE-ENTSOE-1"
        }
      },
      "RemedialAction": "b2c3d4e5-f6a7-4b89-9c0d-1e2f3a4b5c6d",
      "StageForRemedialActionScheme": null
    },
    {
      "mRID": "c3d4e5f6-a7b8-4c9d-0e1f-2a3b4c5d6e7f",
      "PowerFlowResult": {
        "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.BaseCasePowerFlowResult": {
          "absoluteValue": 1050.0,
          "atTime": 1704067200000,
          "isViolation": true,
          "value": 105.0,
          "valueA": 1050.0,
          "valueAngle": null,
          "valueV": null,
          "valueVA": null,
          "valueVAR": null,
          "valueW": null,
          "ACDCTerminal": "d4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f8a",
          "OperationalLimit": "e5f6a7b8-c9d0-4e1f-2a3b-4c5d6e7f8a9b",
          "ReportedByRegion": "10Y-FR-ENTSOE-1"
        }
      },
      "RemedialAction": "f6a7b8c9-d0e1-4f2a-3b4c-5d6e7f8a9b0c",
      "StageForRemedialActionScheme": null
    },
    {
      "mRID": "a7b8c9d0-e1f2-4a3b-4c5d-6e7f8a9b0c1d",
      "PowerFlowResult": {
        "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult": {
          "absoluteValue": 1450.0,
          "atTime": 1704067200000,
          "isViolation": true,
          "value": 145.0,
          "valueA": 1450.0,
          "valueAngle": null,
          "valueV": null,
          "valueVA": null,
          "valueVAR": null,
          "valueW": null,
          "ACDCTerminal": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
          "OperationalLimit": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
          "ReportedByRegion": "10Y-DE-ENTSOE-1",
          "Contingency": "b8c9d0e1-f2a3-4b4c-5d6e-7f8a9b0c1d2e"
        }
      },
      "RemedialAction": "c9d0e1f2-a3b4-4c5d-6e7f-8a9b0c1d2e3f",
      "StageForRemedialActionScheme": null
    },
    {
      "mRID": "d0e1f2a3-b4c5-4d6e-7f8a-9b0c1d2e3f4a",
      "PowerFlowResult": {
        "eu.cim4.ap_voc.securityanalysisresult.extsecurityanalysisresult.ContingencyPowerFlowResult": {
          "absoluteValue": 920.0,
          "atTime": 1704067200000,
          "isViolation": false,
          "value": 92.0,
          "valueA": 920.0,
          "valueAngle": null,
          "valueV": null,
          "valueVA": null,
          "valueVAR": null,
          "valueW": null,
          "ACDCTerminal": "d4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f8a",
          "OperationalLimit": "e5f6a7b8-c9d0-4e1f-2a3b-4c5d6e7f8a9b",
          "ReportedByRegion": "10Y-FR-ENTSOE-1",
          "Contingency": "e1f2a3b4-c5d6-4e7f-8a9b-0c1d2e3f4a5b"
        }
      },
      "RemedialAction": "f2a3b4c5-d6e7-4f8a-9b0c-1d2e3f4a5b6c",
      "StageForRemedialActionScheme": "a3b4c5d6-e7f8-4a9b-0c1d-2e3f4a5b6c7d"
    }
  ]
}

```

**Key Notes:**
- `atTime` values are in milliseconds since Unix epoch (timestamp-millis logical type)
- `ReportedByRegion` uses ENTSO-E EIC format (10Y-XX--)
- Optional fields can be omitted or set to `null`
- All arrays can be empty but cannot be omitted
- Additional examples available in `/examples` directory with comprehensive field documentation

### 8.2. SecurityAnalysisResult_EXTENDED JSON

The EXTENDED version includes additional classes for demonstration purposes (Location, OperationalLimit, OperationalLimitSet, OperationalLimitType with compound types). Example payloads for the EXTENDED schema demonstrate compound type nesting and additional cardinality patterns. See [SecurityAnalysisResult Example Payloads README](example-payloads/README.md) for complete documentation.

### 8.3. Profile Constraint Rules

Next is an example of some of the additional complex validation constraints defined in Clause 2.3 of the [ENTSO-E's Security Analysis Result Profile Specification - 2025-09-11](https://eepublicdownloads.entsoe.eu/clean-documents/CIM_documents/Grid_Model_CIM/NC/2.4/SecurityAnalysisResults_Profile_Specification_2-4-0.pdf). They require additional considerations for how/where they are to be applied when working with Apache Avro and streaming data:

**R:NC:ALL:Region:reference**
The reference to the Region is normally a reference to the capacity calculation region, which is identified by "Y" EIC code of the capacity calculation region. ENTSO-E EIC region codes follow the format "10Y-XX--" where XX represents the country code (e.g., 10Y-DE-- for Germany, 10Y-FR-- for France, 10Y-PL-- for Poland, 10Y-BG-- for Bulgaria). These codes are the first part of the full 16-character EIC code and identify the country or area.

**R:NC:ALL:SystemOperator:reference**
The reference to the System Operator is normally identified by "X" EIC code of TSO.

**C:NC:SAR:PowerFlowResult:value**
PowerFlowResult.value and PowerFlowResult.absoluteValue are required attributes if the association end PowerFlowResult.OperationalLimit is provided.

**C:NC:SAR:PowerFlowResult:ApparentPowerLimit**
PowerFlowResult.valueVA is required attribute if an ApparentPowerLimit is referenced by the association end PowerFlowResult.OperationalLimit.

**C:NC:SAR:PowerFlowResult:ActivePowerLimit**
PowerFlowResult.valueW is required attribute if an ActivePowerLimit is referenced by the association end PowerFlowResult.OperationalLimit.

**C:NC:SAR:PowerFlowResult:ReactivePowerLimit**
PowerFlowResult.valueVAR is required attribute if a ReactivePowerLimit is referenced by the association end PowerFlowResult.OperationalLimit.

**C:NC:SAR:PowerFlowResult:VoltageLimit**
PowerFlowResult.valueV is required attribute if a VoltageLimit is referenced by the association end PowerFlowResult.OperationalLimit.

**C:NC:SAR:PowerFlowResult:VoltageAngleLimit**
PowerFlowResult.valueAngle is required attribute if a VoltageAngleLimit is referenced by the association end PowerFlowResult.OperationalLimit.

**C:NC:SAR:PowerFlowResult:CurrentLimit**
PowerFlowResult.valueA is required attribute if a CurrentLimit is referenced by the association end PowerFlowResult.OperationalLimit.

**C:NC:SAR:RemedialActionApplied.StageForRemedialActionScheme:cardinality**
The association end RemedialActionApplied.StageForRemedialActionScheme is required if RemedialActionApplied.RemedialAction references SchemeRemedialAction and it is not provided for any other RemedialAction objects.

**C:NC:SAR:PowerFlowResult:voltageAndAngle**
PowerFlowResult.valueV and PowerFlowResult.valueAngle shall not be included in the profile if PowerFlowResult is associated with ACDCTerminal or PowerTransferCorridor.

**C:NC:SAR:PowerFlowResult:association**
PowerFlowResult shall be associated with either ACDCTerminal, TopologicalNode or PowerTransferCorridor.

**C:NC:SAR:PowerFlowResult:topologicalNode**
PowerFlowResult.valueW, PowerFlowResult.valueVA, PowerFlowResult.valueVAR and PowerFlowResult.valueA shall not appear in the profile if PowerFlowResult is associated with TopologicalNode.

### 8.4. Complex Constraint Validation

The Security Analysis Result profile constraints fall into three categories based on their validation approach:

**Schema-Level Validation (Enforced by Avro)**

These constraints are enforced directly by the Avro schema through type system and cardinality:

- Required vs optional fields (union with null)
- Field data types (string, float, long, boolean)
- Array vs single-value fields
- Valid enumeration values

Example: The `isViolation` field is defined as `"type": "boolean"` ensuring only true/false values are accepted.

**Application-Level Validation (Conditional Business Rules)**

These constraints cannot be expressed in Avro schemas and must be validated by consuming applications. For example:

1. **Conditional field requirements based on association type:**
   - C:NC:SAR:PowerFlowResult:ApparentPowerLimit
   - C:NC:SAR:PowerFlowResult:ActivePowerLimit
   - C:NC:SAR:PowerFlowResult:ReactivePowerLimit
   - C:NC:SAR:PowerFlowResult:VoltageLimit
   - C:NC:SAR:PowerFlowResult:VoltageAngleLimit
   - C:NC:SAR:PowerFlowResult:CurrentLimit

   Since the OperationalLimit field is a string reference (mRID), the schema cannot determine the limit type. Applications must:

   - Resolve the OperationalLimit mRID to determine its type (CurrentLimit, VoltageLimit, etc.)
   - Validate that the corresponding value field (valueA, valueV, etc.) is present
   - Implement validation logic in the service layer

   Example validation pseudocode:
   ```
   if (powerFlowResult.OperationalLimit != null) {
       limitType = resolveLimitType(powerFlowResult.OperationalLimit)
       if (limitType == "CurrentLimit" && powerFlowResult.valueA == null) {
           throw ValidationError("valueA required for CurrentLimit")
       }
       // Similar checks for other limit types
   }
   ```

2. **Conditional requirements based on remedial action type:**
   - C:NC:SAR:RemedialActionApplied.StageForRemedialActionScheme:cardinality

   Applications must determine if a RemedialAction reference is a SchemeRemedialAction and validate accordingly.

3. **Cross-field validation:**
   - C:NC:SAR:PowerFlowResult:value (requires both value and absoluteValue when OperationalLimit present)

4. **Exclusion rules:**
   - C:NC:SAR:PowerFlowResult:voltageAndAngle (voltage/angle not allowed with certain associations)
   - C:NC:SAR:PowerFlowResult:topologicalNode (power values not allowed with TopologicalNode)

**Format Validation (Convention-Based)**

These constraints define expected formats for reference fields. For example:

1. **R:NC:ALL:Region:reference**
   - Region references should use ENTSO-E EIC "Y" code format
   - Format: "10Y-XX--" where XX is the country code
   - Examples: 
     - "10Y-DE--" for Germany
     - "10Y-FR--" for France  
     - "10Y-PL--" for Poland
     - "10Y-BG--" for Bulgaria
     - "10Y-CZ--" for Czech Republic
   - These codes are the first part of the full 16-character EIC code
   - Example payloads use formats like "10Y-DE-ENTSOE-1" and "10Y-FR-ENTSOE-1"
   - Applications should validate EIC code format if strict compliance required

2. **R:NC:ALL:SystemOperator:reference**
   - System Operator references should use EIC "X" code format
   - Not applicable to current profile (SystemOperator field not defined)

**Implementation Recommendations**

1. **Use Avro schema validation** for all type and cardinality constraints it can enforce

2. **Implement a validation service layer** that:

   - Resolves mRID references to determine object types
   - Applies conditional business rules
   - Validates cross-field dependencies
   - Checks format conventions (EIC codes, UUIDs)

3. **Document validation rules** alongside schema definitions

4. **Provide validation utilities** in code generation (e.g., validate() methods on generated classes)

5. **Consider JSON Schema** for additional validation if needed, though it requires maintaining parallel schemas

### 8.5. Notes on Example Payloads

The example JSON payloads in this case study demonstrate valid messages but use simplified string references for readability. Production implementations should:

- Use proper UUID format for all mRID references
- Use EIC codes for Region and SystemOperator references
- Implement validation logic for all application-level constraints
- Consider adding metadata or type hints if needed for validation efficiency

## 9. SecurityAnalysisResult.avsc Usage

### 9.1. Code Generation

Generate Java classes from schemas:
```bash
java -jar avro-tools.jar compile schema SecurityAnalysisResult.avsc ./output/
```

Generate Python classes:
```bash
avrogen SecurityAnalysisResult.avsc ./output/
```

### 9.2. Writing Messages

**Java Example:**
```java
import org.apache.avro.Schema;
import org.apache.avro.file.DataFileWriter;
import org.apache.avro.generic.GenericData;
import org.apache.avro.generic.GenericDatumWriter;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.io.DatumWriter;

import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class WriteSecurityAnalysisResult {
    public static void main(String[] args) throws IOException {
        // Load schema array
        Schema.Parser parser = new Schema.Parser();
        Schema[] schemas = parser.parse(new File("SecurityAnalysisResult.avsc"))
            .getTypes().toArray(new Schema[0]);
        
        // Find the schemas we need
        Schema containerSchema = null;
        Schema baseCaseSchema = null;
        Schema remedialActionSchema = null;
        
        for (Schema s : schemas) {
            if (s.getName().equals("SecurityAnalysisResult")) {
                containerSchema = s;
            } else if (s.getName().equals("BaseCasePowerFlowResult")) {
                baseCaseSchema = s;
            } else if (s.getName().equals("RemedialActionApplied")) {
                remedialActionSchema = s;
            }
        }
        
        // Create BaseCasePowerFlowResult record
        GenericRecord powerFlowResult = new GenericData.Record(baseCaseSchema);
        powerFlowResult.put("ACDCTerminal", "550e8400-e29b-41d4-a716-446655440000");
        powerFlowResult.put("isViolation", false);
        powerFlowResult.put("atTime", 1704067200000L);  // 2024-01-01T00:00:00Z
        powerFlowResult.put("OperationalLimit", "123e4567-e89b-12d3-a456-426614174000");
        powerFlowResult.put("ReportedByRegion", null);
        powerFlowResult.put("valueW", 125500.0f);
        powerFlowResult.put("valueVAR", 32100.0f);
        powerFlowResult.put("valueVA", null);
        powerFlowResult.put("valueV", 138000.0f);
        powerFlowResult.put("valueAngle", -5.3f);
        powerFlowResult.put("valueA", 940.2f);
        powerFlowResult.put("value", 85.5f);
        powerFlowResult.put("absoluteValue", 125500.0f);
        
        // Create RemedialActionApplied record
        GenericRecord remedialAction = new GenericData.Record(remedialActionSchema);
        remedialAction.put("mRID", "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d");
        remedialAction.put("PowerFlowResult", powerFlowResult);  // Union type
        remedialAction.put("RemedialAction", "c3d4e5f6-a7b8-4c9d-0e1f-2a3b4c5d6e7f");
        remedialAction.put("StageForRemedialActionScheme", null);
        
        // Create RemedialActionApplied array
        List<GenericRecord> remedialActions = new ArrayList<>();
        remedialActions.add(remedialAction);
        
        // Create container record
        GenericRecord container = new GenericData.Record(containerSchema);
        container.put("RemedialActionApplied", remedialActions);
        
        // Create writer
        File outputFile = new File("results.avro");
        DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(containerSchema);
        DataFileWriter<GenericRecord> dataFileWriter = 
            new DataFileWriter<>(datumWriter);
        dataFileWriter.create(containerSchema, outputFile);
        
        // Write container record
        dataFileWriter.append(container);
        dataFileWriter.close();
        
        System.out.println("Successfully wrote SecurityAnalysisResult");
    }
}
```

**Python Example:**
```python
from fastavro import writer, parse_schema
import json

# Load and parse schema
with open("SecurityAnalysisResult.avsc") as f:
    schema_json = json.load(f)

# fastavro can handle the array directly
parsed_schema = parse_schema(schema_json)

# Write message
records = [
    {
        "RemedialActionApplied": [
            {
                "mRID": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
                "PowerFlowResult": {
                    "ACDCTerminal": "550e8400-e29b-41d4-a716-446655440000",
                    "isViolation": False,
                    "atTime": 1704067200000,
                    "valueW": 125500.0,
                    "valueVAR": 32100.0,
                    "valueVA": None,
                    "valueV": 138000.0,
                    "valueAngle": -5.3,
                    "valueA": 940.2,
                    "value": 85.5,
                    "absoluteValue": 125500.0,
                    "OperationalLimit": "123e4567-e89b-12d3-a456-426614174000",
                    "ReportedByRegion": None
                },
                "RemedialAction": "c3d4e5f6-a7b8-4c9d-0e1f-2a3b4c5d6e7f",
                "StageForRemedialActionScheme": None
            }
        ]
    }
]

with open("results.avro", "wb") as out:
    writer(out, parsed_schema, records)

print("Successfully wrote SecurityAnalysisResult")
```

Note: **fastavro** automatically handles union types more intuitively - you don't need to wrap with the fully qualified type name in the dict structure.

### 9.3. Reading Messages

**Java Example:**
```java
import org.apache.avro.file.DataFileReader;
import org.apache.avro.generic.GenericDatumReader;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.io.DatumReader;
import java.io.File;
import java.io.IOException;
import java.util.List;

public class ReadSecurityAnalysisResult {
    public static void main(String[] args) throws IOException {
        // Create reader
        File inputFile = new File("results.avro");
        DatumReader<GenericRecord> datumReader = new GenericDatumReader<>();
        DataFileReader<GenericRecord> dataFileReader = 
            new DataFileReader<>(inputFile, datumReader);
        
        // Read container records
        while (dataFileReader.hasNext()) {
            GenericRecord container = dataFileReader.next();
            
            // Get RemedialActionApplied array
            List<GenericRecord> remedialActions = 
                (List<GenericRecord>) container.get("RemedialActionApplied");
            
            for (GenericRecord remedialAction : remedialActions) {
                System.out.println("Remedial Action mRID: " + remedialAction.get("mRID"));
                System.out.println("Remedial Action: " + remedialAction.get("RemedialAction"));
                
                // Get PowerFlowResult (union type)
                GenericRecord powerFlowResult = 
                    (GenericRecord) remedialAction.get("PowerFlowResult");
                
                // Access fields from the power flow result
                System.out.println("Terminal: " + powerFlowResult.get("ACDCTerminal"));
                System.out.println("Violation: " + powerFlowResult.get("isViolation"));
                System.out.println("Time: " + powerFlowResult.get("atTime"));
                
                // Check for optional power value
                Object valueW = powerFlowResult.get("valueW");
                if (valueW != null) {
                    System.out.println("Power: " + valueW + " W");
                }
                
                // Check for optional operational limit
                Object operationalLimit = powerFlowResult.get("OperationalLimit");
                if (operationalLimit != null) {
                    System.out.println("Limit: " + operationalLimit);
                }
                
                // Check if this is a contingency result
                Object contingency = powerFlowResult.get("Contingency");
                if (contingency != null) {
                    System.out.println("Contingency: " + contingency);
                }
                
                System.out.println("---");
            }
        }
        
        dataFileReader.close();
    }
}
```

**Python Example:**
```python
from avro.datafile import DataFileReader
from avro.io import DatumReader
from datetime import datetime

# Read results
reader = DataFileReader(open("results.avro", "rb"), DatumReader())

for container in reader:
    # Get RemedialActionApplied array
    remedial_actions = container['RemedialActionApplied']
    
    for remedial_action in remedial_actions:
        print(f"Remedial Action mRID: {remedial_action['mRID']}")
        print(f"Remedial Action: {remedial_action['RemedialAction']}")
        
        # Get PowerFlowResult (union type - will be dict with type name as key)
        power_flow_result = remedial_action['PowerFlowResult']
        
        # For union types, Avro wraps in dict with type name as key
        # Get the actual result data (first value in the union dict)
        if isinstance(power_flow_result, dict):
            # The key is the fully qualified type name
            result_data = list(power_flow_result.values())[0]
        else:
            result_data = power_flow_result
        
        print(f"Terminal: {result_data['ACDCTerminal']}")
        print(f"Violation: {result_data['isViolation']}")
        
        # Convert timestamp to readable format
        timestamp_ms = result_data['atTime']
        timestamp = datetime.fromtimestamp(timestamp_ms / 1000.0)
        print(f"Time: {timestamp}")
        
        # Check for optional power value
        if result_data['valueW'] is not None:
            print(f"Power: {result_data['valueW']} W")
        
        # Check for optional operational limit
        if result_data['OperationalLimit'] is not None:
            print(f"Limit: {result_data['OperationalLimit']}")
        
        # Check if this is a contingency result
        if 'Contingency' in result_data and result_data['Contingency'] is not None:
            print(f"Contingency: {result_data['Contingency']}")
        
        print("---")

reader.close()
```

## 10. Schema Evolution

### 10.1. Backward Compatible Changes:
- Adding optional fields (with defaults)
- Adding enum values (if consumers handle unknown gracefully)

### 10.2. Breaking Changes:
- Removing fields
- Changing field types
- Removing enum values
- Changing required/optional status

### 10.3. Best Practices:
1. Always provide defaults for our optional fields
2. Use unions with null for our optional fields
3. Document units and semantics clearly
4. Version our message schemas
5. Test with old and new consumers

## 11. Validation

To validate a message against a schema:

```bash
# Using avro-tools
java -jar avro-tools.jar validate \
  SecurityAnalysisResult.avsc \
  result_message.avro
```

## 12. Tools

Recommended tools:
- **avro-tools** - Command-line tools for Avro
- **avrogen** - Python code generator
- **Kafka** - Streaming platform (commonly used with Avro)
- **Schema Registry** - Schema versioning and compatibility

## 13. References

- **CIMTool**: See [https://cimtool.ucaiug.io/](https://cimtool.ucaiug.io/) 
- **CIMTool Application**: Release 2.3.0-RC3 available at [CIMTool-2.3.0-RC3](https://github.com/cimug-org/CIMTool/releases/download/2.2.0/CIMTool-2.3.0-RC3-win32.win32.x86_64.zip)
- **SAR Profile**: See [ENTSO-E's Security Analysis Result Profile Specification 2.4.0](https://eepublicdownloads.entsoe.eu/clean-documents/CIM_documents/Grid_Model_CIM/NC/2.4/SecurityAnalysisResults_Profile_Specification_2-4-0.pdf)
- **Avro Format**: See https://avro.apache.org/

## 14. License

Generated schemas distributed under the **Apache 2.0 license**.  
See [LICENSE](https://github.com/cimug-org/Avro4CIM/blob/main/LICENSE) for details.
