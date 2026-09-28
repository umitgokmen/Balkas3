# V-Model Documentation

This directory organizes the project documentation according to the V-Model. The numbered folders represent the progression from requirements and design to implementation, verification, and customer validation.

## Lifecycle mapping

| Definition and development phase | Corresponding verification or validation phase |
| --- | --- |
| Customer requirements | Customer acceptance and validation |
| System requirements | System verification |
| System architecture | Integration verification |
| Detailed design | Unit verification |
| Implementation | Implemented system and supporting evidence |

## Directory structure

- `01_Gereksinimler/01_Musteri_Gereksinimleri`: Customer needs, expectations, constraints, and acceptance objectives.
- `01_Gereksinimler/02_Sistem_Gereksinimleri`: Derived functional and non-functional system requirements.
- `02_Sistem_Mimarisi`: System architecture, interfaces, component allocation, and architectural decisions.
- `03_Detayli_Tasarim`: Detailed component, software, data, and interface designs.
- `04_Uygulama`: Implementation records and supporting implementation documentation.
- `05_Birim_Dogrulama`: Unit verification plans, specifications, procedures, and results.
- `06_Entegrasyon_Dogrulama`: Integration verification plans, specifications, procedures, and results.
- `07_Sistem_Dogrulama`: Evidence that the implemented system satisfies the system requirements.
- `08_Musteri_Kabul_Validasyonu`: Evidence that the delivered system satisfies customer needs and intended use.
- `09_Izlenebilirlik`: Traceability matrices and change-impact records linking requirements, design, implementation, and tests.

## Document placement rules

1. Store each document in the folder representing its primary lifecycle purpose.
2. Reference related documents and requirement identifiers instead of duplicating content.
3. Maintain bidirectional traceability from customer requirements through system requirements and design to verification evidence.
4. Record document revisions and approvals within each controlled document.
5. Replace a folder's `.gitkeep` file when the first substantive document is added.

## Current documents

- `01_Gereksinimler/01_Musteri_Gereksinimleri/CUSTOMER_REQUIREMENTS_DOCUMENT.md`
- `01_Gereksinimler/02_Sistem_Gereksinimleri/SISTEM_GEREKSINIMLERI.md`
