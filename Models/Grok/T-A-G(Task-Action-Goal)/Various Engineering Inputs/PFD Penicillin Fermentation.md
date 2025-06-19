# PFD: Penicillin Fermentation Process - 50,000 L Batch
# Notation: --> (material flow), | (instrumentation), [] (control loop), () (equipment tag)
# Tag Format: ISA-5.1 (e.g., TT101 = Temperature Transmitter 101, FCV102 = Flow Control Valve 102)

# 1. Water Tank
  (TK-101: Water Tank, 60,000 L)
    | LT101 (Level Transmitter, 0–100%, Setpoint: 80%)
    | LIC101 [Controls LCV101]
    | TI102 (Temperature Transmitter, 0–50°C, Setpoint: 20°C)
    --> LCV101 (Level Control Valve, 0–100%, Setpoint: 50%) --> 
    --> FT103 (Water Flow, 0–1000 L/min, Setpoint: 800 L/min)
    --> FIC103 [Controls FCV103]
    --> FCV103 (Water Control Valve, 0–100%, Setpoint: 75%) --> 

# 2. Nutrient Mixer
  (MX-101: Nutrient Mixer, 55,000 L)
    | LT104 (Level Transmitter, 0–100%, Setpoint: 90%)
    | LIC104 [Controls FCV103, Nutrient Pumps P-102/103]
    | TI105 (Temperature Transmitter, 0–50°C, Setpoint: 25°C)
    | SIC106 (Agitator Speed, 0–100 RPM, Setpoint: 50 RPM)
    | AIC107 (pH Transmitter, 4.0–7.0, Setpoint: 6.5)
    | [AIC107 Controls Nutrient Pump P-102 (Glucose), P-103 (Nitrogen Source)]
    --> P-101 (Transfer Pump)
    --> FT108 (Nutrient Flow, 0–500 L/min, Setpoint: 400 L/min) --> 

# 3. Sterilizer
  (ST-101: Continuous Steam Sterilizer)
    | TI109 (Sterilization Temperature, 0–150°C, Setpoint: 121°C)
    | TIC109 [Controls Steam Valve SV-101]
    | PT110 (Pressure Transmitter, 0–5 bar, Setpoint: 2.1 bar)
    | FT111 (Sterilized Medium Flow, 0–500 L/min, Setpoint: 400 L/min)
    --> SV-101 (Steam Control Valve, 0–100%, Setpoint: 60%) --> 
    --> FCV112 (Medium Control Valve, 0–100%, Setpoint: 70%) --> 

# 4. Fermenter
  (FR-101: Fermenter, 50,000 L)
    | LT201 (Level Transmitter, 0–100%, Setpoint: 90%)
    | LIC201 [Controls FCV112]
    | TT201 (Temperature Transmitter, 0–50°C, Setpoint: 30°C)
    | TIC201 [Controls Cooling Valve TCV201]
    | TCV201 (Cooling Water Valve, 0–100%, Setpoint: 50%) --> 
    | pH202 (pH Transmitter, 4.0–7.0, Setpoint: 4.8)
    | PIC202 [Controls Base Pump P-201, Acid Pump P-202]
    | P-201 (Base Pump, NaOH, 0–5 L/min, Setpoint: 2 L/min)
    | P-202 (Acid Pump, H2SO4, 0–5 L/min, Setpoint: 1 L/min)
    | FT203 (Air Flow, 0–5000 L/min, Setpoint: 4000 L/min)
    | FIC203 [Controls Air Valve FCV203]
    | FCV203 (Air Control Valve, 0–100%, Setpoint: 80%) --> 
    | SIC204 (Agitator Speed, 0–100 RPM, Setpoint: 60 RPM)
    | LIC205 (Foam Level Transmitter, 0–100%, Setpoint: 85%)
    | [LIC205 Controls Antifoam Pump P-203]
    | P-203 (Antifoam Pump, 0–1 L/min, Setpoint: 0.2 L/min)
    | PT206 (Pressure Transmitter, 0–2 bar, Setpoint: 1.2 bar)
    | PIC206 [Controls Vent Valve PV-201]
    | PV-201 (Vent Valve, 0–100%, Setpoint: 50%) --> 
    | AI207 (Penicillin Concentration, 0–5 g/L, Setpoint: 2 g/L)
    --> P-204 (Broth Transfer Pump)
    --> FT208 (Broth Flow, 0–500 L/min, Setpoint: 300 L/min) --> 

# 5. Separator
  (SP-101: Centrifugal Separator)
    | FT301 (Broth Inlet Flow, 0–500 L/min, Setpoint: 300 L/min)
    | FIC301 [Controls FCV301]
    | FCV301 (Broth Control Valve, 0–100%, Setpoint: 65%) --> 
    | SIC302 (Centrifuge Speed, 0–5000 RPM, Setpoint: 4000 RPM)
    | AI303 (Biomass Concentration, 0–100 g/L, Setpoint: 50 g/L)
    --> FT304 (Penicillin Flow, 0–200 L/min, Setpoint: 150 L/min) --> 

# 6. Product Tank
  (TK-102: Product Tank, 20,000 L)
    | LT401 (Level Transmitter, 0–100%, Setpoint: 80%)
    | LIC401 [Controls FCV301]
    | TI402 (Temperature Transmitter, 0–50°C, Setpoint: 20°C)
    | TIC402 [Controls Cooling Valve TCV402]
    | TCV402 (Cooling Water Valve, 0–100%, Setpoint: 40%) --> 
    | AI403 (Penicillin Concentration, 0–5 g/L, Setpoint: 2 g/L)

# Control Loops Summary
- LIC101: Deaerator level control (LT101) → LCV101
- FIC103: Water flow control (FT103) → FCV103
- LIC104: Mixer level control (LT104) → FCV103, P-102/103
- AIC107: Mixer pH control (AIC107) → P-102/103
- TIC109: Sterilization temperature control (TI109) → SV-101
- LIC201: Fermenter level control (LT201) → FCV112
- TIC201: Fermenter temperature control (TT201) → TCV201
- PIC202: Fermenter pH control (pH202) → P-201/202
- FIC203: Fermenter air flow control (FT203) → FCV203
- LIC205: Fermenter foam control (LIC205) → P-203
- PIC206: Fermenter pressure control (PT206) → PV-201
- FIC301: Separator broth flow control (FT301) → FCV301
- LIC401: Product tank level control (LT401) → FCV301
- TIC402: Product tank temperature control (TI402) → TCV402

# Notes
- Flow direction: --> indicates material flow (water → medium → broth → penicillin).
- Instrumentation: Tag numbers (e.g., TT201, FIC301) follow ISA-5.1.
- Control: [] denotes controller output to valve/pump (e.g., TIC201 [Controls TCV201]).
- Setpoints: Reflect optimal conditions for penicillin production (e.g., 30°C, pH 4.8, 2 g/L yield).
- Safety: Foam (LIC205 > 95%), pressure (PT206 > 1.5 bar), and level (LT201 < 70%) interlocks implied.
