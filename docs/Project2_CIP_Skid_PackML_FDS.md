# Functional Design Specification: PackML State Machine & Handshakes for CIP Skid (CIP-201)

## 1. Project Objective
To establish a standardized, robust control sequence for a modular Clean-In-Place (CIP) skid using the **ISA-TR88 (PackML)** state model definition. This architecture regulates automated chemical routing, heat exchanger control loops, and multi-tank coordination handshakes without manual line interventions.

## 2. PackML State Machine Mapping
The CIP controller follows standard execution states to guarantee valid sanitization loops and prevent chemical cross-contamination with active process vessels:

*   **IDLE State:** The CIP skid is fully charged with utilities (steam pressure >3 bar, soft water supply verified). The system sits waiting for a "Vessel Ready" handshake byte from the production area.
*   **STARTING State:** The PLC opens the main supply valves and starts the distribution pump via a localized Variable Frequency Drive (VFD-201) set to a low-speed priming frequency (25 Hz).
*   **EXECUTE State:** The active sanitization loop is actively running. The PLC executes timed recipe phases (Pre-Rinse, Caustic Wash, Acid Wash, Final Purge) while driving control valves based on temperature feedback (TT-202) and inline conductivity limits (QT-203).
*   **HOLDING State:** If an operational parameter drifts out of GxP specification limits (e.g., Caustic wash temperature drops below 75°C), the state machine suspends the recipe timers, shifts to the `HOLDING` state, isolates chemical dosing, and recirculates fluid locally to prevent un-sanitized discharge.

## 3. Production Tank Interlock Handshake Sequence
To prevent raw chemicals from accidental cross-contamination into food or drug contact spaces, a deterministic digital interlock handshake matrix is hardcoded between the **CIP Skid PLC** and the **Target Storage Vessel PLC**:

| Step | CIP Skid Controller Action | Communication Tag | Target Vessel Controller Action | Safety Interlock Status |
| :--- | :--- | :--- | :--- | :--- |
| 1.0 | Poll availability matrix. | CIP_Ready_To_Sanitize = 1 | Acknowledge safe path configuration. | Line Isolated (Block-and-Bleed Active) |
| 2.0 | Block local recirculation valves. | CIP_Flow_Forward = 1 | Open inlet routing cluster matrix. | Transfer Path Verified Open |
| 3.0 | Monitor return path pressure switches. | CIP_Return_Scavenge_On = 1 | Activate localized scavenge return pump. | Zero Backpressure Hazard Check |
| 4.0 | Complete sequence; shut down pumps. | CIP_Cycle_Complete = 1 | Isolate intake valves; lock out line. | Sanitization Complete Lockout Achieved |

## 4. GxP Alarm & Loop Fail-Safe Matrix

| Alarm ID | Tag Name | Trigger Condition | Operational Threat | Fail-Safe Corrective Action |
| :--- | :--- | :--- | :--- | :--- |
| ALM-201 | QT-203 | High Chemical Residual in Final Rinse | Batch chemical contamination | Hold step; open DI water flush paths; route to neutral effluent system. |
| ALM-202 | PSH-204 | Return Line Blockage (>3.5 Bar) | Piping system over-pressurization | Stop VFD-201 instantly; vent manifold lines to local floor drain. |
| ALM-203 | LSL-205 | Caustic Tank Low-Low Level | Pump cavitation / running dry | Trip distribution pump coil; drop control loops to `SUSPENDED` state. |
