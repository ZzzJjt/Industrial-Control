(* Text-Based System Control Diagram (SCD) for LNG Vaporizer System *)
(* Compliance: NORSOK I-005, Industrial LNG Regasification, 1000 m³/h *)
(* Format: Piping (Pxx) --> Instrumentation (FTxxx, TTxxx) --> Controller (FICxxx) --> Actuator (FCVxxx) --> Equipment *)
(* Sections: LNG Inlet, Vaporizer, Gas Outlet, Heating System *)
(* Tags: FT (Flow Transmitter), TT (Temperature), PT (Pressure), LT (Level), FIC (Flow Controller), TIC (Temperature), PIC (Pressure), FCV (Flow Control Valve), TCV (Temperature), PCV (Pressure), PSH (Pressure Switch High), TSH/TSL (Temperature Switch High/Low), ESD (Emergency Shutdown) *)
(* Signals: Analog (4-20 mA, REAL) unless digital (24 V DC, BOOL) *)
(* Execution: <1 ms per cycle, scan-cycle safe for 10 ms DCS/PLC cycles *)

=== LNG Inlet ===
(* Controls LNG flow into vaporizer *)
P01 --> FT201 --> FIC201 --> FCV201 --> P02 --> VAP01
  (* FT201: LNG Flow, 0-1500 m³/h, FIC201 setpoint 1000 m³/h *)
  (* FCV201: LNG Inlet Valve, 0-100%, setpoint 70% *)
  (* Control: PID loop maintains flow rate to meet demand *)
  (* Interlock: IF PSH301 = 1 THEN close FCV201 (ESD1) *)
  (* Alarm: IF FT201 > 1200 m³/h FOR >1 min THEN alert operator *)
P01 --> TT201 --> TIC201 --> FCV202 --> P03 --> VAP01
  (* TT201: LNG Inlet Temp, -170 to -100°C, TIC201 setpoint -162°C *)
  (* FCV202: LNG Bypass Valve, 0-100%, setpoint 10% *)
  (* Control: PID adjusts bypass to stabilize inlet temp *)
  (* Alarm: IF TT201 > -150°C FOR >2 min THEN alert for warming *)
P01 --> PT201 --> PIC201 --> PCV201 --> P04
  (* PT201: LNG Inlet Pressure, 0-10 bar, PIC201 setpoint 5 bar *)
  (* PCV201: Pressure Relief Valve, 0-100%, setpoint 50% *)
  (* Control: PID maintains safe inlet pressure *)
  (* Interlock: IF PT201 > 6 bar THEN open PCV201 *)

=== Vaporizer ===
(* Vaporizes LNG to gas using ambient air and supplemental heating *)
P02 --> VAP01 --> TT101 --> TIC101 --> TCV101 --> P05
  (* VAP01: Ambient Air Vaporizer *)
  (* TT101: Vaporizer Outlet Temp, -50 to 20°C, TIC101 setpoint 10°C *)
  (* TCV101: Hot Water Valve, 0-100%, setpoint 50% *)
  (* Control: PID loop adjusts hot water flow to maintain gas temp *)
  (* Interlock: IF TSH302 = 1 OR TSL302 = 1 THEN close TCV101 (ESD2) *)
  (* Alarm: IF TT101 < 0°C OR > 15°C FOR >5 min THEN alert *)
VAP01 --> PT301 --> PIC301 --> PCV301 --> P06
  (* PT301: Vaporizer Pressure, 0-12 bar, PIC301 setpoint 8 bar *)
  (* PCV301: Pressure Control Valve, 0-100%, setpoint 50% *)
  (* Control: PID maintains outlet pressure *)
  (* Interlock: IF PSH301 = 1 THEN close TCV101 (ESD1) *)
VAP01 --> PSH301
  (* PSH301: High Pressure Switch, digital, triggers at 10 bar *)
  (* Interlock: IF PSH301 = 1 THEN ESD1: close FCV201, TCV101 *)
VAP01 --> TSH302
  (* TSH302: High Temp Switch, digital, triggers at 20°C *)
VAP01 --> TSL302
  (* TSL302: Low Temp Switch, digital, triggers at -20°C *)
  (* Interlock: IF TSH302 = 1 OR TSL302 = 1 THEN ESD2: close FCV201 *)

=== Gas Outlet ===
(* Delivers vaporized gas to distribution *)
P05 --> FT301 --> FIC301 --> FCV301 --> P07 --> OUT01
  (* FT301: Gas Flow, 0-1500 m³/h, FIC301 setpoint 1000 m³/h *)
  (* FCV301: Gas Outlet Valve, 0-100%, setpoint 70% *)
  (* Control: PID ensures stable gas delivery *)
  (* Alarm: IF FT301 < 800 m³/h OR > 1200 m³/h FOR >2 min THEN alert *)
P07 --> TT301 --> TIC301 --> TCV101
  (* TT301: Gas Outlet Temp, 0-30°C, TIC301 setpoint 10°C *)
  (* TCV101: Shared with Vaporizer, reinforces temp control *)
  (* Control: PID loop, secondary temp check *)
  (* Alarm: IF TT301 > 15°C FOR >5 min THEN alert *)
P07 --> PT302 --> PIC302 --> PCV302 --> P08
  (* PT302: Gas Outlet Pressure, 0-12 bar, PIC302 setpoint 8 bar *)
  (* PCV302: Outlet Pressure Valve, 0-100%, setpoint 50% *)
  (* Control: PID maintains delivery pressure *)
  (* Alarm: IF PT302 > 9 bar FOR >1 min THEN alert *)

=== Heating System ===
(* Supplies hot water for supplemental heating *)
P09 --> FT401 --> FIC401 --> FCV401 --> P10 --> HTR01
  (* FT401: Hot Water Flow, 0-500 L/min, FIC401 setpoint 250 L/min *)
  (* FCV401: Water Inlet Valve, 0-100%, setpoint 50% *)
HTR01 --> TT401 --> TIC401 --> FCV402 --> P11
  (* HTR01: Water Bath Heater *)
  (* TT401: Water Temp, 50-80°C, TIC401 setpoint 70°C *)
  (* FCV402: Steam Valve, 0-100%, setpoint 50% *)
  (* Control: PID maintains water temp for vaporizer *)
  (* Alarm: IF TT401 < 60°C OR > 85°C FOR >5 min THEN alert *)
HTR01 --> LT401 --> LIC401 --> LCV401 --> P12
  (* LT401: Heater Level, 0-100%, LIC401 setpoint 70% *)
  (* LCV401: Water Makeup Valve, 0-100%, setpoint 50% *)
  (* Control: PID maintains water level *)
  (* Interlock: IF LT401 < 20% THEN close FCV401, stop HTR01 *)
P10 --> P05
  (* P10: Hot water loop connects to TCV101 in Vaporizer *)
  (* Control: TCV101 regulates water flow to vaporizer *)

(* Emergency Shutdown (ESD) Logic *)
- ESD1 (High Pressure):
  - Trigger: PSH301 = 1 (PT301 > 10 bar)
  - Actions: Close FCV201, TCV101; open PCV301; alarm
  - Reset: Manual via HMI after PT301 < 8 bar
- ESD2 (Temp Out of Range):
  - Trigger: TSH302 = 1 (TT101 > 20°C) OR TSL302 = 1 (TT101 < -20°C)
  - Actions: Close FCV201; open PCV301; alarm
  - Reset: Manual via HMI after TT101 = 10 ±5°C
- Manual ESD:
  - Trigger: Operator input or plant-wide ESD
  - Actions: Close FCV201, TCV101, FCV301; open PCV301, PCV302; stop HTR01

(* Cause & Effect per NORSOK I-005 *)
- Cause: PSH301 = 1
  Effect: ESD1, close FCV201, TCV101
- Cause: TSH302 = 1 OR TSL302 = 1
  Effect: ESD2, close FCV201
- Cause: LT401 < 20%
  Effect: Close FCV401, stop HTR01
- Cause: FT201 > 1200 m³/h OR FT301 < 800 m³/h
  Effect: Alarm, operator check

(* Notes *)
- Process: LNG at -162°C, 5 bar to gas at 10°C, 8 bar
- Capacity: 1000 m³/h, ambient air vaporizer with hot water heating
- Signals: 4-20 mA analog (REAL) unless digital (BOOL)
- Tags: ISA-5.1, NORSOK I-005 compliant, traceable to P&ID
- Execution: <1 ms per cycle, supports 10 ms DCS/PLC scans
