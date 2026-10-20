# Clean-In-Place (CIP) Skid Automation Design Package & PackML Mapping

## 📌 Project Context & Overview
This repository contains the full industrial control design package for a modular **Clean-In-Place (CIP) Skid Subsystem (CIP-201)**. 

In high-volume pharmaceutical manufacturing, cross-contamination between harsh cleaning chemicals (such as sodium hydroxide or phosphoric acid) and active drug-product lines poses a severe GxP product-quality risk. This project implements the **ISA-TR88 (PackML) standardized state machine architecture** alongside explicit, interlocked **block-and-bleed communication handshakes** to cleanly isolate, verify, and track process lines throughout chemical rinsing, washing, and sanitizing execution tasks.

---

## 🏗️ Architectural Topology & Handshake Infrastructure
The control package establishes a secure, cross-chassis peer-to-peer data bridge over an industrial **EtherNet/IP network**, coordinating operations directly between the localized CIP Skid PAC and distant Target Storage Vessels:

```text
  [ LEVEL 2: SUPERVISORY CONTROL AND DATA ACQUISITION / SCADA ]
                             ▲
                             │ (OPC UA / Machine State Tracking)
                             ▼
  [ LEVEL 1: DETACHED PROGRAMMABLE AUTOMATION CONTROLLERS ]
    ┌────────────────────────┴────────────────────────┐
    ▼                                                 ▼
[ CIP Skid PAC (CIP-201) ] ◄───[EtherNet/IP]───► [ Target Production Vessel PAC ]
  - PackML Logic Execution Model                   - Local Routing Cluster Logic
  - Temperature Control Loops                      - Inlet Valve Isolation Checks
    │                                                 │
    ▼ (Hardwired Signals)                             ▼ (Hardwired Signals)
  [ LEVEL 0: RAW FIELD PROCESS INSTRUMENTATION ]
    - VFD-201 (Distribution Pump Drive)
    - TT-202 (In-Line RTD Sensor)
    - QT-203 (Toroidal Conductivity Transmitter)
    - PSH-204 (Return Discharge Pressure Switch)
```

### Deterministic Interlock Controls:
* **The Handshake Protocol:** The CIP Skid PAC is hard-interlocked to prevent chemical pumping unless a verified `Vessel Ready` handshake byte is successfully scanned from the production area.
* **The Safety Override:** If process instrumentation registers anomalies—such as loop temperature dropping below GxP chemical activation limits (75°C)—the system instantly pauses timers and drops execution to a safe `HOLDING` loop, recirculating fluids locally rather than risking an out-of-spec fluid transfer.

---

## 📁 Repository Directory Structure

```text
📁 pharma-cip-automation
├── 📁 docs
│   ├── 📝 Project2_CIP_Skid_PackML_FDS.md        <- PackML sequence design & handshakes
│   └── 📊 Project2_CIP_Skid_Sequence_Schedule.csv <- Timing, chemical, & temperature matrices
└── 📄 README.md                                  <- Main project description & architectural brief
```

---

## 📑 Core Documentation Breakdown

### 1. Functional PackML Sequence Specification
Located at [`docs/Project2_CIP_Skid_PackML_FDS.md`](./docs/Project2_CIP_Skid_PackML_FDS.md), this design document contains the exact logic routines governing the skid's execution cycles, state mappings, and loop fail-safe matrices.

### 2. Multi-Phase Sequence & Timing Matrix
Located at [`docs/Project2_CIP_Skid_Sequence_Schedule.csv`](./docs/Project2_CIP_Skid_Sequence_Schedule.csv), this data configuration schedule maps out the dynamic process recipe variables (Pre-Rinse, Caustic, Acid, Final WFI Flush), step durations, flow rates, and chemical thresholds.
