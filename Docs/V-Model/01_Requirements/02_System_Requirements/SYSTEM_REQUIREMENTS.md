# Balcas System Requirements Specification

| Document field | Value |
|---|---|
| Document code | BLC-SRS-001 |
| Document type | System Requirements Specification (SRS) |
| Version | 0.1 |
| Status | Draft for technical and customer review |
| Date | 28 September 2026 |
| Prepared by |  |
| Source baseline | BLC-MGD-001, version 0.4 |
| System | Balcas 3D Log Inspection System |

## 1. Purpose

This specification defines the functional, interface, data, and quality requirements for the Balcas 3D Log Inspection System. It translates the approved customer needs into verifiable system-level requirements and provides the baseline for architecture, detailed design, implementation, and system verification.

Each normative system requirement has a unique `SYS-*` identifier, a priority, traceability to one or more customer requirements, and a verification method.

## 2. Scope and system boundary

The system shall acquire three-dimensional data from up to three camera channels, combine and assess that data, measure each log, determine its inspection outcome, exchange status and results with the PLC, present information to the operator, and retain traceable inspection records. It shall also execute the same measurement and decision process against recorded datasets in simulation mode.

## 3. Source and precedence

The normative source for this document is `CUSTOMER_REQUIREMENTS_DOCUMENT.md`, document BLC-MGD-001, version 0.4. If this specification conflicts with the approved customer requirements, the approved customer requirements take precedence until the discrepancy is resolved through change control.

Items marked `TBD` require an approved customer or stakeholder input. A requirement containing a TBD may be designed and tested provisionally, but cannot receive final acceptance until that input has been baselined.

## 4. Terms and definitions

| Term | Definition |
|---|---|
| Accepted | A valid measurement whose final deflection is equal to or below the permitted deflection. |
| Rejected | A valid measurement whose final deflection is above the permitted deflection. |
| Measurement Failure | An inspection for which a reliable measurement or decision cannot be produced. It is not a valid acceptance. |
| Inspection | One complete processing transaction for one log, from an accepted trigger through result acknowledgement or terminal simulation output. |
| Valid trigger | A PLC inspection request accepted according to the approved PLC handshake and the system readiness rules. |
| Manual trigger | An operator command that starts an inspection without a PLC request. |
| Enabled camera | A configured camera channel required to participate in the current production or test configuration. |
| Sector | A configured portion of the measurement region used to assess whether data coverage is sufficient. |
| One-way deflection | The approved one-way deflection calculation method. The detailed mathematical definition is a controlled algorithm specification. |
| Two-way deflection | The approved two-way deflection calculation method. The detailed mathematical definition is a controlled algorithm specification. |
| Result hold | The PLC handshake state in which a completed result remains stable pending PLC acknowledgement. |
| Settings snapshot | The immutable set of effective settings and versions captured for an inspection when that inspection starts. |
| Production mode | Operation using live camera input and the production PLC sequence. |
| Simulation mode | Operation using recorded 3D data without requiring physical cameras or a production PLC transaction. |

## 5. System context and operating principles

### 5.1 External actors and interfaces

| External actor/interface | Information exchanged |
|---|---|
| PLC | Trigger, handshake/acknowledgement, system state, result-valid indication, outcome, measurements, and interface diagnostics according to the approved PLC data contract |
| 3D cameras 1-3 | Connection and acquisition status, calibrated 3D data, acquisition errors, and channel identity |
| Operator | Mode and status display, results, errors, settings, manual command, diagnostics, and simulation commands |
| File/storage environment | Settings, inspection records, raw/processed datasets, exports, logs, and version information |
| IT/maintenance environment | Installation, backup, restoration, storage management, network configuration, and diagnostic access |

### 5.2 Nominal production sequence

1. The system loads and validates the approved configuration.
2. The system establishes PLC communication and connects to every enabled camera.
3. The system indicates Ready only after all readiness conditions are satisfied.
4. A valid PLC or eligible manual trigger creates one inspection and one immutable settings snapshot.
5. The system acquires data from all enabled cameras and evaluates data sufficiency.
6. The system calculates measurements and an internal outcome, or records Measurement Failure.
7. The system displays and records the result and, for a PLC-triggered production inspection, publishes it to the PLC.
8. The system holds PLC result fields stable until acknowledgement.
9. After acknowledgement, the system clears result fields and returns to Ready when all readiness conditions remain satisfied.

### 5.3 Required logical states

| State | Required behaviour |
|---|---|
| Initialising | Outputs are placed in a defined non-result state while settings and interfaces are checked. |
| Not Ready | A new production inspection is not accepted; the blocking condition is visible. |
| Ready | All configured production prerequisites are satisfied and one new inspection may be accepted. |
| Running | One inspection is active; another trigger for the same log cannot create another inspection. |
| Result Hold | A complete PLC result is stable and awaiting acknowledgement. |
| Disabled | Inspection operation is deliberately disabled in accordance with the PLC contract. |
| Simulation | Recorded data is processed separately from production results and counters. |

The architecture may use additional internal states, provided the externally observable behaviour satisfies this specification and the approved PLC contract.

## 6. Functional requirements

### 6.1 Configuration, start-up, and readiness

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-CFG-001 | Mandatory | At start-up, the system shall load the latest configuration that is both valid and marked as approved. | MGR-OPS-001 | Test |
| SYS-CFG-002 | Mandatory | The system shall validate the loaded configuration before enabling inspection operation. | MGR-OPS-001, MGR-OPS-002 | Test |
| SYS-CFG-003 | Mandatory | Configuration validation shall identify missing required values, unreadable or corrupt content, invalid types, values outside approved ranges, and inconsistent combinations. | MGR-OPS-002, MGR-UI-006 | Test |
| SYS-CFG-004 | Mandatory | If configuration validation fails, the system shall enter Not Ready, identify the affected setting and reason, and shall not present itself as capable of reliable production measurement. | MGR-OPS-002 | Test |
| SYS-CFG-005 | Mandatory | PLC communication, camera endpoints, enabled camera channels, recording locations, measurement parameters, and decision parameters shall be configurable without a source-code change. | MGR-NFR-010 | Demonstration |
| SYS-CFG-006 | Mandatory | The approved default production configuration shall enable all three camera channels. | MGR-OPS-003, MGR-INS-003 | Inspection |
| SYS-CFG-007 | Mandatory | A test configuration shall permit each camera channel to be enabled or disabled independently. | MGR-INS-003 | Test |
| SYS-CFG-008 | Mandatory | In Production mode, the system shall enter Ready only when the approved configuration is valid, PLC communication is ready, and every enabled camera is ready. | MGR-OPS-003 | Test |
| SYS-CFG-009 | Mandatory | A camera marked disabled shall not block readiness, and its disabled status shall be visible to the operator. | MGR-INS-003, MKA-013 | Test |
| SYS-CFG-010 | Mandatory | The system shall expose its current operating mode as exactly Production or Simulation. | MGR-OPS-004 | Demonstration |
| SYS-CFG-011 | Mandatory | On controlled shutdown or unexpected restart, the system shall initialise PLC-facing status and result fields to the safe values defined by the approved PLC data contract before it can enter Ready. | MGR-OPS-005 | Test |
| SYS-CFG-012 | Mandatory | After restart, the system shall not publish a result retained from a previous execution as a new result. | MGR-OPS-005, MKA-012 | Test |

### 6.2 Trigger handling and inspection lifecycle

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-LFC-001 | Mandatory | The system shall accept a new production inspection only while it is Ready. | MGR-PLC-001, MGR-UI-009 | Test |
| SYS-LFC-002 | Mandatory | Each accepted valid PLC trigger shall create exactly one inspection. | MGR-INS-001 | Test |
| SYS-LFC-003 | Mandatory | A PLC trigger that remains asserted shall not create a second inspection until the trigger has completed the re-arm condition defined in the approved PLC contract. | MGR-INS-001, MKA-006 | Integration test |
| SYS-LFC-004 | Mandatory | Each inspection shall be assigned a system-unique identifier before acquisition or simulation processing begins. | MGR-INS-002 | Test |
| SYS-LFC-005 | Mandatory | The inspection identifier shall be identical in the user interface, PLC-associated result where supported by the contract, application logs, inspection record, and stored raw or processed data metadata. | MGR-INS-002 | Inspection/Test |
| SYS-LFC-006 | Mandatory | At inspection start, the system shall capture an immutable settings snapshot and shall use that snapshot for all acquisition, measurement, and decision operations in that inspection. | MGR-UI-007, MGR-INS-012 | Test |
| SYS-LFC-007 | Mandatory | A saved setting change made while an inspection is active shall first become effective for the next inspection. | MGR-UI-007 | Test |
| SYS-LFC-008 | Mandatory | The system shall support an operator manual-inspection command. | MGR-UI-009 | Demonstration |
| SYS-LFC-009 | Mandatory | The manual-inspection command shall be enabled only when the system is ready to start a new inspection. | MGR-UI-009 | Test |
| SYS-LFC-010 | Mandatory | An inspection started by the manual command shall record its source as Manual. | MGR-UI-009 | Record review |
| SYS-LFC-011 | Mandatory | Trigger arbitration shall prevent a manual command and a concurrent PLC trigger associated with the same log from creating two inspections. | MGR-UI-009, MKA-019 | Integration test |
| SYS-LFC-012 | Mandatory | The system shall maintain separate counts for received valid triggers and completed inspections so incomplete transactions remain detectable. | MGR-UI-004 | Test |

### 6.3 Camera acquisition, registration, and data sufficiency

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-ACQ-001 | Mandatory | The system shall support three independently configured 3D camera channels. | MGR-INS-003 | Test |
| SYS-ACQ-002 | Mandatory | For one live inspection, the system shall request and associate one acquisition from every camera enabled in that inspection's settings snapshot. | MGR-INS-003 | Test |
| SYS-ACQ-003 | Mandatory | The system shall wait only for camera channels enabled in the current settings snapshot. | MGR-INS-003, MKA-013 | Test |
| SYS-ACQ-004 | Mandatory | Data returned by different cameras for one inspection shall retain camera-channel identity and the common inspection identifier. | MGR-INS-002, MGR-INS-003 | Inspection/Test |
| SYS-ACQ-005 | Mandatory | The system shall transform data from the enabled camera coordinate systems into the approved common coordinate system using the active calibration data. | CRD 3.1, CRD 11 | Reference-data test |
| SYS-ACQ-006 | Mandatory | Missing, empty, timed-out, unreadable, or otherwise unusable required camera data shall cause the inspection's internal outcome to be Measurement Failure. | MGR-DEC-005, MGR-NFR-005, MKA-003 | Failure-scenario test |
| SYS-ACQ-007 | Mandatory | The system shall evaluate valid-point coverage for every configured sector used by the measurement algorithm. | MGR-INS-010 | Reference-data test |
| SYS-ACQ-008 | Mandatory | A dataset shall satisfy the minimum coverage rule only if every defined sector contains at least one valid data point. | MGR-INS-010 | Boundary-value test |
| SYS-ACQ-009 | Mandatory | If any required sector contains no valid data point, the system shall not present a fabricated or interpolated value as a measured result. Where a numeric measurement field is required by the interface, values greater than 900 may be used as predefined error codes. Such values shall be treated as error codes, not measurements, and their meanings shall be defined in the approved interface contract. The internal outcome shall be set to Measurement Failure. | MGR-INS-010, MKA-004 | Failure-scenario test |
| SYS-ACQ-010 | Mandatory | Acquisition and data-quality errors shall identify the inspection, affected channel or sector where applicable, error category, and error description. | MGR-NFR-005, MGR-DATA-006 | Record review |

### 6.4 Measurement processing

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-MET-001 | Mandatory | For data that passes sufficiency checks, the system shall detect the log and calculate its length in millimetres. | MGR-INS-004, MGR-NFR-007 | Reference-object test |
| SYS-MET-002 | Mandatory | The system shall calculate and report minimum, median, and maximum normal-stem diameters in millimetres. | MGR-INS-005, MGR-NFR-007 | Reference-object test |
| SYS-MET-003 | Mandatory | The system shall report orientation using exactly one of: Butt End Leading, Butt End Trailing, or Unknown. | MGR-INS-006 | Labelled-data test |
| SYS-MET-004 | Mandatory | If orientation cannot be determined reliably, the system shall report Unknown rather than infer a leading or trailing direction. | MGR-INS-006, MGR-DEC-008 | Labelled-data test |
| SYS-MET-005 | Mandatory | The system shall report flare presence as a separate Boolean result. | MGR-INS-007 | Labelled-data test |
| SYS-MET-006 | Mandatory | The normal-stem diameter and deflection calculations shall exclude the detected flare region. | MGR-INS-009 | Reference-data test |
| SYS-MET-007 | Mandatory | The normal-stem diameter and deflection calculations shall exclude measurement segments classified as unreliable by the approved data-quality rules. | MGR-INS-009 | Reference-data test |
| SYS-MET-008 | Mandatory | The configurable deflection calculation mode shall permit exactly One-way, Two-way, or Both. | MGR-INS-008 | Inspection/Test |
| SYS-MET-009 | Mandatory | The system shall evaluate deflection over the complete 360-degree angular domain required by the approved measurement algorithm. | MGR-INS-008 | Reference-data test |
| SYS-MET-010 | Mandatory | For One-way mode, the system shall report the one-way deflection together with its relevant angle and longitudinal position. | MGR-INS-008 | Reference-data test |
| SYS-MET-011 | Mandatory | For Two-way mode, the system shall report the two-way deflection together with its relevant angle and longitudinal position. | MGR-INS-008 | Reference-data test |
| SYS-MET-012 | Mandatory | For Both mode, the system shall calculate and report separately labelled one-way and two-way results, each with its relevant angle and longitudinal position. | MGR-INS-008, MGR-DEC-009 | Reference-data test |
| SYS-MET-013 | Mandatory | The system shall use the deflection calculation mode captured in the inspection's settings snapshot and shall not switch modes automatically during that inspection. | MGR-INS-012 | Test |
| SYS-MET-015 | Mandatory | When the same approved reference object or dataset is processed repeatedly under the same conditions and settings, the range between the maximum and minimum result shall not exceed 10 mm for length, for each reported diameter, and for each applicable reported deflection. | MGR-INS-011 | Repeatability test |
| SYS-MET-016 | Mandatory | All physical measurements shall use millimetres (mm). | MGR-NFR-007 | Inspection/Test |

### 6.5 Decision logic

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-DEC-001 | Mandatory | Every completed inspection shall have exactly one internal outcome: Accepted, Rejected, or Measurement Failure. | MGR-DEC-001 | Test |
| SYS-DEC-002 | Mandatory | For a valid measurement, the system shall calculate permitted deflection as log length multiplied by the active customer-approved deflection limit per metre, with units converted consistently. | MGR-DEC-002 | Calculation check |
| SYS-DEC-003 | Mandatory | The customer-approved deflection limit per metre shall be configurable and versioned as a decision setting. Its initial approved value is 15 mm per metre. | MGR-DEC-002, CRD 11 | Inspection/Demonstration |
| SYS-DEC-004 | Mandatory | For a valid measurement, the system shall set the internal outcome to Accepted when final deflection is less than or equal to permitted deflection. | MGR-DEC-003 | Boundary-value test |
| SYS-DEC-005 | Mandatory | For a valid measurement, the system shall set the internal outcome to Rejected when final deflection is greater than permitted deflection. | MGR-DEC-004 | Boundary-value test |
| SYS-DEC-006 | Mandatory | In Both mode, the system shall use the larger of the one-way and two-way deflection values as final deflection for the decision. | MGR-DEC-009 | Calculation check/Test |
| SYS-DEC-007 | Mandatory | An unreliable measurement or decision shall produce Measurement Failure and shall not be counted, displayed, or recorded as a valid Accepted result. | MGR-DEC-005 | Failure-scenario test |
| SYS-DEC-008 | Mandatory | For Measurement Failure, the PLC-facing `Outcome` value shall be `true` so the log follows the acceptance path, while the internal outcome remains Measurement Failure. | MGR-DEC-005, MGR-PLC-007 | Integration test |
| SYS-DEC-009 | Mandatory | Flare presence by itself shall not change an otherwise Accepted result to Rejected. | MGR-DEC-006 | Rule review/Test |
| SYS-DEC-010 | Mandatory | An Unknown orientation shall not alter the acceptance/rejection decision. | MGR-DEC-008 | Rule review/Test |

### 6.6 PLC interface and handshake

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-PLC-001 | Mandatory | The PLC interface shall implement the Idle, Running, and Result Hold behaviours defined by the jointly approved PLC data contract. | MGR-PLC-001 | Interface review/Integration test |
| SYS-PLC-002 | Mandatory | The system shall not assert a final-result-valid indication before measurement and decision processing has reached a terminal outcome. | MGR-PLC-002 | Integration test |
| SYS-PLC-003 | Mandatory | The system shall publish a mutually consistent set of result fields before asserting the result-valid indication. | MGR-PLC-002 | Integration test |
| SYS-PLC-004 | Mandatory | While awaiting PLC acknowledgement, the system shall keep all PLC result fields and the result-valid indication unchanged. | MGR-PLC-003 | Integration test |
| SYS-PLC-005 | Mandatory | The system shall accept acknowledgement only in accordance with the approved PLC handshake sequence. | MGR-PLC-001, MGR-PLC-003 | Integration test |
| SYS-PLC-006 | Mandatory | After valid acknowledgement, the system shall clear result fields to their contract-defined idle values and remove the result-valid indication. | MGR-PLC-004 | Integration test |
| SYS-PLC-007 | Mandatory | After acknowledgement processing, the system shall return to Ready only if all readiness conditions remain satisfied. | MGR-PLC-004 | Integration test |
| SYS-PLC-008 | Mandatory | Loss of the PLC connection or heartbeat shall remove Ready and prevent acceptance of a new production trigger. | MGR-PLC-005 | Failure-scenario test |
| SYS-PLC-009 | Mandatory | Loss of the PLC connection or heartbeat shall produce a visible and recorded diagnostic and initiate controlled reconnection attempts. | MGR-PLC-005 | Failure-scenario test |
| SYS-PLC-010 | Mandatory | Reconnection shall re-establish a defined handshake state before the system may publish a result or return to Ready. | MGR-OPS-005, MGR-PLC-005 | Integration test |
| SYS-PLC-011 | Mandatory | The PLC interface shall use the approved tag names, data types, units, scaling, directions, and update semantics without unilateral change. | MGR-PLC-006 | Interface review |
| SYS-PLC-013 | Mandatory | The approved PLC data contract shall define the PLC-facing value for Accepted and Rejected outcomes; it shall define Measurement Failure as `Outcome=true`. The Accepted/Rejected Boolean mapping is TBD pending the approved contract. | MGR-PLC-006, MGR-PLC-007, CRD 11 | Interface review/Integration test |
| SYS-PLC-014 | Mandatory | A PLC communication interruption shall not cause a partially updated result set to be represented as a valid completed result. | MGR-PLC-002, MGR-NFR-004 | Failure-scenario test |

### 6.7 Operator interface

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-UI-001 | Mandatory | The main screen shall show connection/readiness status separately for Camera 1, Camera 2, Camera 3, and the PLC. | MGR-UI-001 | Demonstration |
| SYS-UI-002 | Mandatory | The interface shall identify each camera as Ready, Not Ready/Error, or Disabled as applicable. | MGR-UI-001, MKA-013 | Demonstration |
| SYS-UI-003 | Mandatory | The main screen shall show the latest inspection identifier, internal outcome, length, minimum/median/maximum diameters, applicable deflection result or results, permitted deflection, orientation, and flare presence. | MGR-UI-002 | Demonstration |
| SYS-UI-004 | Mandatory | If a numeric result is unavailable because of Measurement Failure, the interface shall indicate that it is unavailable and shall not display a sentinel or fabricated measurement as a physical value. | MGR-INS-010, MGR-UI-002 | Test |
| SYS-UI-005 | Mandatory | Accepted, Rejected, and Measurement Failure shall each be identified by explicit text; colour may supplement but shall not replace the text. | MGR-UI-003 | Demonstration |
| SYS-UI-006 | Mandatory | The interface shall display separate counters for total valid triggers, completed inspections, Accepted outcomes, Rejected outcomes, and Measurement Failure outcomes. | MGR-UI-004 | Test |
| SYS-UI-007 | Mandatory | Production counters and latest results shall be stored and displayed separately from simulation counters and results. | MGR-UI-005 | Test |
| SYS-UI-008 | Mandatory | Before saving a setting, the interface shall apply the same validity rules used by start-up configuration validation. | MGR-UI-006 | Test |
| SYS-UI-009 | Mandatory | The system shall refuse to save invalid settings and shall identify each affected field and the reason it is invalid. | MGR-UI-006 | Test |
| SYS-UI-010 | Mandatory | The interface shall display the current operating mode continuously while inspection or simulation commands are available. | MGR-OPS-004 | Demonstration |
| SYS-UI-011 | Mandatory | Errors that affect readiness, measurement validity, PLC communication, camera acquisition, or recording shall be visible with sufficient information to identify the affected subsystem and reason. | MGR-OPS-002, MGR-NFR-004, MGR-NFR-005 | Demonstration/Test |
| SYS-UI-012 | Recommended | The system should provide authorised diagnostic users with views of raw, processed, and measured 3D data associated with a selected inspection. | MGR-UI-008 | Demonstration |

### 6.8 Simulation and batch processing

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-SIM-001 | Mandatory | In Simulation mode, the system shall execute the production measurement and decision logic against recorded 3D data without requiring a physical camera. | MGR-DATA-001 | Equivalence test |
| SYS-SIM-002 | Mandatory | The simulation path shall use the same controlled measurement-algorithm version and decision-rule implementation as Production mode. | MGR-DATA-001, MGR-NFR-008 | Design review/Test |
| SYS-SIM-003 | Mandatory | The system shall provide separate commands to process one selected dataset and all supported datasets in a selected folder. | MGR-DATA-002 | Demonstration |
| SYS-SIM-004 | Mandatory | Failure to read or process one dataset during a batch shall not prevent the system from attempting the remaining datasets. | MGR-DATA-003 | Test |
| SYS-SIM-005 | Mandatory | Batch output shall contain one summary entry for every discovered input dataset, including datasets that fail processing. | MGR-DATA-003, MKA-009 | Output review |
| SYS-SIM-006 | Mandatory | Each batch summary entry shall contain dataset identity, internal outcome, available measurements, processing duration, and error reason when applicable. | MGR-DATA-004 | Output review |
| SYS-SIM-007 | Recommended | The batch summary should be exportable in CSV or JSON format. | MGR-DATA-005 | Demonstration |
| SYS-SIM-008 | Mandatory | Simulation records shall identify Simulation as their operating mode and shall not be published to the production PLC as live-log results. | MGR-UI-005, MGR-DATA-006 | Integration test/Record review |

### 6.9 Records, logging, and raw data

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-DAT-001 | Mandatory | For every inspection, the system shall create a record containing start time, end time, inspection identifier, trigger source, operating mode, internal outcome, available measurements, error information, elapsed duration, application version, measurement-algorithm version, and settings version. | MGR-DATA-006 | Record review |
| SYS-DAT-002 | Mandatory | The inspection record shall distinguish the internal outcome from the PLC-facing `Outcome` value. | MGR-DEC-005, MGR-PLC-007 | Record review |
| SYS-DAT-003 | Mandatory | Recorded timestamps shall use one documented time basis and sufficient precision to order inspection lifecycle events. | MGR-DATA-006 | Inspection/Test |
| SYS-DAT-004 | Mandatory | The system shall record errors and warnings with timestamp, inspection identifier when applicable, subsystem, category, and descriptive reason. | MGR-DATA-006, MGR-NFR-004 | Record review |
| SYS-DAT-005 | Mandatory | Raw-data retention shall be independently configurable for manual inspections, Rejected outcomes, Measurement Failure outcomes, and diagnostic capture. | MGR-DATA-007 | Test |
| SYS-DAT-006 | Mandatory | A raw-data storage decision shall be associated with the inspection's immutable settings snapshot. | MGR-UI-007, MGR-DATA-007 | Test |
| SYS-DAT-007 | Mandatory | If the configured recording destination is temporarily unavailable, the system shall preserve the completed inspection result for later recording and notify the operator of the storage failure. | MGR-DATA-008 | Failure-scenario test |
| SYS-DAT-008 | Mandatory | A recording failure shall not silently change, discard, or reclassify the calculated inspection outcome. | MGR-DATA-008 | Failure-scenario test |
| SYS-DAT-009 | Mandatory | Records retained after a temporary recording failure shall preserve their original inspection identifier and timestamps when later written. | MGR-INS-002, MGR-DATA-008 | Failure-scenario test |
| SYS-DAT-010 | Mandatory | The application shall not automatically delete inspection logs or raw datasets on the basis of age. | MGR-DATA-009 | Design review/Demonstration |
| SYS-DAT-011 | Mandatory | Stored records and datasets shall be accessible to the authorised external IT/operations mechanisms used for quota management, archival, backup, restoration, and deletion. | MGR-DATA-009 | Demonstration |
| SYS-DAT-012 | Mandatory | The system shall record the inspection elapsed time from accepted trigger to result-ready state for Production mode, and the corresponding processing start-to-result interval for Simulation mode. | MGR-NFR-001 | Record review/Test |

## 7. Non-functional requirements

### 7.2 Reliability, recovery, and endurance

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-QUA-006 | Mandatory | An unexpected processing error shall transition the affected inspection to a defined terminal or recoverable state and shall expose the system state and error reason. | MGR-NFR-004 | Resilience test |
| SYS-QUA-007 | Mandatory | An error in one inspection shall not silently prevent subsequent inspections; if recovery is possible the system shall return through Not Ready to Ready, and if recovery is not possible it shall remain visibly Not Ready. | MGR-NFR-004 | Resilience test |
| SYS-QUA-008 | Mandatory | Camera or data-quality failures shall remain traceable as Measurement Failure even though the PLC-facing `Outcome` is `true`. | MGR-NFR-005 | Failure-scenario test |
| SYS-QUA-009 | Mandatory | During the approved endurance-test duration and workload, system memory and camera/device resource usage shall remain bounded and shall not exhibit continuing growth attributable to unreleased application resources. The duration and workload are TBD in the system verification plan. | MGR-NFR-006 | Endurance test |
| SYS-QUA-010 | Mandatory | Resource monitoring shall be configurable as enabled or disabled. When enabled, the system shall record its memory and resource usage at one-minute intervals to support endurance testing and resource-leak detection. | MGR-NFR-006 | Endurance test |

### 7.3 Versioning, integrity, and access protection

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-QUA-010 | Mandatory | Every released software delivery shall have a unique application version. | MGR-NFR-008 | Version review |
| SYS-QUA-011 | Mandatory | Every released measurement algorithm shall have a unique measurement-algorithm version independently identifiable in inspection records. | MGR-NFR-008 | Version review/Record review |
| SYS-QUA-012 | Mandatory | Approved configuration content shall have a unique version or revision identifier recorded with each inspection. | MGR-DATA-006, MGR-NFR-008 | Record review |
| SYS-QUA-013 | Recommended | Critical calibration and decision settings should be protected from unauthorised modification. | MGR-NFR-009 | Security review |
| SYS-QUA-014 | Recommended | Changes to critical calibration and decision settings should create an audit entry identifying timestamp, setting, previous value, new value, and user or accountable actor where available. | MGR-NFR-009 | Security review/Test |

### 7.4 Maintainability and supportability

| ID | Priority | System requirement | Source | Verification |
|---|---|---|---|---|
| SYS-QUA-015 | Mandatory | Configuration, records, errors, and exported results shall use documented formats sufficient to support troubleshooting, regression testing, backup, and restoration. | CRD 10, MGR-NFR-010 | Documentation review/Demonstration |
| SYS-QUA-016 | Mandatory | The delivered system shall include approved default configuration, setting descriptions, operator instructions, installation/backup/restore instructions, error and troubleshooting information, release notes, and known limitations. | CRD 10 | Deliverable inspection |

## 8. External interface baselines

### 8.1 PLC data contract

Before final integration verification, the PLC and software teams shall jointly approve an interface control document defining at minimum:

- protocol and endpoint parameters;
- tag names, directions, data types, units, and scaling;
- trigger validity and re-arm semantics;
- Idle, Running, Result Hold, Disabled, and fault-state values;
- result fields and the atomic/result-valid publication sequence;
- acknowledgement semantics and timeout behaviour;
- heartbeat semantics, loss criteria, and reconnection behaviour;
- Boolean mapping for Accepted and Rejected, with Measurement Failure fixed to `Outcome=true`;
- restart behaviour for both the PLC and inspection system; and
- inspection identifier exchange if supported by the PLC interface.

Changes to this baseline require approval by both parties and a traceable interface change record.

### 8.2 Camera and calibration baseline

Before final measurement verification, the customer or authorised camera/calibration specialist shall approve:

- camera models, channel identities, endpoints, and production enablement;
- placement, field of view, and common-coordinate-system definition;
- calibration references and valid calibration revision;
- acquisition timeout and invalid-data criteria;
- environmental and presentation limits; and
- the process for periodic calibration verification.

Automatic calibration is not required by this specification.

### 8.3 Record and dataset interface

Record and dataset format specifications shall define field names, types, units, enumerations, version identifiers, timestamps, and error representation. Formats shall preserve the distinction between Accepted, Rejected, and Measurement Failure and between Production and Simulation.

## 9. Verification requirements and acceptance rules

### 9.1 Verification methods

| Method | Meaning |
|---|---|
| Test | Execute controlled inputs and compare observable output with the requirement. |
| Integration test | Exercise behaviour across the system and an external interface or representative simulator. |
| Reference-object/data test | Compare calculated results with an approved physical reference or approved reference dataset. |
| Boundary-value test | Exercise equality and adjacent values at a decision or validity boundary. |
| Failure-scenario/resilience test | Inject a defined failure and verify detection, state, recovery, indication, and records. |
| Demonstration | Observe required behaviour during representative operation. |
| Inspection/review | Examine configuration, records, design, interface definitions, or delivered documentation. |
| Calculation check | Independently calculate the expected result from controlled inputs. |
| Endurance/performance test | Operate under an approved workload while measuring timing, responsiveness, and resources. |

### 9.2 Verification principles

- Every Mandatory system requirement shall have at least one passed verification result before system acceptance.
- Recommended requirements not implemented shall have a written justification approved through project review.
- Verification evidence shall identify the system build, algorithm version, settings version, test data/reference, environment, date, and result.
- Requirements dependent on TBD inputs shall not be declared finally passed until the applicable input is approved and used in verification.
- Reference datasets and objects shall be versioned and protected from uncontrolled modification.
- FAT and SAT procedures shall be derived from the customer acceptance scenarios and the system requirements in this document.

## 10. Customer-to-system traceability

| Customer requirement | Derived system requirement(s) |
|---|---|
| MGR-OPS-001 | SYS-CFG-001, SYS-CFG-002 |
| MGR-OPS-002 | SYS-CFG-002, SYS-CFG-003, SYS-CFG-004, SYS-UI-011 |
| MGR-OPS-003 | SYS-CFG-006, SYS-CFG-008 |
| MGR-OPS-004 | SYS-CFG-010, SYS-UI-010 |
| MGR-OPS-005 | SYS-CFG-011, SYS-CFG-012, SYS-PLC-010 |
| MGR-INS-001 | SYS-LFC-002, SYS-LFC-003 |
| MGR-INS-002 | SYS-LFC-004, SYS-LFC-005, SYS-ACQ-004, SYS-DAT-009 |
| MGR-INS-003 | SYS-CFG-006, SYS-CFG-007, SYS-CFG-009, SYS-ACQ-001, SYS-ACQ-002, SYS-ACQ-003 |
| MGR-INS-004 | SYS-MET-001 |
| MGR-INS-005 | SYS-MET-002 |
| MGR-INS-006 | SYS-MET-003, SYS-MET-004 |
| MGR-INS-007 | SYS-MET-005 |
| MGR-INS-008 | SYS-MET-008, SYS-MET-009, SYS-MET-010, SYS-MET-011, SYS-MET-012 |
| MGR-INS-009 | SYS-MET-006, SYS-MET-007 |
| MGR-INS-010 | SYS-ACQ-007, SYS-ACQ-008, SYS-ACQ-009, SYS-UI-004 |
| MGR-INS-011 | SYS-MET-015 |
| MGR-INS-012 | SYS-LFC-006, SYS-MET-013 |
| MGR-DEC-001 | SYS-DEC-001 |
| MGR-DEC-002 | SYS-DEC-002, SYS-DEC-003 |
| MGR-DEC-003 | SYS-DEC-004 |
| MGR-DEC-004 | SYS-DEC-005 |
| MGR-DEC-005 | SYS-ACQ-006, SYS-DEC-007, SYS-DEC-008, SYS-DAT-002 |
| MGR-DEC-006 | SYS-DEC-009 |
| MGR-DEC-007 | SYS-MET-014 |
| MGR-DEC-008 | SYS-MET-004, SYS-DEC-010 |
| MGR-DEC-009 | SYS-MET-012, SYS-DEC-006 |
| MGR-PLC-001 | SYS-LFC-001, SYS-PLC-001, SYS-PLC-005 |
| MGR-PLC-002 | SYS-PLC-002, SYS-PLC-003, SYS-PLC-014 |
| MGR-PLC-003 | SYS-PLC-004, SYS-PLC-005 |
| MGR-PLC-004 | SYS-PLC-006, SYS-PLC-007 |
| MGR-PLC-005 | SYS-PLC-008, SYS-PLC-009, SYS-PLC-010 |
| MGR-PLC-006 | SYS-MET-016, SYS-PLC-011, SYS-PLC-012, SYS-PLC-013 |
| MGR-PLC-007 | SYS-DEC-008, SYS-PLC-013, SYS-DAT-002 |
| MGR-UI-001 | SYS-UI-001, SYS-UI-002 |
| MGR-UI-002 | SYS-UI-003, SYS-UI-004 |
| MGR-UI-003 | SYS-UI-005 |
| MGR-UI-004 | SYS-LFC-012, SYS-UI-006 |
| MGR-UI-005 | SYS-UI-007, SYS-SIM-008 |
| MGR-UI-006 | SYS-CFG-003, SYS-UI-008, SYS-UI-009 |
| MGR-UI-007 | SYS-LFC-006, SYS-LFC-007, SYS-DAT-006 |
| MGR-UI-008 | SYS-UI-012 |
| MGR-UI-009 | SYS-LFC-001, SYS-LFC-008, SYS-LFC-009, SYS-LFC-010, SYS-LFC-011 |
| MGR-DATA-001 | SYS-SIM-001, SYS-SIM-002 |
| MGR-DATA-002 | SYS-SIM-003 |
| MGR-DATA-003 | SYS-SIM-004, SYS-SIM-005 |
| MGR-DATA-004 | SYS-SIM-006 |
| MGR-DATA-005 | SYS-SIM-007 |
| MGR-DATA-006 | SYS-ACQ-010, SYS-DAT-001, SYS-DAT-003, SYS-DAT-004, SYS-QUA-012 |
| MGR-DATA-007 | SYS-DAT-005, SYS-DAT-006 |
| MGR-DATA-008 | SYS-DAT-007, SYS-DAT-008, SYS-DAT-009 |
| MGR-DATA-009 | SYS-DAT-010, SYS-DAT-011 |
| MGR-NFR-001 | SYS-DAT-012 |
| MGR-NFR-002 | SYS-DEC-011, SYS-QUA-001, SYS-QUA-002, SYS-QUA-003, SYS-QUA-004 |
| MGR-NFR-003 | SYS-QUA-005 |
| MGR-NFR-004 | SYS-PLC-014, SYS-UI-011, SYS-DAT-004, SYS-QUA-006, SYS-QUA-007 |
| MGR-NFR-005 | SYS-ACQ-006, SYS-ACQ-010, SYS-UI-011, SYS-QUA-008 |
| MGR-NFR-006 | SYS-QUA-009 |
| MGR-NFR-007 | SYS-MET-001, SYS-MET-002, SYS-MET-016 |
| MGR-NFR-008 | SYS-SIM-002, SYS-QUA-010, SYS-QUA-011, SYS-QUA-012 |
| MGR-NFR-009 | SYS-QUA-013, SYS-QUA-014 |
| MGR-NFR-010 | SYS-CFG-005, SYS-QUA-015 |

## 11. Acceptance-scenario coverage

| Customer scenario | Primary system requirements |
|---|---|
| MKA-001 | SYS-LFC-002, SYS-MET-001 through SYS-MET-014 as applicable, SYS-DEC-004, SYS-PLC-004 |
| MKA-002 | SYS-DEC-002, SYS-DEC-005, SYS-DAT-001 |
| MKA-003 | SYS-ACQ-006, SYS-DEC-008, SYS-ACQ-010 |
| MKA-004 | SYS-ACQ-007, SYS-ACQ-008, SYS-ACQ-009, SYS-DEC-008 |
| MKA-005 | SYS-PLC-005, SYS-PLC-006, SYS-PLC-007 |
| MKA-006 | SYS-LFC-003 |
| MKA-007 | SYS-DEC-004 |
| MKA-008 | SYS-UI-008, SYS-UI-009 |
| MKA-009 | SYS-SIM-004, SYS-SIM-005, SYS-SIM-006 |
| MKA-010 | SYS-LFC-006, SYS-LFC-007 |
| MKA-011 | SYS-MET-015, SYS-MET-016 |
| MKA-012 | SYS-CFG-011, SYS-CFG-012 |
| MKA-013 | SYS-CFG-007, SYS-CFG-009, SYS-ACQ-003, SYS-UI-002 |
| MKA-014 | SYS-MET-005, SYS-DEC-009 |
| MKA-015 | SYS-MET-004, SYS-DEC-010 |
| MKA-016 | SYS-MET-008 through SYS-MET-014, SYS-DEC-006 |
| MKA-017 | SYS-DEC-011, SYS-QUA-002, SYS-QUA-003, SYS-QUA-004 |
| MKA-018 | SYS-DAT-010, SYS-DAT-011 |
| MKA-019 | SYS-LFC-008, SYS-LFC-009, SYS-LFC-010, SYS-LFC-011 |

## 12. Open inputs and dependencies

The following inputs are required before the affected requirements and verification procedures can be fully baselined:

| ID | Required input | Owner | Affected requirements |
|---|---|---|---|
| TBD-001 | Approved maximum permitted deflection per metre | Process/quality owner | SYS-DEC-002, SYS-DEC-003, SYS-DEC-004, SYS-DEC-005 |
| TBD-002 | Approved mathematical definitions and reference implementation/data for one-way and two-way deflection | Process/quality owner/software supplier | SYS-MET-008 through SYS-MET-014, SYS-DEC-006 |
| TBD-003 | Measurement ranges, reference objects/datasets, expected values, and uncertainty | Process/quality owner | SYS-MET-001, SYS-MET-002, SYS-MET-015 |
| TBD-004 | Sector geometry and valid-point/data-quality rules | Camera/calibration specialist and process owner | SYS-ACQ-007 through SYS-ACQ-010 |
| TBD-005 | Camera placement, field of view, calibration, timing, and environment limits | Camera/calibration specialist | SYS-ACQ-001 through SYS-ACQ-006 |
| TBD-006 | Approved PLC interface control document, including Accepted/Rejected Boolean mapping | Automation/PLC team | SYS-LFC-003, SYS-CFG-011, SYS-PLC-001 through SYS-PLC-014 |
| TBD-008 | Endurance-test duration, production workload, and resource acceptance criteria | Customer and software supplier | SYS-QUA-009 |
| TBD-009 | Storage paths, capacity, external retention, archival, backup, restore, quota, and deletion policy | Maintenance/IT | SYS-DAT-005 through SYS-DAT-011 |
| TBD-010 | Production network, permissions, and cybersecurity constraints | Maintenance/IT | SYS-CFG-005, SYS-QUA-013, SYS-QUA-014 |
| TBD-011 | FAT/SAT environments, schedules, and acceptance authorities | Customer/project management | All acceptance verification |

## 13. Assumptions and constraints

- Logs are presented within the approved camera field of view and remain sufficiently stable during acquisition.
- Camera, PLC, network, computer, and storage hardware are suitable for the approved production environment and are operational.
- Calibration data and physical references are available and valid before production inspection.
- The PLC interface and line sequence can be tested with the automation team or an approved representative simulator.
- Customer-approved reference logs and datasets are available for measurement and decision verification.
- Ambient light, vibration, contamination, temperature, network load, and other site conditions remain within approved limits.
- Millimetres are the base unit for physical measurement results.
- Any change to scope, algorithm, decision rules, interface, or acceptance criteria is subject to documented impact assessment and approval.

## Appendix A — Revision history

| Version | Date | Author | Change | Approval |
|---|---|---|---|---|
| 0.1 | 28 September 2026 |  | Initial system requirements derived from BLC-MGD-001 version 0.4 |  |
