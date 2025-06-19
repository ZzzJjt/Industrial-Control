# Steam-Water Cycle P&ID

# Feedwater Supply
Feedwater Tank T01
  --> LT101 (Level Transmitter, Range: 0–100%, Setpoint: 60%)
      --> LIC101 (Level Indicating Controller)
          --> LV101 (Level Control Valve, 0–100%)
  --> P01 (Feedwater Pump)
      --> PS101 (Pump Status, Digital: 0 or 1)
      --> SC101 (Speed Controller, Analog: 0–3600 RPM, Setpoint: 3000 RPM)
  --> FT102 (Flow Transmitter, Range: 0–120 t/h, Setpoint: 80 t/h)
      --> FIC102 (Flow Indicating Controller)
          --> FCV102 (Feedwater Control Valve, 0–100%)
  --> PT103 (Pressure Transmitter, Range: 0–200 bar, Setpoint: 150 bar)
  --> TT104 (Temperature Transmitter, Range: 0–200°C, Setpoint: 120°C)

# Boiler and Steam Generation
  --> Boiler B01
      --> TT105 (Boiler Temperature Transmitter, Range: 0–400°C, Setpoint: 350°C)
          --> TIC105 (Temperature Indicating Controller)
              --> TV105 (Steam Valve, 0–100%)
      --> PT106 (Boiler Pressure Transmitter, Range: 0–250 bar, Setpoint: 180 bar)
      --> B01 --> Steam Drum D01
          --> LT107 (Drum Level Transmitter, Range: 0–100%, Setpoint: 50%)
              --> LIC107 (Drum Level Indicating Controller)
                  --> LV107 (Drum Level Control Valve, 0–100%)
          --> PT108 (Drum Pressure Transmitter, Range: 0–250 bar, Setpoint: 180 bar)
              --> PIC108 (Pressure Indicating Controller)
                  --> PV108 (Pressure Control Valve, 0–100%)

# Steam Distribution and Turbine
  --> Steam Turbine ST01
      --> FT109 (Steam Flow Transmitter, Range: 0–150 t/h, Setpoint: 100 t/h)
      --> ST01 --> Condenser C01
          --> TT110 (Condenser Temperature Transmitter, Range: 0–100°C, Setpoint: 40°C)
              --> TIC110 (Temperature Indicating Controller)
                  --> CV110 (Cooling Water Valve, 0–100%)
          --> LT111 (Condensate Level Transmitter, Range: 0–100%, Setpoint: 30%)
              --> LIC111 (Condensate Level Indicating Controller)
                  --> LV111 (Condensate Level Control Valve, 0–100%)

# Condensate Recovery
  --> Condensate Pump P02
      --> PS112 (Pump Status, Digital: 0 or 1)
      --> SC112 (Speed Controller, Analog: 0–3600 RPM, Setpoint: 2800 RPM)
  --> FT113 (Condensate Flow Transmitter, Range: 0–100 t/h, Setpoint: 70 t/h)
  --> PT114 (Condensate Pressure Transmitter, Range: 0–50 bar, Setpoint: 10 bar)
  --> TT115 (Condensate Temperature Transmitter, Range: 0–100°C, Setpoint: 40°C)
  --> Feedwater Tank T01 (Return Loop)

# Safety and Alarms
  --> HS116 (High Drum Level Alarm, Digital: 0 or 1, Trigger: LT107 > 70%)
  --> LS117 (Low Drum Level Alarm, Digital: 0 or 1, Trigger: LT107 < 30%)
  --> HS118 (High Boiler Pressure Alarm, Digital: 0 or 1, Trigger: PT106 > 200 bar)
  --> ES119 (Emergency Shutdown, Digital: 0 or 1, Trigger: LT107 < 20% or PT106 > 210 bar)
