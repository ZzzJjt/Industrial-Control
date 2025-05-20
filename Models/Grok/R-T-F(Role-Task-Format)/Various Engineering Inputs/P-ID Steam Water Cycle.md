(* Text-Based P&ID for Steam-Water Cycle in Power Plant *)
(* Describes feedwater, boiler, steam, and condensate return sections *)
(* Uses ISA-style tags and text arrows (-->) for flow and signal paths *)
(* Designed for documentation and control system integration *)

[Feedwater Supply Section]
  P01 (Deaerator Tank) --> FT101 --> FIC101 --> FCV101 --> P02 (Feedwater Pump)
  |                        |                           |
  |                        v                           v
  |                   LT102 --> LIC102 --> PCV102 --> P02
  |                        |
  |                        v
  |                   TT103 --> TIC103 --> TCV103 --> P01
  |
  v
  P03 (Economizer) --> FT104 --> FIC104 --> FCV104 --> B01

[Boiler Section]
  B01 (Boiler/Steam Drum) --> LT105 --> LIC105 --> FCV101 (Feedwater Valve)
  |                           |
  |                           v
  |                        PT106 --> PIC106 --> PCV106 --> S01
  |                           |
  |                           v
  |                        TT107 --> TIC107 --> FCV108 --> B01
  |
  v
  S01 (Superheater) --> TT109 --> TIC109 --> TCV109 --> T01

[Steam Section]
  T01 (Turbine) --> PT110 --> PIC110 --> PCV110 --> C01
  |                 |
  |                 v
  |              FT111 --> FIC111 --> FCV111 --> C01

[Condensate Return Section]
  C01 (Condenser) --> LT112 --> LIC112 --> PCV112 --> P04
  |                   |
  |                   v
  |                TT113 --> TIC113 --> TCV113 --> C01
  |
  v
  P04 (Condensate Pump) --> FT114 --> FIC114 --> FCV114 --> P01

(* Control Loop Descriptions *)
(* Feedwater Flow Control: FT101 measures flow from P01, FIC101 computes error from setpoint (150 t/h), FCV101 adjusts valve *)
(* Drum Level Control: LT105 measures drum level, LIC105 adjusts FCV101 to maintain 50% level *)
(* Boiler Pressure Control: PT106 measures pressure, PIC106 controls PCV106 to maintain 160 bar *)
(* Superheater Temperature Control: TT109 measures steam temperature, TIC109 adjusts TCV109 to maintain 500°C *)
(* Turbine Steam Flow Control: FT111 measures steam flow, FIC111 adjusts FCV111 to maintain 150 t/h *)
(* Condensate Level Control: LT112 measures condenser level, LIC112 controls PCV112 to maintain 60% *)

(* Equipment List *)
(* P01: Deaerator Tank, P02: Feedwater Pump, P03: Economizer, B01: Boiler/Steam Drum *)
(* S01: Superheater, T01: Turbine, C01: Condenser, P04: Condensate Pump *)

(* Instrumentation List *)
(* FT101, FT104, FT111, FT114: Flow Transmitters (0–200 t/h) *)
(* LT102, LT105, LT112: Level Transmitters (0–100%) *)
(* TT103, TT107, TT109, TT113: Temperature Transmitters (0–600°C) *)
(* PT106, PT110: Pressure Transmitters (0–200 bar) *)
(* FIC101, FIC104, FIC111, FIC114: Flow Indicating Controllers *)
(* LIC102, LIC105, LIC112: Level Indicating Controllers *)
(* TIC103, TIC107, TIC109, TIC113: Temperature Indicating Controllers *)
(* PIC106, PIC110: Pressure Indicating Controllers *)
(* FCV101, FCV104, FCV108, FCV111, FCV114: Flow Control Valves (0–100%) *)
(* PCV102, PCV106, PCV110, PCV112: Pressure Control Valves (0–100%) *)
(* TCV103, TCV109, TCV113: Temperature Control Valves (0–100%) *)

(* Notes *)
(* - Arrows (-->) indicate physical flow (e.g., water, steam) or signal paths *)
(* - Control loops are closed (e.g., FT101 --> FIC101 --> FCV101) for automated regulation *)
(* - Tags follow ISA format: XX-NNN (e.g., FT=Flow Transmitter, 101=unique ID) *)
(* - Setpoints: Feedwater Flow=150 t/h, Drum Level=50%, Pressure=160 bar, Steam Temp=500°C *)
(* - Diagram is grouped by process section for clarity and P&ID integration *)
