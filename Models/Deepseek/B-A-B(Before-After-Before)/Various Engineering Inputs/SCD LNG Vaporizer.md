Here is a **text-based NORSOK-style System Control Diagram (SCD)** for an **LNG vaporizer system**, designed to be readable in plain-text environments such as documentation tools, collaborative platforms, or version-controlled engineering files.

This layout includes:

- Major process elements  
- Flow direction using arrows (`-->`)  
- Instrumentation with realistic tag numbers following standard conventions  
- Interlocks and ESD logic  
- Logical grouping of control functions by subsystem  
- Reference to typical P&ID areas (e.g., U-610, L-120)  

---

# ⚙️ Text-Based SCD: LNG Vaporizer System (NORSOK Style)

## 📍 Area U-610 – LNG Inlet & Feed Regulation

```
LNG Inlet --> FT201 --> FIC201 --> FCV201 --> ESDV201 --> Vaporizer VAP-01
               |                               |
               v                               v
           [P&ID Ref: U-610]            [P&ID Ref: U-620]
```

- **FT201** = LNG Feed Flow Transmitter  
- **FIC201** = Flow Indicating Controller  
- **FCV201** = Flow Control Valve  
- **ESDV201** = Emergency Shutdown Valve  
- **VAP-01** = LNG Vaporizer Unit  

> **Control Loop:** `FT201 → FIC201 → FCV201`  
> **Function:** Regulates LNG feed rate to vaporizer based on demand signal.

---

## 📍 Area U-620 – Vaporizer & Temperature Control

```
Vaporizer VAP-01:
    |--> TT101 --> TIC101 --> TCV101 <--- Steam Supply
    |       Temp Monitoring        Temp Control Valve
    |
    '--> LT102 --> LIC102 --> LV102 <--- Drain Line
             Level Monitoring      Level Control Valve

    |
    v
Gas Outlet --> PT301 --> PIC301 --> PCV301 --> Pressure Regulator
                   |                        |
                   v                        v
               PSHH301                  PSV301 (Relief Valve)
                   |
                   v
               ESD Logic: Trip Signal ESD1
```

- **TT101** = Vaporizer Outlet Temperature Transmitter  
- **TIC101** = Temperature Indicating Controller  
- **TCV101** = Temperature Control Valve (Steam modulation)  
- **LT102** = Vaporizer Liquid Level Transmitter  
- **LIC102** = Level Indicating Controller  
- **LV102** = Level Control Valve (Condensate drain)  
- **PT301** = Gas Outlet Pressure Transmitter  
- **PIC301** = Pressure Indicating Controller  
- **PCV301** = Pressure Control Valve  
- **PSHH301** = Pressure Switch High-High (Trip Setpoint)  
- **PSV301** = Pressure Safety Valve  
- **ESD1** = Emergency Shutdown Signal  

> **Control Loops:**
> - `TT101 → TIC101 → TCV101` — Regulates vaporizer temperature via steam input  
> - `LT102 → LIC102 → LV102` — Maintains condensate level within vaporizer shell  
> - `PT301 → PIC301 → PCV301` — Manages gas outlet pressure to downstream systems  

> **Interlock Logic:**
> - If `PSHH301` activates due to overpressure → `ESD1` initiated  
> - `ESD1` signal closes `ESDV201` and stops all vaporizer operation  
> - Triggers alarm: `PAHH-301` (High-High Pressure Alarm)

---

## 🔌 Area L-120 – Electrical & Automation Interface

```
PLC/DCS System:
    |
    |-- AI Inputs:
    |     - TT101 (Temp)
    |     - PT301 (Pressure)
    |     - FT201 (Flow)
    |     - LT102 (Level)
    |     - PSHH301 (High-High Pressure)
    |
    |-- AO Outputs:
    |     - TCV101 (Steam Control Valve)
    |     - FCV201 (LNG Feed Valve)
    |     - PCV301 (Pressure Control Valve)
    |     - LV102 (Drain Valve)
    |
    |-- DO Outputs:
          - ESDV201 Open/Close Command
          - ESD1 Trip Output
```

> This section represents the **automation interface** where instrumentation signals are connected to the PLC/DCS. It supports full traceability for **control logic, alarms, and interlocks**.

---

## 📋 Instrument Tag Summary

| Tag      | Description                      | Type             | Location              |
|----------|----------------------------------|------------------|------------------------|
| FT201    | LNG Feed Flow                    | Analog Input     | LNG Inlet              |
| FIC201   | LNG Flow Controller              | PID Block        | DCS / PLC              |
| FCV201   | LNG Flow Control Valve           | Analog Output    | LNG Feed Line          |
| TT101    | Vaporizer Outlet Temp            | Analog Input     | Vaporizer Outlet       |
| TIC101   | Temp Controller                  | PID Block        | DCS / PLC              |
| TCV101   | Steam Control Valve              | Analog Output    | Steam Inlet            |
| LT102    | Vaporizer Level                  | Analog Input     | Condensate Side        |
| LIC102   | Level Controller                 | PID Block        | DCS / PLC              |
| LV102    | Condensate Drain Valve           | Analog Output    | Condensate Drain       |
| PT301    | Gas Outlet Pressure              | Analog Input     | Gas Outlet             |
| PIC301   | Pressure Controller              | PID Block        | DCS / PLC              |
| PCV301   | Gas Outlet Control Valve         | Analog Output    | Gas Discharge          |
| PSHH301  | Pressure Switch High-High        | Digital Input    | Gas Outlet             |
| PSV301   | Pressure Relief Valve            | Passive Device   | Gas Outlet             |
| ESDV201  | Emergency Shutoff Valve          | Digital Output   | LNG Inlet              |
| ESD1     | Emergency Shutdown Signal        | Digital Output   | Safety System          |

---

## 🧾 Summary of Key Functions

| Function Type     | Component(s) Involved                          | Purpose                                      |
|-------------------|-----------------------------------------------|----------------------------------------------|
| Flow Control      | FT201 → FIC201 → FCV201                       | Regulates LNG feed to vaporizer              |
| Temperature Control| TT101 → TIC101 → TCV101                      | Maintains vaporizer temp via steam control   |
| Level Control     | LT102 → LIC102 → LV102                       | Drains condensate to maintain safe level     |
| Pressure Control  | PT301 → PIC301 → PCV301                      | Stabilizes gas outlet pressure               |
| Overpressure Protection | PSHH301 → ESD1 → ESDV201 + PSV301     | Prevents overpressure in vaporizer system    |
| ESD Logic         | ESD1                                         | Initiates system shutdown on critical fault  |

---

This **text-based SCD** provides a structured and standardized representation of the **LNG vaporizer system**, aligned with NORSOK-style notation and suitable for use in early design phases, control system development, or safety verification.

Would you like this exported in **Markdown**, extended with **alarm list**, **sequence of operations**, or formatted for **PLC logic block import**?
