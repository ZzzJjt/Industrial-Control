(* Text-Based PFD for Penicillin Fermentation Process *)
(* Industrial-Scale Batch, 10,000 L Fermenter, Penicillium chrysogenum *)
(* Format: Piping (Pxx) --> Instrumentation (FTxxx, TTxxx) --> Controller (FICxxx) --> Actuator (FCVxxx) --> Equipment *)
(* Units: Water Tank, Nutrient Mixer, Sterilizer, Fermenter, Separator, Product Tank *)
(* Tags: FT (Flow Transmitter), LT (Level), TT (Temperature), PT (Pressure), pH (pH), DO (Dissolved Oxygen), FIC (Flow Controller), LIC (Level), TIC (Temperature), pHC (pH), DOC (DO), SIC (Speed), FCV (Flow Control Valve), LCV (Level), PMP (Pump), DP (Dosing Pump), AG (Agitator) *)
(* Signals: Analog (4-20 mA, REAL) unless digital (24 V DC, BOOL) *)
(* Execution: <1 ms per cycle, scan-cycle safe for 10 ms PLC cycles *)

=== Water Tank ===
(* Supplies purified water to nutrient mixer *)
P01 --> FT101 --> FIC101 --> FCV101 --> P02 --> MX01
  (* FT101: Water Flow, 0-100 L/min, FIC101 setpoint 70 L/min *)
  (* FCV101: Water Control Valve, 0-100%, setpoint 70% *)
P01 --> LT101 --> LIC101 --> LCV101 --> P03 --> MX01
  (* LT101: Tank Level, 0-100%, LIC101 setpoint 80% *)
  (* LCV101: Inlet Valve, 0-100%, setpoint 80% *)
WT01 --> TT101 --> TIC101 --> FCV102 --> P04
  (* WT01: Water Tank *)
  (* TT101: Water Temperature, 10-50°C, TIC101 setpoint 25°C *)
  (* FCV102: Cooling Valve, 0-100%, setpoint 50% *)
  (* Control: PID loop maintains level and temperature *)
  (* Interlock: IF LT101 > 95% THEN close LCV101 *)
  (* Alarm: IF TT101 > 30°C FOR >5 min THEN alert operator *)

=== Nutrient Mixer ===
(* Blends water, glucose, nutrients *)
P02 --> FT102 --> FIC102 --> FCV103 --> P05 --> MX01
  (* FT102: Glucose Flow, 0-20 L/min, FIC102 setpoint 14 L/min *)
  (* FCV103: Glucose Valve, 0-100%, setpoint 70% *)
P03 --> FT103 --> FIC103 --> FCV104 --> P06 --> MX01
  (* FT103: Nutrient Flow, 0-10 L/min, FIC103 setpoint 7 L/min *)
  (* FCV104: Nutrient Valve, 0-100%, setpoint 70% *)
MX01 --> LT102 --> LIC102 --> LCV101
  (* MX01: Nutrient Mixer *)
  (* LT102: Mixer Level, 0-100%, LIC102 setpoint 85% *)
  (* LCV101: Shared with Water Tank, controls total inflow *)
MX01 --> AG101 --> SIC101
  (* AG101: Agitator *)
  (* SIC101: Agitator Speed, 50-200 RPM, setpoint 100 RPM *)
  (* Control: PID for flow and level, manual glucose/nutrient dosing via HMI *)
  (* Interlock: IF LT102 > 90% THEN close FCV103, FCV104 *)
  (* Alarm: IF SIC101 < 80 RPM FOR >1 min THEN alert for motor failure *)

=== Sterilizer ===
(* Sterilizes media to prevent contamination *)
P05 --> FT104 --> FIC104 --> FCV105 --> P07 --> ST01
  (* FT104: Media Flow, 0-100 L/min, FIC104 setpoint 50 L/min *)
  (* FCV105: Media Inlet Valve, 0-100%, setpoint 50% *)
ST01 --> TT102 --> TIC102 --> FCV106 --> P08
  (* ST01: Sterilizer *)
  (* TT102: Sterilization Temp, 100-150°C, TIC102 setpoint 121°C *)
  (* FCV106: Steam Valve, 0-100%, setpoint 50% *)
ST01 --> PT101 --> PIC101 --> PCV101 --> P09
  (* PT101: Sterilizer Pressure, 0-5 bar, PIC101 setpoint 2 bar *)
  (* PCV101: Pressure Relief Valve, 0-100%, setpoint 50% *)
  (* Control: PID for temp and pressure, batch sterilization at 121°C, 2 bar for 30 min *)
  (* Interlock: IF TT102 < 121°C OR PT101 < 2 bar FOR >5 min THEN halt process, alarm *)
  (* Alarm: IF TT102 > 130°C FOR >1 min THEN open PCV101, alert *)

=== Fermenter ===
(* Conducts penicillin fermentation *)
P07 --> FT105 --> FIC105 --> FCV107 --> P10 --> FR01
  (* FT105: Sterile Media Flow, 0-100 L/min, FIC105 setpoint 80 L/min *)
  (* FCV107: Fermenter Inlet Valve, 0-100%, setpoint 80% *)
FR01 --> LT103 --> LIC103 --> FCV107
  (* FR01: Fermenter, 10,000 L *)
  (* LT103: Fermenter Level, 0-100%, LIC103 setpoint 90% *)
FR01 --> TT103 --> TIC103 --> FCV108 --> P11
  (* TT103: Fermentation Temp, 20-40°C, TIC103 setpoint 25°C *)
  (* FCV108: Cooling Water Valve, 0-100%, setpoint 50% *)
FR01 --> pH201 --> pHC201 --> DP201 --> P12
  (* pH201: pH Probe, 4.0-7.0, pHC201 setpoint 5.2 *)
  (* DP201: Acid/Base Dosing Pump, 0-5 L/min, setpoint 2 L/min *)
FR01 --> DO201 --> DOC201 --> FCV109 --> P13
  (* DO201: Dissolved Oxygen, 0-100%, DOC201 setpoint 40% *)
  (* FCV109: Air Sparger Valve, 0-100%, setpoint 50% *)
FR01 --> AG201 --> SIC201
  (* AG201: Fermenter Agitator *)
  (* SIC201: Agitator Speed, 100-300 RPM, setpoint 150 RPM *)
FR01 --> FLT201 --> LIC201 --> DP202 --> P14
  (* FLT201: Foam Level, 0-100%, LIC201 setpoint 20% *)
  (* DP202: Antifoam Dosing Pump, 0-2 L/min, setpoint 0.5 L/min *)
  (* Control: PID for temp, pH, DO, agitation; pulse dosing for antifoam *)
  (* Interlock: IF LT103 > 95% THEN close FCV107; IF TT103 > 30°C THEN ESD *)
  (* Alarm: IF pH201 < 5.0 OR > 5.5 FOR >30 min THEN alert; IF FLT201 > 30% THEN alarm *)

=== Separator ===
(* Separates penicillin broth from biomass *)
P10 --> FT106 --> FIC106 --> FCV110 --> P15 --> SP01
  (* FT106: Broth Flow, 0-50 L/min, FIC106 setpoint 30 L/min *)
  (* FCV110: Separator Inlet Valve, 0-100%, setpoint 60% *)
SP01 --> LT104 --> LIC104 --> FCV111 --> P16
  (* SP01: Centrifuge Separator *)
  (* LT104: Separator Level, 0-100%, LIC104 setpoint 50% *)
  (* FCV111: Outlet Valve, 0-100%, setpoint 50% *)
SP01 --> SS201
  (* SS201: Separator Running Status, digital, 0/1, expected 1 *)
SP01 <-- SC201
  (* SC201: Separator Start Command, digital, 0/1, setpoint 1 *)
  (* Control: PID for flow and level, manual start/stop via HMI *)
  (* Interlock: IF LT104 > 80% THEN close FCV110 *)
  (* Alarm: IF SS201 = 0 FOR >1 min THEN alert for separator failure *)

=== Product Tank ===
(* Stores clarified penicillin broth *)
P15 --> FT107 --> FIC107 --> FCV112 --> P17 --> PT01
  (* FT107: Product Flow, 0-50 L/min, FIC107 setpoint 30 L/min *)
  (* FCV112: Product Tank Inlet Valve, 0-100%, setpoint 60% *)
PT01 --> LT105 --> LIC105 --> FCV112
  (* PT01: Product Tank *)
  (* LT105: Tank Level, 0-100%, LIC105 setpoint 80% *)
PT01 --> TT104 --> TIC104 --> FCV113 --> P18
  (* TT104: Product Temperature, 0-20°C, TIC104 setpoint 4°C *)
  (* FCV113: Cooling Valve, 0-100%, setpoint 50% *)
  (* Control: PID for flow, level, temp; chilled storage at 4°C *)
  (* Interlock: IF LT105 > 95% THEN close FCV112 *)
  (* Alarm: IF TT104 > 8°C FOR >10 min THEN alert for cooling failure *)

(* Safety Interlocks *)
- IF TT102 < 121°C OR PT101 < 2 bar DURING STERILIZATION THEN halt, alarm
- IF TT103 > 30°C OR pH201 < 4.8 OR DO201 < 20% FOR >1 h THEN ESD
- IF FLT201 > 50% FOR >1 min THEN ESD, stop DP202
- IF LT103 > 95% OR LT105 > 95% THEN close FCV107, FCV112
- IF EStop = TRUE THEN close all valves, stop pumps, agitators

(* Notes *)
- Batch process: 5-7 days fermentation, 10,000 L per batch
- Control loops: PID for flow, level, temp, pH, DO; pulse dosing for antifoam
- Signals: 4-20 mA analog (REAL) unless digital (BOOL)
- Tags follow ISA-5.1, traceable to P&ID sheets
- Execution: <1 ms per cycle, supports 10 ms PLC scans
