# Balcas Customer Requirements Document

| Document field | Value |
|---|---|
| Document code | BLC-CRD-001 |
| Document type | Customer Requirements Document (CRD / URS) |
| Version | 0.5 |

| Date | 1 October 2026 |

| Customer | Balcas |

## 2. Business need and objectives

Balcas requires a reliable quality-control solution that automatically inspects the three-dimensional data of every log on the production line and communicates the result to the PLC.

The system's business objectives are:

- Perform exactly one log inspection for each valid line trigger.
- Produce the log's length, diameter, orientation, butt flare, and deflection information.
- Distinguish a valid product rejection from a measurement failure.
- When a reliable measurement cannot be produced, record the condition as "Measurement Failure" and, in accordance with the production-loss business rule, send `Outcome=true` to the PLC so that the log continues along the acceptance path.
- Present results clearly to the operator and transfer them safely to the PLC.
- Record inspections traceably so that production issues can be investigated.
- Support repeat analysis independently of the production line by using recorded 3D data.

## 3. Scope

- Acquiring data from three independent 3D camera channels
- Combining camera data in a common coordinate system
- Detecting the log and checking the quality of the measurement data
- Measuring length, diameter, orientation, flare, and deflection
- Producing acceptance, rejection, and measurement-failure decisions
- Exchanging trigger, status, result, and acknowledgement information with the PLC
- Selecting either an external PLC or a built-in PLC for the request, result, and acknowledgement exchange
- Selecting either live cameras or recorded 3D data as the inspection input
- Operator screens, settings, and diagnostic views
- Recording inspection results, errors, and selected raw data
- Single-dataset and batch processing using recorded 3D data

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

1. The system starts, loads the last successfully saved valid settings, and prepares the selected PLC and 3D data sources.
2. If the selected sources and required settings are ready, the system enters the "Ready" state.
3. The selected PLC sends an inspection request for a new log.
4. The system obtains 3D data from the selected source and creates a single inspection record.
5. Data sufficiency is checked; if the data is valid, measurements and the product decision are calculated.
6. The result is sent to the selected PLC and displayed on the operator screen.
7. The system holds the result unchanged until the selected PLC acknowledges receipt.
8. After acknowledgement, the result fields are cleared and the system becomes ready for the next log.

## 7. Customer requirements

### 7.1 Start-up and readiness

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| CR-OPS-001 | Mandatory | On start-up, the system shall load the last successfully saved valid settings. | Test |
| CR-OPS-002 | Mandatory | If settings are missing, corrupt, or invalid, the system shall inform the operator and shall not behave as though it is producing reliable measurements. | Test |
| CR-OPS-003 | Mandatory | The system shall not indicate "Ready" until the selected PLC source and selected 3D data source are ready. When Live Cameras is selected, every enabled camera shall be ready. All three cameras shall be enabled in the default configuration. | Test |
| CR-OPS-004 | Mandatory | The system shall clearly display the selected PLC source and 3D data source while inspections can be started. | Demonstration |
| CR-OPS-005 | Mandatory | After a controlled shutdown or an unexpected restart, the system shall return to a safe, defined initial state with the PLC. | Test |
| CR-OPS-006 | Mandatory | The settings shall allow External PLC or Built-in PLC to be selected as the PLC source, independently of selecting Live Cameras or Recorded 3D Data as the inspection input. | Demonstration |
| CR-OPS-007 | Mandatory | A change to either selected source shall not take effect during an active inspection or while a result awaits PLC acknowledgement. The new selection shall take effect only after the current transaction has completed and the newly selected sources are ready. | Test |

### 7.2 Inspection and measurement

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| CR-INS-001 | Mandatory | Each valid PLC trigger shall create exactly one inspection. A trigger that remains active for an extended period shall not cause a second inspection. | Test |
| CR-INS-002 | Mandatory | Each inspection shall have a unique identifier, and the same identifier shall be used on screen, in logs, in results, and in stored data. | Inspection/Test |
| CR-INS-003 | Mandatory | When Live Cameras is selected, the system shall support three independent 3D camera channels and collect data from enabled cameras within the same inspection. All three cameras shall be enabled by default, and each camera shall be capable of being disabled individually. | Test |
| CR-INS-004 | Mandatory | The system shall measure the log length in millimetres. | Reference-object test |
| CR-INS-005 | Mandatory | The system shall measure the minimum, median, and maximum stem diameters in millimetres. | Reference-object test |
| CR-INS-006 | Mandatory | The system shall report the log orientation as "butt end leading", "butt end trailing", or "unknown". | Labelled-data test |
| CR-INS-007 | Mandatory | The system shall report the presence or absence of flare as a separate result. | Labelled-data test |
| CR-INS-008 | Mandatory | The system shall evaluate log deflection around 360 degrees and report the maximum deflection together with the relevant angle and position. | Reference-data test/Demonstration |
| CR-INS-009 | Mandatory | The flare region and measurement segments determined to be unreliable shall not distort the normal stem-diameter or deflection calculation. | Reference-data test |
| CR-INS-010 | Mandatory | The system shall verify that measurement data is sufficient and reliable before producing a valid measurement. If the required data-quality criteria are not satisfied, the system shall return "Measurement Failure". | Reference-data/Failure-scenario test |
| CR-INS-011 | Mandatory | When the same reference object/data is measured repeatedly under the same conditions and with the same settings, the difference between the maximum and minimum values obtained for each of length, diameter, and deflection shall not exceed 10 mm. | Repeatability test |
| CR-INS-012 | Mandatory | The deflection measurement and product decision shall be completed using the calculation mode selected in the settings at the start of the inspection; the system shall not automatically switch to another mode during measurement. | Design review/Test |

### 7.3 Decision rules

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| CR-DEC-001 | Mandatory | The system shall distinguish three outcomes: "Accepted", "Rejected", and "Measurement Failure". | Test |
| CR-DEC-002 | Mandatory | The permitted deflection shall be calculated from the log length and a configurable deflection limit per metre. The default value shall be 15 mm per metre, and the operator shall be able to change it in the settings. | Calculation check/Demonstration |
| CR-DEC-003 | Mandatory | For a valid measurement, if the deflection is equal to or less than the permitted value, the outcome shall be "Accepted". | Boundary-value test |
| CR-DEC-004 | Mandatory | For a valid measurement, if the deflection is greater than the permitted value, the outcome shall be "Rejected". | Boundary-value test |
| CR-DEC-005 | Mandatory | When a reliable decision cannot be produced, the internal outcome shall be "Measurement Failure". To reduce production loss, the system shall send `Outcome=true` to the PLC so that the log proceeds along the acceptance path. The system shall also set a separate PLC `MeasurementError` tag so that this condition can be distinguished from a valid accepted measurement, and error details shall be recorded separately. | Failure-scenario test |
| CR-DEC-006 | Mandatory | The presence of flare alone shall not cause rejection. | Rule review/Test |
| CR-DEC-007 | Mandatory | The final deflection shall be the value produced by the measurement method; no deflection correction factor shall be applied to the result. | Calculation check/Test |
| CR-DEC-008 | Mandatory | An undetermined log orientation shall not affect the acceptance/rejection outcome; the orientation shall be recorded as "Unknown". | Rule review/Test |


### 7.4 PLC integration

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| CR-PLC-001 | Mandatory | The system shall manage idle, running, result-hold, and disabled states in accordance with the agreed PLC contract. A valid PLC trigger shall be a new inspection-request transition received while the system is Ready, no inspection or unacknowledged result is active, and the previous request has returned to its inactive state. Once a trigger is accepted, subsequent PLC or manual triggers shall not create another inspection until the current transaction is complete and the interface has re-armed. | Integration test |
| CR-PLC-002 | Mandatory | A valid final result shall not be presented to the PLC before measurement is complete. | Integration test |
| CR-PLC-003 | Mandatory | The system shall hold results unchanged until PLC acknowledgement is received. | Integration test |
| CR-PLC-004 | Mandatory | PLC acknowledgement shall be accepted only while a result is being held. After acknowledgement is received, the result-valid indication and result fields shall be cleared. A new trigger shall be accepted only after the inspection request and acknowledgement signals have returned to their inactive states and the system is Ready. | Integration test |
| CR-PLC-005 | Mandatory | Loss of communication with the selected PLC source shall prevent PLC-triggered inspections but shall not prevent local system operation. | Failure-scenario test |
| CR-PLC-006 | Mandatory | Existing PLC tags, data types, and units shall not be changed without an interface change approved by both parties. Modbus is not part of this integration. | Interface review |
| CR-PLC-007 | Mandatory | In a "Measurement Failure" condition, the PLC shall receive `Outcome=true` and `MeasurementError=true`. For a valid Accepted or Rejected result, `MeasurementError` shall be false. The system shall preserve the actual internal outcome as "Measurement Failure" on the operator screen and in the records. | Integration test |
| CR-PLC-008 | Mandatory | When Built-in PLC is selected, the system shall exchange inspection requests, status, results, and acknowledgements without requiring a physical PLC. Built-in PLC and External PLC shall use the same approved communication protocol and PLC data contract. | Integration test |
| CR-PLC-009 | Mandatory | Built-in PLC shall allow the user to set and clear the PLC-to-system signals required for an inspection, including the inspection request and result acknowledgement, and to observe the system-to-PLC status and result values. The same trigger, result-hold, acknowledgement, and re-arm rules shall apply with either PLC source. | Demonstration/Integration test |
| CR-PLC-010 | Mandatory | Only the selected PLC source shall exchange inspection requests and results with the system. When Built-in PLC is selected, its requests and results shall not be exchanged with External PLC. | Integration test |

### 7.5 Operator interface

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| CR-UI-001 | Mandatory | The main screen shall show the status of the selected PLC source and 3D data source. When Live Cameras is selected, it shall show the status of each camera channel. | Demonstration |
| CR-UI-002 | Mandatory | The main screen shall show the latest inspection's identifier, outcome, length, diameters, deflection, permitted deflection, orientation, and flare information. | Demonstration |
| CR-UI-003 | Mandatory | Acceptance, rejection, and measurement failure shall be distinguished by explicit text as well as by colour. | Demonstration |
| CR-UI-004 | Mandatory | Counters for total triggers, completed inspections, acceptances, rejections, and measurement failures shall be displayed separately. | Test |
| CR-UI-005 | Mandatory | Results and counters shall be available separately for each PLC-source and 3D-data-source combination. The sources used for each result shall be visible. | Test |
| CR-UI-006 | Mandatory | Invalid settings shall not be saved; the affected field and reason for the error shall be shown to the operator. | Test |
| CR-UI-007 | Mandatory | An inspection in progress shall be completed using the settings present at its start; subsequent changes shall apply to the next inspection. | Test |
| CR-UI-008 | Recommended | Authorised users should be able to view raw, processed, and measured 3D data for diagnostic purposes. | Demonstration |
| CR-UI-009 | Mandatory | The operator interface shall provide a manual inspection command. The command shall be enabled only when the system is Ready to start a new inspection. PLC and manual requests shall be handled on a first-come, first-served basis: acceptance of either request shall immediately prevent every subsequent request from creating another inspection until the current inspection reaches its terminal state and the system becomes Ready again. A manually initiated inspection shall be clearly marked in the records. | Demonstration/Test |
| CR-UI-010 | Mandatory | When Built-in PLC is selected, the operator shall be able to open an independent Built-in PLC window. The window shall show the PLC-to-system signal values entered by the user and the system-to-PLC status and result values. | Demonstration |

### 7.6 Recorded data, recording, and reporting

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| CR-DATA-001 | Mandatory | The system shall be able to run the same measurement and decision implementation on recorded 3D data without requiring a physical camera. Live Cameras and Recorded 3D Data shall use the same application version, measurement algorithms, data-quality filters, settings interpretation, and decision rules. | Equivalence test |
| CR-DATA-002 | Mandatory | A single dataset and all datasets in a folder shall be executable through separate commands. | Demonstration |
| CR-DATA-003 | Mandatory | In batch processing of recorded 3D data, a corrupt dataset shall not prevent other datasets from being processed. | Test |
| CR-DATA-004 | Mandatory | The batch-processing summary shall include the outcome, measurements, duration, and error reason for each dataset. | Output review |
| CR-DATA-005 | Recommended | The batch-processing summary should be exportable in CSV or JSON format. | Demonstration |
| CR-DATA-006 | Mandatory | For each inspection, the start/end time, identifier, trigger source, selected PLC source, selected 3D data source, internal outcome, PLC `Outcome` and `MeasurementError` values where applicable, measurements, error, duration, application version, active calibration JSON file identifier/version, and the settings snapshot used for the inspection shall be recorded. Calibration data shall be maintained in a JSON file. | Record review |
| CR-DATA-007 | Mandatory | Storage of raw data for manual inspections, rejections, measurement failures, or diagnostic purposes shall be independently configurable. | Test |
| CR-DATA-008 | Mandatory | The application is not required to delete logs or raw data automatically based on age. Retention, archiving, backup, disk quotas, deletion, and storage-failure recovery shall be managed by IT/operations outside the application. | Design review/Demonstration |

## 8. Non-functional requirements

| ID | Priority | Requirement | Verification |
|---|---|---|---|
| CR-NFR-001 | Mandatory | The elapsed time from the PLC trigger until the result is ready shall be measured and recorded. | Record review |
| CR-NFR-002 | Mandatory | The user interface shall remain responsive during camera acquisition, 3D processing, and file writing. | Performance test |
| CR-NFR-003 | Mandatory | An unexpected error shall not cause subsequent inspections to stop silently; the system state and error reason shall be visible. | Resilience test |
| CR-NFR-004 | Mandatory | If a measurement cannot be produced because of camera or data quality, the error shall be visible and traceable. In accordance with the customer business rule, the internal outcome shall remain "Measurement Failure" while `Outcome=true` and `MeasurementError=true` are sent for line flow. | Failure-scenario test |
| CR-NFR-005 | Mandatory | An endurance test shall demonstrate that memory and device-resource usage do not grow without control during prolonged operation. | Endurance test |
| CR-NFR-006 | Mandatory | Millimetres shall be used for length, diameter, deflection, and longitudinal-position values, and degrees shall be used for angular values. The user interface, records, exported data, and PLC interface shall use the same documented units; any PLC scaling shall be defined in the PLC contract. | Inspection/Test |
| CR-NFR-007 | Mandatory | Every software delivery shall have a unique application version. Any change to the measurement algorithm shall require a new application version. | Version review |

| CR-NFR-009 | Mandatory | PLC source, 3D data source, PLC connection, camera, recording-path, and other settings shall be configurable without modifying the source code. | Demonstration |

## 9. Minimum customer acceptance scenarios

| ID | Scenario | Expected result |
|---|---|---|
| MKA-001 | A valid log within limits is inspected while the system is ready. | One inspection is created; measurements are published; the outcome is "Accepted" and is held until PLC acknowledgement. |
| MKA-002 | The deflection exceeds the permitted limit in a valid measurement. | The outcome is "Rejected"; the measured and permitted values are recorded. |
| MKA-003 | One of the enabled cameras times out or returns empty data. | The internal outcome is "Measurement Failure", the camera error is recorded, and `Outcome=true` with `MeasurementError=true` is sent to the PLC. |
| MKA-004 | A dataset fails one or more required data-quality filters. | No special or magic number is used in place of a physical measurement; the internal outcome is "Measurement Failure", and `Outcome=true` with `MeasurementError=true` is sent to the PLC. |
| MKA-005 | The PLC acknowledges the result. | The result-valid indication and result fields are cleared. A new trigger is accepted only after the request and acknowledgement signals are inactive and the system has returned to "Ready". |
| MKA-006 | The PLC trigger signal remains high. | A second inspection does not start for the same log. |
| MKA-007 | The deflection is exactly equal to the approved limit. | The outcome is "Accepted". |
| MKA-008 | An attempt is made to save an invalid setting. | The save is rejected and an explanatory field error is displayed. |
| MKA-009 | A batch of ten recorded 3D datasets is processed, including one corrupt dataset. | The corrupt dataset is reported as an error; the other nine datasets are processed, and the summary contains ten records. |
| MKA-010 | A decision setting is changed during an active inspection. | The current inspection is completed using the old setting, and the next inspection uses the new setting. |
| MKA-011 | The same reference object/data is processed repeatedly under the same conditions and with the same settings. | Screen, record, and PLC values use the same units; for each of length, diameter, and deflection, the difference between the maximum and minimum obtained values does not exceed 10 mm. |
| MKA-012 | The application restarts after an unexpected shutdown. | An old result is not published as though it were new, and a safe initial state is established with the PLC. |
| MKA-013 | Live Cameras is selected and one or two cameras are disabled in the settings. | The system waits only for enabled cameras, clearly identifies disabled cameras, and runs the inspection using data from the enabled cameras. |
| MKA-014 | Flare is detected in a valid measurement and the deflection is within limits. | The flare is recorded; it does not cause rejection by itself, and the outcome is "Accepted". |
| MKA-015 | Orientation cannot be determined and all other measurements are valid. | The orientation is recorded as "Unknown"; orientation uncertainty does not change the acceptance/rejection outcome. |
| MKA-016 | Reference data with a known deflection is processed with the deflection calculation mode set in turn to "One-way", "Two-way", and "Both". | Values for the selected mode are reported separately with correct labels; in "Both" mode, the larger value is used as the final deflection, and no correction factor is applied to any result. |
| MKA-017 | Stored logs and raw data must be archived or deleted under an external IT/operations policy. | The application does not perform automatic age-based deletion; the data can be managed outside the application. |
| MKA-018 | PLC and manual inspection requests are issued while the system is Ready. | The first request received starts exactly one inspection and its source is recorded. The later request does not create another inspection while the first inspection is active or its result transaction is incomplete. |
| MKA-019 | Built-in PLC and Live Cameras are selected, with the cameras ready and no physical PLC connected. The user opens the Built-in PLC window and issues an inspection request. | The system receives the request through the approved PLC protocol, creates one inspection, and displays its status and result values in the independent window. The record identifies Built-in PLC and Live Cameras as the sources. |
| MKA-020 | A Built-in PLC inspection request remains active; the user then clears it and sends result acknowledgement from the Built-in PLC window. | The active request does not create a second inspection. The result remains unchanged until valid acknowledgement; after acknowledgement, the result fields are cleared and a new request is accepted only after the interface has re-armed. No request or result is exchanged with External PLC. |
| MKA-021 | Each PLC source is selected in turn with each 3D data source, with the selected sources ready. | Inspections use the selected PLC and 3D data sources in each of the four combinations. The current selections and sources in each inspection record are visible, and results and counters for the four combinations can be viewed separately. |
| MKA-022 | The operator changes the PLC or 3D data source selection while an inspection is active or its result awaits acknowledgement. | The active transaction continues with its original sources; the new selection takes effect only after that transaction is complete and the newly selected sources are ready. |

## 11. Customer inputs and responsibilities

The customer or a party authorised by the customer shall provide and approve the following inputs:

- Measurement ranges and references against which the 10 mm repeatability parameter will be verified
- Confirmation of the production deflection limit per metre when a value other than the default 15 mm per metre is required
- Placement, field of view, and calibration references for the three cameras
- PLC protocol, tag list, and data types
- Acceptance/rejection business rules and line behaviour in the event of failure
- Reference logs/datasets and expected results
- Disk quota, archiving, backup, and deletion policy for raw data and logs, to be operated outside the application
- Production-network access, user permissions, and cybersecurity rules
- FAT and SAT environments, test time, and acceptance authorities

The affected requirements cannot receive final verification until these inputs have been provided.


## 12. Traceability and change management

Each customer requirement is tracked by a unique `CR-*` identifier. System requirements, design items, and acceptance tests shall reference the relevant customer-requirement identifier.

After approval, changes to scope, business rules, interfaces, or acceptance criteria shall be managed through a written change request. At a minimum, the change request shall identify the affected requirements, cost/schedule impact, verification needs, and approving parties.

## 13. Assumptions and dependencies

- The log is presented in a mechanically suitable position within the camera field of view and remains sufficiently stable during measurement.
- The camera, PLC, network, and computer hardware are suitable for production conditions and are operational.
- Suitable physical references are provided for calibration and its periodic verification.
- The PLC interface and line sequence can be tested jointly with the automation team.
- Customer-verified samples are provided for acceptance of the measurement algorithm.
- Site conditions such as ambient light, vibration, contamination, temperature, and network load are maintained within the agreed operating range.

## Appendix A — Revision history

| Version | Date | Change |
|---|---|---|
| 0.5 | 1 October 2026 | Added independent PLC and 3D data source selection, Built-in PLC window, and related acceptance scenarios. |
