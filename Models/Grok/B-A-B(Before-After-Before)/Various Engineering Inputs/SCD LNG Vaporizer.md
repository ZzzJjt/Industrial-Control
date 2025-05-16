```
NORSOK-Style System Control Diagram (SCD) for LNG Vaporizer System
-----------------------------------------------------------------
Description: Represents an LNG vaporizer system in a plain-text, NORSOK-compliant
format, detailing major process elements, instrumentation, control loops, interlocks,
and piping flow. The system converts LNG to gaseous natural gas through feed, 
vaporization, pressure regulation, and outlet stages, ensuring safety and efficiency.

Notation:
- Equipment: TK01 (LNG Feed Tank), VP01 (Vaporizer), HT01 (Heater), etc.
- Instruments: FT (Flow Transmitter), TT (Temperature Transmitter), FCV (Flow Control Valve).
- Flow: --> indicates material flow direction.
- Control Loops: [tag:controller-->actuator] (e.g., [FT201:FIC201-->FCV201]).
- Interlocks: Condition --> Action (e.g., PSH301 > 30 bar --> ESD1).
- Piping: Labeled (e.g., LNG1 for LNG Feed Line 1, GAS1 for Gas Outlet Line).
- Format: Grouped by process stage with indentation for readability.

Text-Based SCD:
===============

1. LNG Feed
-----------
  - LNG Feed Tank (TK01): Stores liquefied natural gas at -162°C.
    - LT101 (Level Transmitter, 0–100%): Monitors tank level.
    - LIC101 (Level Controller): Setpoint 70%, LIC101-->LV101 (feed valve).
    - TT102 (Temperature Transmitter, -200–0°C): Monitors LNG temperature.
    - PT103 (Pressure Transmitter, 0–10 bar): Monitors tank pressure.
    - LAH104 (High Level Alarm, 0/1): Triggers at 90% level.
    - LAL105 (Low Level Alarm, 0/1): Triggers at 20% level.
  - Piping: LNG1 (LNG Feed Line 1) --> [LT101:LIC101-->LV101] --> TK01
  - Control: Maintains tank level at 70% via PID-controlled feed valve; interlock stops LV101 if LAH104 = 1.

  - Feed Pump (P01): Delivers LNG to vaporizer.
    - FT106 (Flow Transmitter, 0–1000 m³/h): Monitors feed flow.
    - FIC106 (Flow Controller): Setpoint 500 m³/h, FIC106-->FV106 (pump valve).
    - PT107 (Pressure Transmitter, 0–50 bar): Monitors pump discharge pressure.
    - ST108 (Speed Transmitter, 0–3600 RPM): Monitors pump speed.
    - ZS109 (Status Switch, 0/1): Indicates pump running.
    - XA110 (Fault Alarm, 0/1): Indicates pump fault.
    - YC111 (Start/Stop Command, 0/1): Controls pump operation.
  - Piping: TK01 --> [FT106:FIC106-->FV106] --> P01 --> LNG2 (LNG Feed Line 2)
  - Control: Regulates feed flow at 500 m³/h via PID-controlled pump valve; interlock stops P01 if XA110 = 1 or LAL105 = 1.

2. Vaporization
---------------
  - Vaporizer (VP01): Converts LNG to gaseous natural gas at 10°C.
    - TT201 (Temperature Transmitter, -200–50°C): Monitors vaporizer outlet temperature.
    - TIC201 (Temperature Controller): Setpoint 10°C, TIC201-->TCV201 (heater valve).
    - PT202 (Pressure Transmitter, 0–50 bar): Monitors vaporizer pressure.
    - FT203 (Flow Transmitter, 0–1000 m³/h): Monitors gas flow.
    - PSH204 (Pressure Switch High, 0/1): Triggers at 30 bar.
    - TAH205 (High Temperature Alarm, 0/1): Triggers at 15°C.
  - Piping: LNG2 --> [FT203] --> VP01 --> GAS1 (Gas Line 1)
  - Control: Maintains outlet temperature at 10°C via PID loop TT201:TIC201-->TCV201; interlock reduces TCV201 if TAH205 = 1.

  - Heater (HT01): Provides heat to vaporizer.
    - TT206 (Temperature Transmitter, 0–100°C): Monitors heater fluid temperature.
    - TIC206 (Temperature Controller): Setpoint 80°C, TIC206-->TCV206 (heating valve).
    - FT207 (Flow Transmitter, 0–2000 L/min): Monitors heating fluid flow.
    - FIC207 (Flow Controller): Setpoint 1500 L/min, FIC207-->FV207 (fluid valve).
  - Piping: HTF1 (Heat Transfer Fluid Line 1) --> [FIC207:FV207] --> HT01 --> HTF2
  - Piping: HTF2 --> VP01 (heat exchange)
  - Control: Regulates heater fluid at 80°C and 1500 L/min via PID loops TIC206-->TCV206 and FIC207-->FV207.

3. Pressure Regulation
----------------------
  - Pressure Regulator (PR01): Maintains gas outlet pressure.
    - PT301 (Pressure Transmitter, 0–50 bar): Monitors outlet pressure.
    - PIC301 (Pressure Controller): Setpoint 25 bar, PIC301-->PCV301 (pressure valve).
    - PSHH302 (Pressure Switch High-High, 0/1): Triggers at 35 bar.
    - PSL303 (Pressure Switch Low, 0/1): Triggers at 15 bar.
  - Piping: GAS1 --> [PT301:PIC301-->PCV301] --> PR01 --> GAS2 (Gas Outlet Line 2)
  - Control: Maintains outlet pressure at 25 bar via PID loop PT301:PIC301-->PCV301; interlock opens PCV301 if PSL303 = 1.

4. Gas Outlet
-------------
  - Gas Outlet (GO01): Delivers gaseous natural gas to pipeline.
    - FT401 (Flow Transmitter, 0–1000 m³/h): Monitors outlet flow.
    - TT402 (Temperature Transmitter, 0–50°C): Monitors outlet temperature.
    - PT403 (Pressure Transmitter, 0–50 bar): Monitors outlet pressure.
    - PSV404 (Pressure Safety Valve, 40 bar): Mechanical relief.
    - XV405 (Emergency Shutdown Valve, 0/1): Closes on ESD.
  - Piping: GAS2 --> [FT401] --> GO01 --> Pipeline
  - Control: Monitors flow, temperature, and pressure; PSV404 opens at 40 bar; XV405 closes on ESD.

Interlocks and Safety:
- High Pressure Trip: PSH204 > 30 bar --> Reduce FIC106 to 50%.
- High-High Pressure Trip: PSHH302 > 35 bar --> ESD1 (Close XV405, Stop P01).
- Low Flow Trip: FT203 < 100 m³/h --> Reduce TCV201 to 0%.
- High Temperature Trip: TAH205 > 15°C --> Reduce TCV201 to 0%.
- Pump Fault: XA110 = 1 --> Stop YC111 (P01 shutdown).
- Low Tank Level: LAL105 = 1 --> Stop YC111 (P01 shutdown).
- Emergency Shutdown (ESD1): Triggered by PSHH302, manual XS501, or control room command.
  - Actions: Close XV405, Stop YC111, Open PCV301 to 100%, Close TCV201/TCV206.

Notes:
- Process: Converts LNG to gaseous natural gas via feed, vaporization, pressure regulation, and outlet stages.
- Equipment: TK01 (LNG Feed Tank), P01 (Feed Pump), VP01 (Vaporizer), HT01 (Heater), PR01 (Pressure Regulator), GO01 (Gas Outlet).
- Instrumentation: Follows ISA-5.1 and NORSOK I-005 (e.g., TT for Temperature Transmitter, FCV for Flow Control Valve).
- Flow: --> indicates material flow (e.g., LNG1 --> TK01).
- Control Loops: Denoted as [transmitter:controller-->actuator] (e.g., [PT301:PIC301-->PCV301]).
- Interlocks: Listed as condition --> action, ensuring safety (e.g., PSHH302 > 35 bar --> ESD1).
- Piping: Labeled (e.g., LNG1, GAS1, HTF1) for clarity.
- Ranges/Setpoints: Typical for LNG vaporizer system, adjustable per plant.
  - LT101: 0–100%, setpoint 70%.
  - FT106: 0–1000 m³/h, setpoint 500 m³/h.
  - TT201: -200–50°C, setpoint 10°C.
  - PT301: 0–50 bar, setpoint 25 bar.
- P&ID References: Hypothetical (U-610, L-120, etc.); actual drawings required.
- Safety: PSV404, XV405, and interlocks (e.g., PSHH302, TAH205) ensure compliance with NORSOK safety standards.
- Usage: Guides PLC configuration, HMI setup, safety verification, and control narrative development.
- Format: Plain-text, readable in code editors, suitable for documentation/collaboration.
- Assumptions:
  - LNG vaporizer system for medium-scale plant (~1 MMSCFD).
  - PLC supports 4–20 mA analog and 24 VDC digital I/O.
  - Signals and interlocks are typical for LNG vaporization.
  - NORSOK I-005 conventions apply for tags and control logic.
-----------------------------------------------------------------
```
