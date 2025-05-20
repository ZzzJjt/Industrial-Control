(* NORSOK-Compliant Text-Based SCD for LNG Vaporizer System *)
(* Details process flow, instrumentation, control loops, and safety interlocks *)
(* Uses ISA-style tags, arrows (-->) for flow/signals, and grouped sections *)
(* Designed for automation design, PLC configuration, and safety documentation *)

[Process Flow Diagram]

LNG Inlet (P-101) --> FT201 --> FIC201 --> FCV201 --> V-301 (Vaporizer)
                             |
                             v
                          TT202 --> TIC202 --> TCV202 --> H-401 (Hot Water Heater)
                             |
                             v
                          PT203 --> PIC203 --> PCV203 --> P-501 (Gas Outlet)

[Control Loops]

[Temperature Control Loop]
  TT202 --> TIC202 --> TCV202 (Hot Water Control Valve, 0–100%)
    * Measurement: TT202 (Temperature Transmitter, 0–100°C, Setpoint: 20°C)
    * Controller: TIC202 (Temperature Indicating Controller, PID)
    * Final Element: TCV202 (Modulates hot water flow to maintain 20°C ± 1°C)
    * Logic: TIC202 computes error from TT202 (vaporizer outlet temp) and setpoint, adjusting TCV202 to control heat input
    * Alarms: High Temp (>25°C), Low Temp (<15°C)

[Flow Control Loop]
  FT201 --> FIC201 --> FCV201 (LNG Feed Valve, 0–100%)
    * Measurement: FT201 (Flow Transmitter, 0–500 t/h, Setpoint: 400 t/h)
    * Controller: FIC201 (Flow Indicating Controller, PID)
    * Final Element: FCV201 (Modulates LNG flow to maintain 400 t/h ± 10 t/h)
    * Logic: FIC201 computes error from FT201 (LNG inlet flow) and setpoint, adjusting FCV201 to ensure steady feed
    * Alarms: High Flow (>450 t/h), Low Flow (<350 t/h)

[Pressure Control Loop]
  PT203 --> PIC203 --> PCV203 (Gas Outlet Valve, 0–100%)
    * Measurement: PT203 (Pressure Transmitter, 0–50 bar, Setpoint: 40 bar)
    * Controller: PIC203 (Pressure Indicating Controller, PID)
    * Final Element: PCV203 (Modulates gas outlet to maintain 40 bar ± 1 bar)
    * Logic: PIC203 computes error from PT203 (outlet pressure) and setpoint, adjusting PCV203 to stabilize gas delivery
    * Alarms: High Pressure (>45 bar), Low Pressure (<35 bar)

[Hot Water System Control Loop]
  TT401 --> TIC401 --> TCV401 (Heater Steam Valve, 0–100%)
    * Measurement: TT401 (Temperature Transmitter, 0–150°C, Setpoint: 80°C)
    * Controller: TIC401 (Temperature Indicating Controller, PID)
    * Final Element: TCV401 (Modulates steam to maintain 80°C ± 2°C in H-401)
    * Logic: TIC401 computes error from TT401 (hot water temp) and setpoint, adjusting TCV401 to ensure sufficient heat for V-301
    * Alarms: High Temp (>85°C), Low Temp (<75°C)

[Safety Interlocks]

[High Pressure Shutdown]
  Condition: PT203 > 47 bar OR PSH204 (High-Pressure Switch, 47 bar)
  Action: 
    * Close FCV201 (LNG Feed Valve)
    * Open ESDV101 (Emergency Shutdown Valve at LNG Inlet)
    * Stop P-401 (Hot Water Pump)
    * Alarm: High Pressure Trip
  Logic:
    IF PT203 > 47.0 OR PSH204 THEN
        FCV201_Open := FALSE;
        ESDV101_Open := TRUE;
        P_401_Running := FALSE;
        Alarm_HighPressure := TRUE;
    END_IF;

[Low Temperature Shutdown]
  Condition: TT202 < 10°C OR TSL205 (Low-Temperature Switch, 10°C)
  Action:
    * Close TCV202 (Hot Water Valve)
    * Close FCV201 (LNG Feed Valve)
    * Open ESDV101 (Emergency Shutdown Valve)
    * Alarm: Low Temperature Trip
  Logic:
    IF TT202 < 10.0 OR TSL205 THEN
        TCV202_Open := FALSE;
        FCV201_Open := FALSE;
        ESDV101_Open := TRUE;
        Alarm_LowTemp := TRUE;
    END_IF;

[Low Flow Interlock]
  Condition: FT201 < 300 t/h for 30 seconds
  Action:
    * Reduce TCV202 opening by 10%
    * Alarm: Low Flow Warning
    * If FT201 < 250 t/h for 60 seconds, close FCV201 and trigger ESDV101
  Logic:
    IF FT201 < 300.0 THEN
        LowFlowTimer(IN := TRUE, PT := T#30s);
        IF LowFlowTimer.Q THEN
            TCV202_Setpoint := TCV202_Setpoint * 0.9;
            Alarm_LowFlow := TRUE;
        END_IF;
        IF FT201 < 250.0 THEN
            CriticalFlowTimer(IN := TRUE, PT := T#60s);
            IF CriticalFlowTimer.Q THEN
                FCV201_Open := FALSE;
                ESDV101_Open := TRUE;
                Alarm_CriticalFlow := TRUE;
            END_IF;
        END_IF;
    ELSE
        LowFlowTimer(IN := FALSE);
        CriticalFlowTimer(IN := FALSE);
    END_IF;

[High Hot Water Temperature Interlock]
  Condition: TT401 > 90°C OR TSH402 (High-Temperature Switch, 90°C)
  Action:
    * Close TCV401 (Steam Valve)
    * Stop P-401 (Hot Water Pump)
    * Alarm: High Hot Water Temperature
  Logic:
    IF TT401 > 90.0 OR TSH402 THEN
        TCV401_Open := FALSE;
        P_401_Running := FALSE;
        Alarm_HighHotWaterTemp := TRUE;
    END_IF;

(* Equipment List *)
(* P-101: LNG Inlet Pipe *)
(* V-301: LNG Vaporizer *)
(* H-401: Hot Water Heater *)
(* P-401: Hot Water Pump *)
(* P-501: Gas Outlet Pipe *)

(* Instrumentation List *)
(* FT201: Flow Transmitter (0–500 t/h) *)
(* TT202, TT401: Temperature Transmitters (0–100°C, 0–150°C) *)
(* PT203: Pressure Transmitter (0–50 bar) *)
(* FIC201: Flow Indicating Controller *)
(* TIC202, TIC401: Temperature Indicating Controllers *)
(* PIC203: Pressure Indicating Controller *)
(* FCV201: LNG Feed Control Valve (0–100%) *)
(* TCV202, TCV401: Temperature Control Valves (0–100%) *)
(* PCV203: Pressure Control Valve (0–100%) *)
(* ESDV101: Emergency Shutdown Valve (Open/Closed) *)
(* PSH204: High-Pressure Switch (47 bar) *)
(* TSL205: Low-Temperature Switch (10°C) *)
(* TSH402: High-Temperature Switch (90°C) *)

(* Control Philosophy *)
(* Temperature: Maintains vaporizer outlet at 20°C (TIC202) and hot water at 80°C (TIC401) using PID loops to modulate TCV202 and TCV401, ensuring efficient vaporization without freezing or overheating. *)
(* Flow: Regulates LNG feed at 400 t/h (FIC201) via PID control of FCV201 to balance throughput and system stability. *)
(* Pressure: Controls gas outlet at 40 bar (PIC203) via PID adjustment of PCV203, preventing overpressure in downstream systems. *)
(* Safety: Interlocks (high pressure, low temp, low flow, high hot water temp) trigger ESDV101 and stop critical equipment to protect system integrity, with alarms for operator intervention. *)

(* Notes *)
(* - Arrows (-->) indicate material flow (e.g., LNG, gas, hot water) or signal paths *)
(* - Tags follow ISA/NORSOK format: XXNNN (e.g., FT201, TIC202) *)
(* - Control loops use PID for dynamic regulation of flow, temperature, and pressure *)
(* - Interlocks ensure safety compliance with NORSOK standards (e.g., I-501 for safety systems) *)
(* - Structured layout with grouped sections enhances readability for documentation *)
(* - Supports PLC programming, HMI design, and safety validation *)
