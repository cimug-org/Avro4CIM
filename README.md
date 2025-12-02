# Avro4CIM

This UCAIug repository hosts the exploration and development of approaches for producing **Apache Avro schemas** for the **Common Information Model (CIM)**.

The focus is on generating Avro schemas for data suitable for exchange on **Kafka**, scheduled transfers, or real-time streams where minimum latency between source and destination is critical. The design goal is to enable **lightweight online validation** while still providing sufficient structure for **extensive offline validation**. Time-series data are expected to remain within defined limits, consistent with the resources described in the structured data exchange.

The **Avro4CIM** project leverages **CIMTool (version 2.3.0-RC3)** to generate Avro schema definitions in alignment with the evolving CIM specification. This approach is intended to complement existing **CIM XML/XSD-based exchanges** by providing Avro as an additional serialization and integration option.

## CSA 2.4 Security Analysis Result (SAR) Profile Case Study

![](https://img.plantuml.biz/plantuml/svg/hLVRJkD847ttLxJ8mqYB4W84jA08ZUF4cf56B4eG-x3QXyQk92tQRgkx3PCT-FUjQZl1bpWxZDOMYbrrwjIrA-6piLpRF96ULHcpYZqgQrN2Og4XiaAbdlU9gOoUs4hpM41gsIEFdbPQvMbC5aOakUGMsCndfvU3xsZB4PeOppo9DEFuo2OxYq19fLaldD3TxqqlfsV1Z9ny4J6mI79Zc8XKQhor4mWTIIYU8SdiXYLWroesP7A_5eQHoZ2x1iQjrU8nnqWAWdVtZjRVgh-YTY7-0JQiG7ojrKiPtEXYT3cwg2ZDciVgHQGhPdKEv9fqAGcMQ90g2rLtzCv-U8nzQkMz7DADghOcMluEvwHHEWPT3uMnKAMsb9sxgvJTyWGmUC5bAN2PdPxjYB18qLCAMDPJb2tgcRnvpIgvkDljceL1UFNQ2wzcqndnx8fPYUl27mr7GJtMeRkseTcD19mm6LCa5ZGekybW7_mBS8pziAP6jxzg-qf0VIdhCFjk8JKqsovtd2FZE6DOYbIuyMyLOpnksDxSiDuifFkr9yLgRN7uDri__gEncaVsGmvuQKJCeRjAvWr8gLwstBCqJq1pOlavNdWtS45VXU7iuhKt7OP-SEaLd7WCjFbGj40cPQZd8VA5OJ0C9jKv68zdMz2kZ-q9QgFUwAYviryhYjcDZbSgjp5KDDw-0_cJKpAug1scKVc6G3ThxR5ympZzE6QyS6NL2zTniPNoB0FD92nn1fHEyLajdQ9Savwpu8Oj8Cgri1i_vN9ZX9c3oQMz9MsZ--nEgwMGSDrXGn3Q2oRZ4OBMrcRc-kI4P_rd5GqYaOm6a9_ijpqv6unw7k1gfxYe_dum3Uv19m9n7g_DSqt7watwGTr3ymat4E3VdNh1Oui957wOIDU0xWrRQlo-QMYzWhxBTZo-3T15uQEbVEttkomfe02xXmiGqcZJJAUYwiYcWlYLX0wO_tTEtSRU0TwZumFAuM3msn7hzs_gC7PwLAl5THaTLU2Fh5mXN7uJYR3D6ZRr7OLiPj1WwQXEWmhd_BZKe5n9t2OFNEQ00QCjj-mlJF3qxnhEtek405FyoKAFLQKm3q995pqY9af9u6aLy5oot-4IeLz2RNw7y14Iv5eZX7BeXAKhIMpkSmeh3RXoEx0FY7jGql8LT4B8hP_XoaE_he3To3b2vu1r5Czm4FedSRPXUsRqwFZGTDKs1OvNRQjyZAerOHJIKNlmTbFRfdU0zrvNsUUGAMqXdcpci2gLo5w-L4z28MfERWdSFx35xCyX0Tp-qYz-ORS8gjeF467hfO3OsqTJwYfRfjBePFvrYfajy0ivNH7rVbkgpJvvjNhDKcFbcg9T2N_TIwD4hZqroV_WOq57BLIwZbn4QtmlPy91QYFX9Cs65h3EFdx6Brlw1ypx5m00)

For the full case study and beta release of the CIMTool Apache Avro builder visit [Case Study Overview](./case-study/README.md).

## The CIMTool SAR Project

- **Clone This Repository**: to access the CIMTool project that defines the CSA 2.4 Security Analysis Result Avro Profile.
- **For Information on CIMTool**: see [https://cimtool.ucaiug.io/](https://cimtool.ucaiug.io/) 
- **To Download**: see [CIMTool-2.3.0-RC3](https://github.com/cimug-org/CIMTool/releases/download/2.2.0/CIMTool-2.3.0-RC3-win32.win32.x86_64.zip)
- **Import the Avro builder**: use the "Import XSLT Transform Builder" wizard with the following settings:
  
  <img width="50%" height="50%" alt="image" src="https://github.com/user-attachments/assets/8bf76618-e444-4f46-ab29-78c09f574efc" />

## Avro Schema Builder Latest Release

- 1.0.0-beta
- 
  - The beta release is available here on GitHub at [CIMTool-2.2.0](./builder/apache-avro-schema.xsl).
    
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
