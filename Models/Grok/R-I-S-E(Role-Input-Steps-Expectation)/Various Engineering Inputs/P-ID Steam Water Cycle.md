(* Text-Based P&ID for Steam-Water Cycle *)
(* Power Plant: ~500 MW, High-Pressure Boiler *)
(* Format: Piping (Pxx) --> Instrumentation (FTxxx, LTxxx) --> Controller (FICxxx) --> Actuator (FCVxxx) --> Equipment *)
(* Blocks: Feedwater System, Boiler Section, Steam System, Turbine and Condenser, Condensate Return *)
(* Tags: FT (Flow Transmitter), LT (Level), PT (Pressure), TT (Temperature), FIC (Flow Controller), LIC (Level), PIC (Pressure), TIC (Temperature), FCV (Flow Control Valve), LCV (Level), PCV (Pressure), SIC (Speed), LSH/LSL (Level Switch High/Low) *)
(* All signals are 4-20 mA analog (REAL) unless noted as digital (BOOL) *)
(* Execution: <1 ms per cycle, scan-cycle safe for 10 ms PLC cycles *)

=== Feedwater System ===
(* Controls feedwater supply to boiler, maintains drum level *)
P01 --> FT101 --> FIC101 --> FCV101 --> P02 --> PMP01
  (* FT101: Feedwater Flow, 0-1000 t/h, FIC101 setpoint 500 t/h *)
  (* FCV101: Feedwater Control Valve, 0-100% *)
P02 --> TT101 --> TIC101 --> FCV102 --> P03 --> PMP01
  (* TT101: Feedwater Temperature, 100-200°C, TIC101 setpoint 150°C *)
  (* FCV102: Preheat Valve, 0-100%, controls economizer bypass *)
PMP01 --> SIC101 --> VFD101
  (* SIC101: Pump Speed Controller, 0-3600 RPM, setpoint 1800 RPM *)
  (* VFD101: Variable Frequency Drive, output 0-3600 RPM *)
PMP01 --> SS101
  (* SS101: Pump Running Status, digital, 0/1, expected 1 *)
PMP01 <-- SC101
  (* SC101: Pump Start Command, digital, 0/1, setpoint 1 *)
PMP01 <-- SC102
  (* SC102: Pump Stop Command, digital, 0/1, setpoint 0 *)

=== Boiler Section ===
(* Generates steam via economizer, drum, superheater *)
P03 --> EC01 --> TT102 --> TIC102 --> FCV103 --> P04
  (* EC01: Economizer *)
  (* TT102: Economizer Outlet Temp, 100-350°C, TIC102 setpoint 320°C *)
  (* FCV103: Cooling Water Valve, 0-100%, setpoint 50% *)
P04 --> DR01 --> LT101 --> LIC101 --> FCV101
  (* DR01: Steam Drum *)
  (* LT101: Drum Level, 0-100%, LIC101 setpoint 50% *)
  (* FCV101: Feedwater Valve, three-element control with FT101, FT103 *)
DR01 --> LSH101
  (* LSH101: Drum High Level Switch, digital, 0/1, triggers at 90% *)
DR01 --> LSL101
  (* LSL101: Drum Low Level Switch, digital, 0/1, triggers at 20% *)
DR01 --> PT101 --> PIC101 --> PCV101 --> P05
  (* PT101: Drum Pressure, 0-200 bar, PIC101 setpoint 150 bar *)
  (* PCV101: Pressure Control Valve, 0-100%, vents excess steam *)
P05 --> SH01 --> TT103 --> TIC103 --> FCV104 --> P06
  (* SH01: Superheater *)
  (* TT103: Superheater Steam Temp, 400-600°C, TIC103 setpoint 540°C *)
  (* FCV104: Attemperator Valve, 0-100%, injects water for temp control *)

=== Steam System ===
(* Delivers superheated steam to turbine *)
P06 --> FT103 --> FIC103 --> FCV105 --> P07
  (* FT103: Steam Flow, 0-1000 t/h, FIC103 setpoint 500 t/h *)
  (* FCV105: Steam Control Valve, 0-100%, setpoint 50% *)
P07 --> PT102 --> PIC102 --> PCV102 --> P08
  (* PT102: Steam Pressure, 0-200 bar, PIC102 setpoint 145 bar *)
  (* PCV102: Turbine Bypass Valve, 0-100%, diverts excess steam *)
P08 --> TT104 --> TIC104 --> FCV106 --> P09
  (* TT104: Steam Temperature, 400-600°C, TIC104 setpoint 540°C *)
  (* FCV106: Secondary Attemperator Valve, 0-100% *)

=== Turbine and Condenser ===
(* Converts steam energy to work, condenses exhaust *)
P09 --> TUR01 --> P10
  (* TUR01: Turbine *)
P10 --> FT104 --> FIC104 --> FCV107 --> P11
  (* FT104: Exhaust Steam Flow, 0-1000 t/h, FIC104 setpoint 500 t/h *)
  (* FCV107: Exhaust Control Valve, 0-100%, setpoint 50% *)
P11 --> CON01 --> LT102 --> LIC102 --> FCV108 --> P12
  (* CON01: Condenser *)
  (* LT102: Condenser Hotwell Level, 0-100%, LIC102 setpoint 60% *)
  (* FCV108: Hotwell Level Valve, 0-100%, setpoint 50% *)
CON01 --> TT105 --> TIC105 --> FCV109 --> P13
  (* TT105: Condenser Cooling Water Temp, 10-50°C, TIC105 setpoint 25°C *)
  (* FCV109: Cooling Water Valve, 0-100%, setpoint 50% *)
CON01 --> PT103 --> PIC103 --> PCV103 --> P14
  (* PT103: Condenser Vacuum Pressure, -1 to 0 bar, PIC103 setpoint -0.9 bar *)
  (* PCV103: Vacuum Control Valve, 0-100%, setpoint 50% *)

=== Condensate Return ===
(* Returns condensate to feedwater via deaerator *)
P12 --> PMP02 --> P15
  (* PMP02: Condensate Pump *)
PMP02 --> SIC102 --> VFD102
  (* SIC102: Condensate Pump Speed, 0-3600 RPM, setpoint 1800 RPM *)
  (* VFD102: Variable Frequency Drive, output 0-3600 RPM *)
PMP02 --> SS102
  (* SS102: Pump Running Status, digital, 0/1, expected 1 *)
PMP02 <-- SC103
  (* SC103: Pump Start Command, digital, 0/1, setpoint 1 *)
PMP02 <-- SC104
  (* SC104: Pump Stop Command, digital, 0/1, setpoint 0 *)
P15 --> DEA01 --> LT103 --> LIC103 --> FCV110 --> P16
  (* DEA01: Deaerator *)
  (* LT103: Deaerator Level, 0-100%, LIC103 setpoint 70% *)
  (* FCV110: Deaerator Water Valve, 0-100%, setpoint 50% *)
DEA01 --> PT104 --> PIC104 --> PCV104 --> P01
  (* PT104: Deaerator Pressure, 0-10 bar, PIC104 setpoint 2 bar *)
  (* PCV104: Deaerator Pressure Valve, 0-100%, setpoint 50% *)

(* Safety Interlocks *)
- IF LSH101 = 1 THEN close FCV101, stop PMP01 (drum overfill)
- IF LSL101 = 1 THEN open FCV101, start PMP01 (drum low level)
- IF PT101 > 180 bar THEN open PCV101, alarm (overpressure)
- IF TT103 > 600°C THEN open FCV104, alarm (superheater overtemp)
- IF PT103 > -0.8 bar THEN open PCV103, alarm (condenser vacuum loss)

(* Notes *)
- Three-element drum level control: LIC101 uses FT101 (feedwater flow), LT101 (drum level), FT103 (steam flow).
- All analog signals: 4-20 mA (REAL), digital: 24 V DC (BOOL).
- Piping (Pxx): Represents physical flow paths (e.g., P01 = feedwater inlet).
- Execution: <1 ms per cycle, supports 10 ms PLC scans.
- Tags follow ISA-5.1, traceable to P&ID sheets (not shown).
