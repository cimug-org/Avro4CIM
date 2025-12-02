# Avro4CIM

This UCAIug repository hosts the exploration and development of approaches for producing **Apache Avro schemas** for the **Common Information Model (CIM)**.

The focus is on generating Avro schemas for data suitable for exchange on **Kafka**, scheduled transfers, or real-time streams where minimum latency between source and destination is critical. The design goal is to enable **lightweight online validation** while still providing sufficient structure for **extensive offline validation**. Time-series data are expected to remain within defined limits, consistent with the resources described in the structured data exchange.

The **Avro4CIM** project leverages **CIMTool (version 2.3.0-RC3)** to generate Avro schema definitions in alignment with the evolving CIM specification. This approach is intended to complement existing **CIM XML/XSD-based exchanges** by providing Avro as an additional serialization and integration option.

## CSA 2.4 Security Analysis Result 2.4 Case Study
For the full case study and beta release of the CIMTool Apache Avro builder visit [Case Study Overview](./case-study/case_study_readme.md).

The **CIMTool** project for 

## Contributing
We welcome contributions. Please use the standard **fork → branch → pull request** workflow when proposing changes.

- Make branch names informative (e.g., include an issue or feature number).  
- Editorial fixes (clarity, typos, documentation improvements) are always welcome.  
- Please review [CONTRIBUTING.md](./CONTRIBUTING.md) for guidance on licensing contributions.

## Code of Conduct
This repository operates under a [Code of Conduct](./CODE_OF_CONDUCT.md).

## License
Distributed under the **Apache 2.0 license**.  
See [LICENSE](./LICENSE) for details.

## Tagline
*Experimental use of CIMTool to generate Apache Avro schemas for CIM streaming and real-time exchanges*
