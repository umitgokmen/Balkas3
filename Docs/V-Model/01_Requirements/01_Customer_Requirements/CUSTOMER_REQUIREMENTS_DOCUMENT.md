# Balcas Customer Requirements Document

| Document field | Value |
|---|---|
| Document code | BLC-MGD-001 |
| Document type | Customer Requirements Document (CRD / URS) |
| Version | 0.4 |
| Status | Draft for customer review |
| Date | 28 September 2026 |
| Prepared by |  |
| Customer | Balcas |


## 1. Purpose

## 2. Business need and objectives

Balcas requires a reliable quality-control solution that automatically inspects the three-dimensional data of every log on the production line and communicates the result to the PLC.

The system's business objectives are:

- Perform exactly one log inspection for each valid line trigger.
- Produce the log's length, diameter, orientation, butt flare, and deflection information.
- Distinguish a valid product rejection from a measurement failure.
- When a reliable measurement cannot be produced, record the condition as "Measurement Failure" and, in accordance with the production-loss business rule, send `Outcome=true` to the PLC so that the log continues along the acceptance path.
- Present results clearly to the operator and transfer them safely to the PLC.
- Record inspections traceably so that production issues can be investigated.
- Support testing and repeat analysis independently of the production line by using recorded data.

## 3. Scope

### 3.1 In scope

- Acquiring data from three independent 3D camera channels
- Combining camera data in a common coordinate system
- Detecting the log and checking the quality of the measurement data
- Measuring length, diameter, orientation, flare, and deflection
- Producing acceptance, rejection, and measurement-failure decisions
- Exchanging trigger, status, result, and acknowledgement information with the PLC
- Operator screens, settings, and diagnostic views
- Recording inspection results, errors, and selected raw data
- Single-dataset and batch simulation using recorded data

## 4. Stakeholders and users

| Role | Primary expectation / responsibility |
|---|---|
| Operator | Monitor system status, understand errors, and use permitted commands |
| Process and quality owner | Approve measurement, tolerance, and acceptance/rejection rules |
| Automation/PLC team | Approve the PLC data contract and handshake sequence |
| Camera/calibration specialist | Verify camera placement, calibration, and data quality |
| Maintenance/IT | Operate the computer, network, storage, and backup environment |
| Software supplier | Implement, verify, and deliver the approved requirements |

## 5. Priority and compliance language

| Term | Definition |
|---|---|
| Mandatory | Must be satisfied for system acceptance. |
| Recommended | Improves operational quality; a written justification is required if it is not implemented. |
| Optional | May be considered under separate planning or in a later phase. |
| Decision pending | Must be clarified by the customer or the relevant process owner. |

Compliance with a requirement shall be demonstrated by the applicable method of test, inspection, demonstration, or measurement.

## 6. General operating scenario

1. The system starts, loads the approved settings, and connects to the PLC and enabled cameras.
2. If the required connections and settings are valid, the system enters the "Ready" state.
3. The PLC sends an inspection request for a new log.
4. The system acquires 3D data from the enabled cameras and creates a single inspection record.
5. Data sufficiency is checked; if the data is valid, measurements and the product decision are calculated.
6. The result is sent to the PLC and displayed on the operator screen.
7. The system holds the result unchanged until the PLC acknowledges receipt.
8. After acknowledgement, the result fields are cleared and the system becomes ready for the next log.

## 7. Customer requirements

### 7.1 Start-up and readiness

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| MGR-OPS-001 | Mandatory | On start-up, the system shall load the latest valid and approved settings. | Test |
| MGR-OPS-002 | Mandatory | If settings are missing, corrupt, or invalid, the system shall inform the operator and shall not behave as though it is producing reliable measurements. | Test |
| MGR-OPS-003 | Mandatory | In production mode, the system shall not indicate "Ready" until all enabled cameras and PLC communication are ready. All three cameras shall be enabled in the default production configuration. | Test |
| MGR-OPS-004 | Mandatory | The system shall clearly indicate whether its operating mode is "Production" or "Simulation". | Demonstration |
| MGR-OPS-005 | Mandatory | After a controlled shutdown or an unexpected restart, the system shall return to a safe, defined initial state with the PLC. | Test |

### 7.2 Inspection and measurement

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| MGR-INS-001 | Mandatory | Each valid PLC trigger shall create exactly one inspection. A trigger that remains active for an extended period shall not cause a second inspection. | Test |
| MGR-INS-002 | Mandatory | Each inspection shall have a unique identifier, and the same identifier shall be used on screen, in logs, in results, and in stored data. | Inspection/Test |
| MGR-INS-003 | Mandatory | The system shall support three independent 3D camera channels and collect data from enabled cameras within the same inspection. All three cameras shall be enabled by default, and each camera shall be capable of being disabled individually for testing. | Test |
| MGR-INS-004 | Mandatory | The system shall measure the log length in millimetres. | Reference-object test |
| MGR-INS-005 | Mandatory | The system shall measure the minimum, median, and maximum stem diameters in millimetres. | Reference-object test |
| MGR-INS-006 | Mandatory | The system shall report the log orientation as "butt end leading", "butt end trailing", or "unknown". | Labelled-data test |
| MGR-INS-007 | Mandatory | The system shall report the presence or absence of flare as a separate result. | Labelled-data test |
| MGR-INS-008 | Mandatory | The system shall evaluate log deflection around 360 degrees. The calculation mode shall be selectable in the settings as "One-way", "Two-way", or "Both". The deflection value or values for the selected mode shall be reported together with the relevant angle and position. | Reference-data test/Demonstration |
| MGR-INS-009 | Mandatory | The flare region and measurement segments determined to be unreliable shall not distort the normal stem-diameter or deflection calculation. | Reference-data test |
| MGR-INS-010 | Mandatory | For a measurement to be considered valid, each defined sector shall contain at least one valid data point. If any sector contains no valid data, the system shall not fabricate a numeric measurement and shall return "Measurement Failure". | Failure-scenario test |
| MGR-INS-011 | Mandatory | When the same reference object/data is measured repeatedly under the same conditions and with the same settings, the difference between the maximum and minimum values obtained for each of length, diameter, and deflection shall not exceed 10 mm. | Repeatability test |
| MGR-INS-012 | Mandatory | The deflection measurement and product decision shall be completed using the approved calculation mode selected at the start of the inspection; the system shall not automatically switch to another mode during measurement. | Design review/Test |

### 7.3 Decision rules

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| MGR-DEC-001 | Mandatory | The system shall distinguish three outcomes: "Accepted", "Rejected", and "Measurement Failure". | Test |
| MGR-DEC-002 | Mandatory | The permitted deflection shall be calculated from the log length and the customer-approved deflection limit per metre. | Calculation check |
| MGR-DEC-003 | Mandatory | For a valid measurement, if the deflection is equal to or less than the permitted value, the outcome shall be "Accepted". | Boundary-value test |
| MGR-DEC-004 | Mandatory | For a valid measurement, if the deflection is greater than the permitted value, the outcome shall be "Rejected". | Boundary-value test |
| MGR-DEC-005 | Mandatory | When a reliable decision cannot be produced, the internal outcome shall be "Measurement Failure". To reduce production loss, the system shall send `Outcome=true` to the PLC so that the log proceeds along the acceptance path. A measurement failure shall not be reported as a valid accepted measurement, and error details shall be recorded separately. | Failure-scenario test |
| MGR-DEC-006 | Mandatory | The presence of flare alone shall not cause rejection. | Rule review/Test |
| MGR-DEC-007 | Mandatory | The final deflection shall be the value produced by the measurement method; no deflection correction factor shall be applied to the result. | Calculation check/Test |
| MGR-DEC-008 | Mandatory | An undetermined log orientation shall not affect the acceptance/rejection outcome; the orientation shall be recorded as "Unknown". | Rule review/Test |
| MGR-DEC-009 | Mandatory | When the deflection calculation mode is "Both", the one-way and two-way results shall be reported separately. The larger of the two values shall be used as the final deflection for the acceptance/rejection decision. | Calculation check/Test |

### 7.4 PLC integration

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| MGR-PLC-001 | Mandatory | The system shall manage idle, running, result-hold, and disabled states in accordance with the agreed PLC contract. | Integration test |
| MGR-PLC-002 | Mandatory | A valid final result shall not be presented to the PLC before measurement is complete. | Integration test |
| MGR-PLC-003 | Mandatory | The system shall hold results unchanged until PLC acknowledgement is received. | Integration test |
| MGR-PLC-004 | Mandatory | After PLC acknowledgement is received, the results shall be cleared and the system shall prepare for a new inspection. | Integration test |
| MGR-PLC-005 | Mandatory | If the PLC connection or heartbeat is lost, the "Ready" state shall be removed, an error shall be displayed, and controlled reconnection shall be attempted. | Failure-scenario test |
| MGR-PLC-006 | Mandatory | Existing PLC tags, data types, and units shall not be changed without an interface change approved by both parties. Modbus is not part of this integration. | Interface review |
| MGR-PLC-007 | Mandatory | In a "Measurement Failure" condition, the PLC shall receive `Outcome=true`. The system shall nevertheless preserve the actual outcome as "Measurement Failure" on the operator screen and in the records. | Integration test |

### 7.5 Operator interface

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| MGR-UI-001 | Mandatory | The main screen shall show the status of each camera channel and the PLC connection. | Demonstration |
| MGR-UI-002 | Mandatory | The main screen shall show the latest inspection's identifier, outcome, length, diameters, deflection, permitted deflection, orientation, and flare information. | Demonstration |
| MGR-UI-003 | Mandatory | Acceptance, rejection, and measurement failure shall be distinguished by explicit text as well as by colour. | Demonstration |
| MGR-UI-004 | Mandatory | Counters for total triggers, completed inspections, acceptances, rejections, and measurement failures shall be displayed separately. | Test |
| MGR-UI-005 | Mandatory | Production and simulation results and counters shall not be mixed. | Test |
| MGR-UI-006 | Mandatory | Invalid settings shall not be saved; the affected field and reason for the error shall be shown to the operator. | Test |
| MGR-UI-007 | Mandatory | An inspection in progress shall be completed using the settings present at its start; subsequent changes shall apply to the next inspection. | Test |
| MGR-UI-008 | Recommended | Authorised users should be able to view raw, processed, and measured 3D data for diagnostic purposes. | Demonstration |
| MGR-UI-009 | Mandatory | The operator interface shall provide a manual inspection command. The command shall be enabled only when the system is ready to start a new inspection. A manually initiated inspection shall be clearly marked in the records, and a PLC trigger for the same log shall not create a second inspection. | Demonstration/Test |

### 7.6 Simulation, recording, and reporting

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| MGR-DATA-001 | Mandatory | The system shall be able to run the same measurement and decision process on recorded 3D data without requiring a physical camera. | Test |
| MGR-DATA-002 | Mandatory | A single dataset and all datasets in a folder shall be executable through separate commands. | Demonstration |
| MGR-DATA-003 | Mandatory | In batch simulation, a corrupt dataset shall not prevent other datasets from being processed. | Test |
| MGR-DATA-004 | Mandatory | The batch-simulation summary shall include the outcome, measurements, duration, and error reason for each dataset. | Output review |
| MGR-DATA-005 | Recommended | The batch-simulation summary should be exportable in CSV or JSON format. | Demonstration |
| MGR-DATA-006 | Mandatory | For each inspection, the start/end time, identifier, operating mode, outcome, measurements, error, duration, and software/settings version shall be recorded. | Record review |
| MGR-DATA-007 | Mandatory | Storage of raw data for manual inspections, rejections, measurement failures, or diagnostic purposes shall be independently configurable. | Test |
| MGR-DATA-008 | Mandatory | If the recording destination is temporarily unavailable, the measurement result shall not be lost, and the operator shall be notified of the storage error. | Failure-scenario test |
| MGR-DATA-009 | Mandatory | The application is not required to delete logs or raw data automatically based on age. Retention, archiving, backup, disk quotas, and deletion shall be manageable by IT/operations outside the application. | Design review/Demonstration |

## 8. Non-functional requirements

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| MGR-NFR-001 | Mandatory | The elapsed time from the PLC trigger until the result is ready shall be measured and recorded. | Record review |
| MGR-NFR-003 | Mandatory | The user interface shall remain responsive during camera acquisition, 3D processing, and file writing. | Performance test |
| MGR-NFR-004 | Mandatory | An unexpected error shall not cause subsequent inspections to stop silently; the system state and error reason shall be visible. | Resilience test |
| MGR-NFR-005 | Mandatory | If a measurement cannot be produced because of camera or data quality, the error shall be visible and traceable. In accordance with the customer business rule, the internal outcome shall remain "Measurement Failure" while `Outcome=true` is sent for line flow. | Failure-scenario test |
| MGR-NFR-006 | Mandatory | An endurance test shall demonstrate that memory and device-resource usage do not grow without control during prolonged operation. | Endurance test |
| MGR-NFR-007 | Mandatory | Millimetres shall be the base unit for all physical length and diameter results. | Inspection/Test |
| MGR-NFR-008 | Mandatory | Every software delivery shall have unique application and measurement-algorithm versions. | Version review |
| MGR-NFR-009 | Recommended | Critical calibration and decision settings should be protected against unauthorised changes, and changes should be auditable. | Security review |
| MGR-NFR-010 | Mandatory | PLC, camera, recording-path, and other settings shall be configurable without modifying the source code. | Demonstration |

## 9. Minimum customer acceptance scenarios

| ID | Scenario | Expected result |
|---|---|---|
| MKA-001 | A valid log within limits is inspected while the system is ready. | One inspection is created; measurements are published; the outcome is "Accepted" and is held until PLC acknowledgement. |
| MKA-002 | The deflection exceeds the permitted limit in a valid measurement. | The outcome is "Rejected"; the measured and permitted values are recorded. |
| MKA-003 | One of the enabled cameras times out or returns empty data. | The internal outcome is "Measurement Failure", the camera error is recorded, and `Outcome=true` is sent to the PLC. |
| MKA-004 | At least one defined sector contains no valid data. | No special or magic number is used in place of a physical measurement; the internal outcome is "Measurement Failure", and `Outcome=true` is sent to the PLC. The coverage condition is satisfied when every sector contains at least one valid data point. |
| MKA-005 | The PLC acknowledges the result. | The result fields are cleared and the system returns to the "Ready" state for a new inspection. |
| MKA-006 | The PLC trigger signal remains high. | A second inspection does not start for the same log. |
| MKA-007 | The deflection is exactly equal to the approved limit. | The outcome is "Accepted". |
| MKA-008 | An attempt is made to save an invalid setting. | The save is rejected and an explanatory field error is displayed. |
| MKA-009 | A batch simulation is run with one corrupt dataset among ten datasets. | The corrupt dataset is reported as an error; the other nine datasets are processed, and the summary contains ten records. |
| MKA-010 | A decision setting is changed during an active inspection. | The current inspection is completed using the old setting, and the next inspection uses the new setting. |
| MKA-011 | The same reference object/data is processed repeatedly under the same conditions and with the same settings. | Screen, record, and PLC values use the same units; for each of length, diameter, and deflection, the difference between the maximum and minimum obtained values does not exceed 10 mm. |
| MKA-012 | The application restarts after an unexpected shutdown. | An old result is not published as though it were new, and a safe initial state is established with the PLC. |
| MKA-013 | One or two cameras are disabled in the test configuration. | The system waits only for enabled cameras, clearly identifies disabled cameras, and runs the test inspection using data from the enabled cameras. |
| MKA-014 | Flare is detected in a valid measurement and the deflection is within limits. | The flare is recorded; it does not cause rejection by itself, and the outcome is "Accepted". |
| MKA-015 | Orientation cannot be determined and all other measurements are valid. | The orientation is recorded as "Unknown"; orientation uncertainty does not change the acceptance/rejection outcome. |
| MKA-016 | Reference data with a known deflection is processed with the deflection calculation mode set in turn to "One-way", "Two-way", and "Both". | Values for the selected mode are reported separately with correct labels; in "Both" mode, the larger value is used as the final deflection, and no correction factor is applied to any result. |
| MKA-017 | The inspection-duration warning is enabled and the configured threshold is exceeded. | The overrun is recorded and a warning is shown to the operator; the product outcome is determined solely by the measurement and decision rules. No timing warning is generated when timing supervision is disabled. |
| MKA-018 | Stored logs and raw data must be archived or deleted under an external IT/operations policy. | The application does not perform automatic age-based deletion; the data can be managed outside the application. |
| MKA-019 | The operator issues the manual-inspection command while the system is ready for a new inspection. | Exactly one inspection starts, the source type is recorded as "Manual", and a concurrent PLC trigger for the same log does not start a second inspection. |

The final factory acceptance test (FAT) and site acceptance test (SAT) procedures shall be derived from these scenarios and the approved quantitative targets.

## 10. Deliverables

The minimum delivery scope includes:

- Executable Balcas inspection application and version information
- Approved default configuration and setting descriptions
- PLC interface/data contract
- Operator instructions
- Installation, backup, and restore instructions
- Error-code and troubleshooting list
- FAT/SAT test procedures and test results
- Approved reference datasets and regression-test summary
- Release notes and known limitations

## 11. Customer inputs and responsibilities

The customer or a party authorised by the customer shall provide and approve the following inputs:

- Warning threshold where the optional inspection-duration warning is used
- Measurement ranges and references against which the 10 mm repeatability parameter will be verified
- Maximum permitted deflection per metre
- Placement, field of view, and calibration references for the three cameras
- PLC protocol, tag list, and data types
- Acceptance/rejection business rules and line behaviour in the event of failure
- Reference logs/datasets and expected results
- Disk quota, archiving, backup, and deletion policy for raw data and logs, to be operated outside the application
- Production-network access, user permissions, and cybersecurity rules
- FAT and SAT environments, test time, and acceptance authorities

The affected requirements cannot receive final verification until these inputs have been provided.


## 13. Traceability and change management

Each customer requirement is tracked by a unique `MGR-*` identifier. System requirements, design items, and acceptance tests shall reference the relevant customer-requirement identifier.

After approval, changes to scope, business rules, interfaces, or acceptance criteria shall be managed through a written change request. At a minimum, the change request shall identify the affected requirements, cost/schedule impact, verification needs, and approving parties.

## 14. Assumptions and dependencies

- The log is presented in a mechanically suitable position within the camera field of view and remains sufficiently stable during measurement.
- The camera, PLC, network, and computer hardware are suitable for production conditions and are operational.
- Suitable physical references are provided for calibration and its periodic verification.
- The PLC interface and line sequence can be tested jointly with the automation team.
- Customer-verified samples are provided for acceptance of the measurement algorithm.
- Site conditions such as ambient light, vibration, contamination, temperature, and network load are maintained within the agreed operating range.

## Appendix A — Revision history
