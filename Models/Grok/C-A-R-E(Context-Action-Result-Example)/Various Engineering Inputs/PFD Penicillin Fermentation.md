# Penicillin Fermentation Process PFD

# Material Flow Path
Water Tank T01
  --> Media Preparation Tank T02
      --> Sterilizer S01
          --> Fermentation Tank F01
              --> Centrifuge C01
                  --> Filtration Unit FU01
                      --> Product Collection Tank T03
                          --> Waste Tank T04 (for biomass and filtrate)

# Unit Operations and Control Philosophy

[Water Tank T01]
  - Purpose: Store and supply purified water for media preparation.
  - Instrumentation:
    - LT101 (Level Transmitter, Range: 0–100%, Setpoint: 70%)
        --> LIC101 (Level Indicating Controller)
            --> LV101 (Level Control Valve, 0–100%): Maintains tank level by adjusting inlet water flow.
  - Control: PID loop ensures stable water supply; alarm if level < 20% or > 90%.

[Media Preparation Tank T02]
  - Purpose: Mix water, carbon source (e.g., glucose), nitrogen source (e.g., corn steep liquor), and minerals.
  - Instrumentation:
    - LT102 (Level Transmitter, Range: 0–100%, Setpoint: 60%)
        --> LIC102 (Level Indicating Controller)
            --> LV102 (Level Control Valve, 0–100%): Controls nutrient inlet flow.
    - FT103 (Flow Transmitter, Range: 0–50 L/min, Setpoint: 30 L/min)
        --> FIC103 (Flow Indicating Controller)
            --> FV103 (Flow Control Valve, 0–100%): Regulates glucose feed rate.
    - AT104 (pH Sensor, Range: 4.0–7.0, Setpoint: 6.5)
        --> AIC104 (pH Indicating Controller)
            --> AP104 (Acid/Base Dosing Pump, 0–0.5 L/min): Adjusts pH during mixing.
  - Control: PID loops for level, flow, and pH; agitator runs at fixed 200 RPM; alarm if pH deviates by ±0.5 or level > 80%.

[Sterilizer S01]
  - Purpose: Sterilize media at high temperature to eliminate contaminants.
  - Instrumentation:
    - TT105 (Temperature Transmitter, Range: 0–150°C, Setpoint: 121°C)
        --> TIC105 (Temperature Indicating Controller)
            --> TV105 (Steam Valve, 0–100%): Maintains sterilization temperature.
    - PT106 (Pressure Transmitter, Range: 0–5 bar, Setpoint: 2 bar)
        --> PIC106 (Pressure Indicating Controller)
            --> PV106 (Pressure Relief Valve, 0–100%): Ensures safe pressure.
  - Control: PID loop holds temperature at 121°C for 30 minutes; pressure control prevents overpressurization; alarm if temperature < 120°C or pressure > 2.5 bar; interlock stops steam if pressure > 3 bar.

[Fermentation Tank F01]
  - Purpose: Cultivate Penicillium mold to produce penicillin under controlled conditions.
  - Instrumentation:
    - TT201 (Temperature Transmitter, Range: 20–40°C, Setpoint: 25°C)
        --> TIC201 (Temperature Indicating Controller)
            --> CV201 (Cooling Water Valve, 0–100%): Maintains fermentation temperature via cooling jacket.
    - AT202 (pH Sensor, Range: 4.0–7.0, Setpoint: 5.0)
        --> AIC202 (pH Indicating Controller)
            --> AP202 (Base Dosing Pump, 0–0.5 L/min): Adjusts pH with sodium hydroxide.
    - LT203 (Level Transmitter, Range: 0–100%, Setpoint: 50%)
        --> LIC203 (Level Indicating Controller)
            --> LV203 (Level Control Valve, 0–100%): Regulates media inlet flow.
    - FT204 (Air Flow Transmitter, Range: 0–100 m³/h, Setpoint: 50 m³/h)
        --> FIC204 (Flow Indicating Controller)
            --> FV204 (Air Control Valve, 0–100%): Supplies sterile air for aeration.
    - ST205 (Agitator Speed Transmitter, Range: 0–500 RPM, Setpoint: 300 RPM)
        --> SIC205 (Speed Indicating Controller)
            --> SV205 (VFD for Agitator Motor, 0–100%): Controls agitation speed.
    - OT206 (Dissolved Oxygen Sensor, Range: 0–100%, Setpoint: 30%)
        --> OIC206 (Oxygen Indicating Controller)
            --> FV204 (Air Control Valve, Cascade): Adjusts air flow to maintain DO.
  - Control: 
    - Temperature: PID loop maintains 25°C using cooling water; alarm if > 27°C or < 23°C; interlock stops process if > 30°C.
    - pH: PID loop maintains 5.0 via base dosing; alarm if pH < 4.5 or > 5.5; interlock halts dosing if pH < 4.0 for 5 minutes.
    - Level: PID loop maintains 50% to prevent overflow; alarm if > 70%.
    - Aeration: Cascade PID loop adjusts air flow based on DO and flow feedback; alarm if DO < 20%.
    - Agitation: PID loop maintains 300 RPM; alarm if speed deviates by ±50 RPM; interlock stops motor if fault detected.
  - Note: Fermentation runs for 5–7 days; HMI allows operator override for pH and agitation setpoints.

[Centrifuge C01]
  - Purpose: Separate biomass (Penicillium mold) from penicillin-containing broth.
  - Instrumentation:
    - FT207 (Broth Flow Transmitter, Range: 0–20 L/min, Setpoint: 15 L/min)
        --> FIC207 (Flow Indicating Controller)
            --> FV207 (Broth Control Valve, 0–100%): Regulates broth feed to centrifuge.
    - ST208 (Centrifuge Speed Transmitter, Range: 0–5000 RPM, Setpoint: 4000 RPM)
        --> SIC208 (Speed Indicating Controller)
            --> SV208 (Centrifuge VFD, 0–100%): Controls centrifuge speed.
  - Control: PID loop maintains broth flow and centrifuge speed; alarm if speed < 3500 RPM or flow > 18 L/min; interlock stops centrifuge if vibration sensor (not shown) detects imbalance.

[Filtration Unit FU01]
  - Purpose: Filter broth to remove residual solids and clarify penicillin solution.
  - Instrumentation:
    - PT209 (Pressure Transmitter, Range: 0–10 bar, Setpoint: 2 bar)
        --> PIC209 (Pressure Indicating Controller)
            --> PV209 (Backpressure Valve, 0–100%): Maintains filtration pressure.
    - FT210 (Filtrate Flow Transmitter, Range: 0–15 L/min, Setpoint: 10 L/min)
  - Control: PID loop maintains filtration pressure; alarm if pressure > 3 bar or flow < 5 L/min; interlock stops filtration if pressure > 4 bar to prevent membrane damage.

[Product Collection Tank T03]
  - Purpose: Store clarified penicillin solution for downstream processing.
  - Instrumentation:
    - LT211 (Level Transmitter, Range: 0–100%, Setpoint: 80%)
        --> LIC211 (Level Indicating Controller)
            --> LV211 (Outlet Control Valve, 0–100%): Controls transfer to next stage.
  - Control: PID loop maintains tank level; alarm if level > 90% or < 10%; interlock stops inflow if level > 95%.

[Waste Tank T04]
  - Purpose: Collect biomass and filtrate waste.
  - Instrumentation:
    - LT212 (Level Transmitter, Range: 0–100%, Setpoint: 50%)
  - Control: Manual monitoring; alarm if level > 90% to prompt waste disposal.

# Safety and Alarms
  - HS213 (High Fermentation Temperature Alarm, Digital: 0 or 1, Trigger: TT201 > 27°C)
  - LS214 (Low Dissolved Oxygen Alarm, Digital: 0 or 1, Trigger: OT206 < 20%)
  - HS215 (High Tank Level Alarm, Digital: 0 or 1, Trigger: LT203 > 70%)
  - ES216 (Emergency Shutdown, Digital: 0 or 1, Trigger: TT201 > 30°C or AT202 < 4.0 for 5 min)
