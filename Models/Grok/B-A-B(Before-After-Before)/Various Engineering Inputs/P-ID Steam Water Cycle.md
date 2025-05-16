```
Text-Based P&ID for Steam-Water Cycle in Power Plant
--------------------------------------------------
Description: Represents the steam-water cycle of a power plant in a plain-text
format, detailing major equipment, instrumentation, piping connections, and
control loops. The cycle includes feedwater supply, steam generation, turbine
operation, and condensate recovery, with clear signal flow and control logic.

Notation:
- Equipment: B01 (Boiler), D01 (Drum), P01 (Pump), etc.
- Instruments: FT (Flow Transmitter), LT (Level Transmitter), FCV (Flow Control Valve).
- Flow: --> indicates process flow direction.
- Control Loops: e.g., FT101:FIC101-->FCV101 (transmitter:controller-->actuator).
- Piping: Named (e.g., FW1 for Feedwater Line 1).
- Format: Grouped by subsystem (Feedwater, Steam Generation, Condensation).

Text-Based P&ID:
================

1. Feedwater Supply
------------------
- Deaerator (DA01): Stores and deaerates feedwater.
  - LT101 (Level Transmitter, 0–100%): Monitors deaerator level.
  - LIC101 (Level Controller): Controls level setpoint 70%.
  - LV101 (Level Valve): LIC101-->LV101 (adjusts makeup water flow).
  - PT102 (Pressure Transmitter, 0–10 bar): Monitors deaerator pressure.
  - TT103 (Temperature Transmitter, 0–150°C): Monitors deaerator temperature.
- Piping: Makeup Water --> [LV101] --> DA01
- Piping: DA01 --> FW1 (Feedwater Line 1)

- Feedwater Pump (P01): Supplies feedwater to economizer.
  - FT104 (Flow Transmitter, 0–2000 kg/h): Monitors pump discharge flow.
  - PT105 (Pressure Transmitter, 0–200 bar): Monitors discharge pressure.
  - ST106 (Speed Transmitter, 0–3600 RPM): Monitors pump speed.
  - ZS107 (Status Switch, 0/1): Indicates pump running.
  - XA108 (Fault Alarm, 0/1): Indicates pump fault.
  - YC109 (Start/Stop Command, 0/1): Controls pump operation.
- Piping: FW1 --> P01 --> FW2 (Feedwater Line 2)

2. Steam Generation
-------------------
- Economizer (E01): Preheats feedwater.
  - FT110 (Flow Transmitter, 0–2000 kg/h): Monitors economizer inlet flow.
  - TT111 (Temperature Transmitter, 0–250°C): Monitors outlet temperature.
- Piping: FW2 --> [FT110] --> E01 --> FW3 (Feedwater Line 3)

- Boiler Drum (D01): Separates steam and water.
  - LT112 (Level Transmitter, 0–100%): Monitors drum level.
  - LIC112 (Level Controller): Controls level setpoint 50%.
  - FCV113 (Flow Control Valve, 0–100%): FT104:LIC112-->FCV113 (adjusts feedwater flow).
  - PT114 (Pressure Transmitter, 0–180 bar): Monitors drum pressure.
  - TT115 (Temperature Transmitter, 0–400°C): Monitors drum temperature.
  - LAH116 (High Level Alarm, 0/1): Triggers at 80% level.
  - LAL117 (Low Level Alarm, 0/1): Triggers at 20% level.
- Piping: FW3 --> [FT104:FCV113] --> D01
- Piping: D01 --> STM1 (Steam Line 1)

- Boiler (B01): Generates steam.
  - FT118 (Steam Flow Transmitter, 0–2500 kg/h): Monitors steam flow to turbine.
  - PT119 (Pressure Transmitter, 0–180 bar): Monitors boiler outlet pressure.
  - TT120 (Temperature Transmitter, 0–500°C): Monitors steam temperature.
- Piping: STM1 --> [FT118] --> B01 --> STM2 (Steam Line 2)

3. Turbine and Condensation
---------------------------
- Turbine (T01): Converts steam energy to mechanical work.
  - ST121 (Speed Transmitter, 0–3600 RPM): Monitors turbine speed.
  - PT122 (Pressure Transmitter, 0–10 bar): Monitors exhaust pressure.
- Piping: STM2 --> T01 --> STM3 (Exhaust Steam Line)

- Condenser (C01): Condenses exhaust steam to water.
  - LT123 (Level Transmitter, 0–100%): Monitors condensate level.
  - LIC123 (Level Controller): Controls level setpoint 60%.
  - LV124 (Level Valve, 0–100%): LIC123-->LV124 (adjusts condensate pump flow).
  - TT125 (Temperature Transmitter, 0–100°C): Monitors condensate temperature.
  - FT126 (Cooling Water Flow Transmitter, 0–5000 L/min): Monitors cooling flow.
- Piping: STM3 --> C01
- Piping: C01 --> COND1 (Condensate Line 1)

- Condensate Pump (P02): Returns condensate to deaerator.
  - FT127 (Flow Transmitter, 0–2000 kg/h): Monitors condensate flow.
  - PT128 (Pressure Transmitter, 0–10 bar): Monitors pump discharge pressure.
  - ZS129 (Status Switch, 0/1): Indicates pump running.
  - YC130 (Start/Stop Command, 0/1): Controls pump operation.
- Piping: COND1 --> [FT127:LV124] --> P02 --> COND2 (Condensate Line 2)
- Piping: COND2 --> DA01 (returns to deaerator)

Control Loops:
- Deaerator Level: LT101:LIC101-->LV101 (maintains 70% level).
- Feedwater Flow: FT104:LIC112-->FCV113 (maintains drum level 50%).
- Drum Level: LT112:LIC112-->FCV113 (cascade with FT104).
- Condensate Level: LT123:LIC123-->LV124 (maintains 60% level).

Notes:
- Equipment: B01 (Boiler), D01 (Drum), DA01 (Deaerator), etc., represent major components.
- Instrumentation: Follows ISA-5.1 (e.g., FT for Flow Transmitter, FCV for Flow Control Valve).
- Flow Direction: --> indicates process flow (e.g., FW1 --> P01).
- Control Loops: Denoted as transmitter:controller-->actuator (e.g., FT104:LIC112-->FCV113).
- Piping: Labeled (e.g., FW1, STM2, COND1) for clarity.
- Ranges/Setpoints: Typical for medium-sized boiler (e.g., 100 MW), adjustable per plant.
  - LT112: 0–100%, setpoint 50%.
  - FT104: 0–2000 kg/h, setpoint 1000 kg/h.
  - PT114: 0–180 bar, setpoint 160 bar.
- P&ID References: Hypothetical (P&ID-SW-001 to P&ID-SW-004); actual drawings required.
- Safety: LAL117 (low drum level) triggers boiler trip; XA108 (pump fault) halts P01.
- Usage: Guides PLC/HMI configuration, wiring, and control narrative development.
- Format: Plain-text, readable in code editors, suitable for documentation/email.
- Assumptions:
  - Single-drum boiler with economizer, turbine, and condenser.
  - PLC supports 4–20 mA analog and 24 VDC digital I/O.
  - Signals are typical for feedwater and steam control.
--------------------------------------------------
```
