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

- `01_Requirements/01_Customer_Requirements`: Customer needs, expectations, constraints, and acceptance objectives.
- `01_Requirements/02_System_Requirements`: Derived functional and non-functional system requirements.
- `02_System_Architecture`: System architecture, interfaces, component allocation, and architectural decisions.
- `03_Detailed_Design`: Detailed component, software, data, and interface designs.
- `04_Implementation`: Implementation records and supporting implementation documentation.
- `05_Unit_Verification`: Unit verification plans, specifications, procedures, and results.
- `06_Integration_Verification`: Integration verification plans, specifications, procedures, and results.
- `07_System_Verification`: Evidence that the implemented system satisfies the system requirements.
- `08_Customer_Acceptance_Validation`: Evidence that the delivered system satisfies customer needs and intended use.
- `09_Traceability`: Traceability matrices and change-impact records linking requirements, design, implementation, and tests.

## Document placement rules

1. Store each document in the folder representing its primary lifecycle purpose.
2. Reference related documents and requirement identifiers instead of duplicating content.
3. Maintain bidirectional traceability from customer requirements through system requirements and design to verification evidence.
4. Record document revisions and approvals within each controlled document.
5. Replace a folder's `.gitkeep` file when the first substantive document is added.

## Current documents

- `01_Requirements/01_Customer_Requirements/CUSTOMER_REQUIREMENTS_DOCUMENT.md`
- `01_Requirements/02_System_Requirements/SYSTEM_REQUIREMENTS.md`
