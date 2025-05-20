(* Text-Based PFD for Penicillin Fermentation Process *)
(* Details process units, flow paths, instrumentation, and control philosophy *)
(* Uses ISA-style tags and arrows (-->) for material and signal flows *)
(* Designed for automation design and process control implementation *)

[Process Flow Diagram]

Water Tank (T-101) --> Nutrient Mixer (M-201) --> Sterilizer (S-301) --> Fermenter (F-401)
                                              |                          |
                                              v                          v
                                          Separator (S-501) --> Product Tank (T-601)

[Process Units and Instrumentation]

T-101: Water Tank
  Instruments:
    LT101: Level Transmitter (0–100%, Setpoint: 80%)
    FT101: Water Flow Transmitter (0–500 L/h, Setpoint: 400 L/h)
    TT101: Temperature Transmitter (0–50°C, Setpoint: 25°C)
  Control Loops:
    LT101 --> LIC101 --> FCV101 (Water Inlet Valve, 0–100%)
      * LIC101 maintains 80% level via PID control of FCV101
    FT101 --> FIC101 --> FCV101
      * FIC101 ensures 400 L/h flow to M-201
    TT101 --> TIC101 --> TCV101 (Cooling Water Valve, 0–100%)
      * TIC101 maintains 25°C to prevent microbial growth

M-201: Nutrient Mixer
  Instruments:
    FT201: Nutrient Flow Transmitter (0–50 kg/h, Setpoint: 40 kg/h)
    ST201: Agitator Speed Transmitter (0–200 RPM, Setpoint: 150 RPM)
    LT201: Mixer Level Transmitter (0–100%, Setpoint: 70%)
  Control Loops:
    FT201 --> FIC201 --> FCV201 (Nutrient Dosing Valve, 0–100%)
      * FIC201 controls nutrient feed at 40 kg/h via PID
    ST201 --> SIC201 --> MCV201 (Agitator Motor, 0–100%)
      * SIC201 maintains 150 RPM for uniform mixing
    LT201 --> LIC201 --> FCV202 (Outlet Valve, 0–100%)
      * LIC201 maintains 70% level to ensure mixing capacity

S-301: Sterilizer
  Instruments:
    TT301: Sterilizer Temperature Transmitter (0–150°C, Setpoint: 121°C)
    PT301: Pressure Transmitter (0–5 bar, Setpoint: 1.013 bar)
    FT301: Sterilized Media Flow Transmitter (0–500 L/h, Setpoint: 400 L/h)
  Control Loops:
    TT301 --> TIC301 --> TCV301 (Steam Control Valve, 0–100%)
      * TIC301 maintains 121°C for 15 minutes via PID to ensure sterility
    PT301 --> PIC301 --> PCV301 (Pressure Relief Valve, 0–100%)
      * PIC301 maintains 1.013 bar to support sterilization
    FT301 --> FIC301 --> FCV301 (Outlet Valve, 0–100%)
      * FIC301 controls flow to F-401 at 400 L/h

F-401: Fermenter
  Instruments:
    TT401: Fermenter Temperature Transmitter (0–50°C, Setpoint: 25°C)
    pH401: pH Transmitter (0–14, Setpoint: 6.5)
    ST401: Agitator Speed Transmitter (0–300 RPM, Setpoint: 200 RPM)
    LT401: Fermenter Level Transmitter (0–100%, Setpoint: 80%)
    DO401: Dissolved Oxygen Transmitter (0–100%, Setpoint: 30%)
  Control Loops:
    TT401 --> TIC401 --> TCV401 (Cooling Jacket Valve, 0–100%)
      * TIC401 maintains 25°C ± 0.5°C via PID to optimize yeast activity
    pH401 --> AIC401 --> DCV401 (Acid/Base Dosing Valve, 0–100%)
      * AIC401 adjusts pH to 6.5 ± 0.2 using NaOH or H2SO4 via PID
    ST401 --> SIC401 --> MCV401 (Agitator Motor, 0–100%)
      * SIC401 maintains 200 RPM for uniform fermentation
    LT401 --> LIC401 --> FCV401 (Inlet Valve, 0–100%)
      * LIC401 maintains 80% level to prevent overflow
    DO401 --> DIC401 --> ACV401 (Aeration Valve, 0–100%)
      * DIC401 maintains 30% DO to support microbial growth

S-501: Separator
  Instruments:
    FT501: Product Flow Transmitter (0–400 L/h, Setpoint: 350 L/h)
    TT501: Separator Temperature Transmitter (0–50°C, Setpoint: 20°C)
    PT501: Separator Pressure Transmitter (0–5 bar, Setpoint: 1.0 bar)
  Control Loops:
    FT501 --> FIC501 --> FCV501 (Product Outlet Valve, 0–100%)
      * FIC501 controls flow to T-601 at 350 L/h
    TT501 --> TIC501 --> TCV501 (Cooling Valve, 0–100%)
      * TIC501 maintains 20°C to preserve penicillin stability
    PT501 --> PIC501 --> PCV501 (Pressure Control Valve, 0–100%)
      * PIC501 maintains 1.0 bar for safe operation

T-601: Product Tank
  Instruments:
    LT601: Level Transmitter (0–100%, Setpoint: 90%)
    TT601: Temperature Transmitter (0–50°C, Setpoint: 15°C)
    FT601: Outlet Flow Transmitter (0–400 L/h, Manual Setpoint)
  Control Loops:
    LT601 --> LIC601 --> FCV601 (Inlet Valve, 0–100%)
      * LIC601 maintains 90% level to prevent overflow
    TT601 --> TIC601 --> TCV601 (Cooling Valve, 0–100%)
      * TIC601 maintains 15°C to ensure product stability
    FT601 --> FIC601 (Manual) --> FCV602 (Outlet Valve, 0–100%)
      * FIC601 monitors flow for downstream processing, manually set

(* Control Philosophy *)
(* Temperature: Maintained at 25°C in F-401 (TIC401) and 15–20°C in S-501/T-601 (TIC501, TIC601) using PID loops to control cooling valves, ensuring optimal microbial activity and product stability. Alarms trigger at ±2°C deviations, with interlocks stopping feed if >30°C for 5 minutes. *)
(* pH: Controlled at 6.5 ± 0.2 in F-401 (AIC401) via PID dosing of acid/base, critical for Penicillium growth. Alarms at pH < 6.0 or > 7.0, with interlocks reducing dosing if pH < 5.5. *)
(* Agitation: Fixed at 200 RPM in F-401 (SIC401) and 150 RPM in M-201 (SIC201) using manual setpoints to ensure mixing without shear stress. Alarms trigger on motor failure, stopping feed. *)
(* Flow: Controlled at 400 L/h (FIC101, FIC301, FIC401) and 350 L/h (FIC501) via PID loops to maintain steady-state operation. Alarms at ±10% deviation, with interlocks stopping pumps if flow < 100 L/h. *)
(* Level: Maintained at 70–90% across tanks (LIC101, LIC201, LIC401, LIC601) using PID loops to prevent overflow or dry running. Low-level alarms (<20%) stop pumps. *)
(* Dissolved Oxygen: Maintained at 30% in F-401 (DIC401) via PID-controlled aeration to support aerobic fermentation. Alarms at <15%, with interlocks increasing aeration. *)

(* Notes *)
(* - Arrows (-->) indicate material flow (e.g., water, media, product) or signal paths *)
(* - ISA tags: XX-NNN (e.g., TT=Temperature Transmitter, 101=unique ID) *)
(* - Control loops use PID for dynamic regulation (e.g., TIC401) or manual setpoints (e.g., SIC201) *)
(* - Instrumentation includes transmitters (TT, FT, LT, pH, DO, PT, ST) and controllers (FIC, LIC, TIC, AIC, DIC, PIC, SIC) *)
(* - Diagram is structured by process unit with indented instrument blocks for clarity *)
(* - Supports PLC configuration, HMI design, and process validation *)
