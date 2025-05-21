# P&ID: Steam-Water Cycle - 500 MW Power Plant
# Notation: --> (flow direction), | (connection to instrument), [] (control loop), () (equipment tag)
# Tag Format: ISA-5.1 (e.g., FT101 = Flow Transmitter 101, FCV102 = Flow Control Valve 102)

# 1. Feedwater System
  (TK-101: Deaerator Tank)
    | LT101 (Level Transmitter, 0–100%, Setpoint: 60%)
    | LIC101 [Controls LCV101]
    --> LCV101 (Level Control Valve, 0–100%, Setpoint: 50%) --> 
  (P-101A/B: Feedwater Pumps, 2x100% capacity)
    | XI101A (Pump A Run Status, 0/1)
    | XI101B (Pump B Run Status, 0/1)
    | YO101A [Pump A Start Command, 0/1]
    | YO101B [Pump B Start Command, 0/1]
    | PI102 (Discharge Pressure, 0–150 bar, Setpoint: 110 bar)
    --> FT103 (Feedwater Flow, 0–200 kg/s, Setpoint: 150 kg/s)
    --> FIC103 [Controls FCV103]
    --> FCV103 (Feedwater Control Valve, 0–100%, Setpoint: 75%) --> 
    | TI104 (Feedwater Temperature, 0–350°C, Setpoint: 300°C)

# 2. Economizer
  (EC-101: Economizer)
    --> FT105 (Inlet Flow, 0–200 kg/s, Setpoint: 150 kg/s)
    --> TI106 (Outlet Temperature, 0–400°C, Setpoint: 320°C) --> 

# 3. Boiler and Steam Drum
  (BL-101: Boiler)
    (SD-101: Steam Drum)
      | LT107 (Drum Level Transmitter 1, 0–100%, Setpoint: 50%)
      | LT108 (Drum Level Transmitter 2, 0–100%, Setpoint: 50%)
      | LIC107 [Controls FCV103 via FIC103, 3-element control]
      | LS109 (Low Level Switch, 0/1, Trip: <30%)
      | LS110 (High Level Switch, 0/1, Trip: >70%)
      | PT111 (Drum Pressure, 0–120 bar, Setpoint: 100 bar)
      | TIC112 (Drum Temperature, 0–600°C, Setpoint: 540°C)
      --> FT113 (Steam Flow, 0–200 kg/s, Setpoint: 150 kg/s)
      --> FIC113 [Controls FCV114]
      --> FCV114 (Steam Control Valve, 0–100%, Setpoint: 80%) --> 

# 4. Turbine
  (TB-101: Steam Turbine)
    | SI114 (Turbine Speed, 0–3600 RPM, Setpoint: 3000 RPM)
    | XI115 (Turbine Trip Status, 0/1, Normal: 0)
    --> PT116 (Exhaust Pressure, 0–10 bar, Setpoint: 0.1 bar) --> 

# 5. Condenser
  (CD-101: Condenser)
    | TI117 (Condensate Temperature, 0–100°C, Setpoint: 40°C)
    | FT118 (Cooling Water Flow, 0–1000 L/min, Setpoint: 800 L/min)
    | FIC118 [Controls FCV119]
    --> FCV119 (Cooling Water Valve, 0–100%, Setpoint: 75%) --> 
    | LT120 (Condensate Level, 0–100%, Setpoint: 50%)
    | LIC120 [Controls LCV121]
    --> LCV121 (Condensate Control Valve, 0–100%, Setpoint: 50%) --> 

# 6. Condensate Return
  (P-102: Condensate Pump)
    | XI122 (Pump Run Status, 0/1)
    | YO122 [Pump Start Command, 0/1]
    | PI123 (Discharge Pressure, 0–10 bar, Setpoint: 5 bar)
    --> FT124 (Condensate Flow, 0–200 kg/s, Setpoint: 150 kg/s) --> 
    (TK-101: Deaerator Tank) [Loop back to start]

# Control Loops Summary
- LIC107/FIC103: 3-element drum level control (LT107/108, FT103, FT113) → FCV103
- FIC113: Steam flow control (FT113) → FCV114
- LIC101: Deaerator level control (LT101) → LCV101
- FIC118: Cooling water flow control (FT118) → FCV119
- LIC120: Condensate level control (LT120) → LCV121

# Notes
- Flow direction: --> indicates process flow (feedwater → steam → condensate).
- Instrumentation: Tag numbers (e.g., FT101, LIC107) follow ISA-5.1.
- Control: [] denotes controller output to valve/pump (e.g., LIC107 [Controls FCV103]).
- Setpoints: Reflect steady-state at 500 MW (e.g., 50% drum level, 150 kg/s flow).
- Safety: LS109/110 trigger trips for low/high drum level (<30%, >70%).
