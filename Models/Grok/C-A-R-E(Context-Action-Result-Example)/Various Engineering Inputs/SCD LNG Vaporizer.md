# System Control Diagram (SCD) for LNG Vaporizer System
# NORSOK I-005 Compliant

# Main Flow Path
LNG Inlet
  --> FT101 (Flow Transmitter, Range: 0–1000 kg/h, Setpoint: 800 kg/h)
      --> FIC101 (Flow Indicating Controller)
          --> FCV101 (Flow Control Valve, 0–100%)
  --> Vaporizer V01
      --> Gas Outlet

# Equipment and Instrumentation

[LNG Inlet]
  - Purpose: Supply LNG to vaporizer at controlled flow and pressure.
  - Instrumentation:
    - FT101 (Flow Transmitter, Range: 0–1000 kg/h, Setpoint: 800 kg/h)
        --> FIC101 (Flow Indicating Controller)
            --> FCV101 (Flow Control Valve, 0–100%): Maintains LNG inlet flow.
    - PT102 (Pressure Transmitter, Range: 0–50 bar, Setpoint: 40 bar)
        --> PIC102 (Pressure Indicating Controller)
            --> PCV102 (Pressure Control Valve, 0–100%): Regulates inlet pressure.
    - TT103 (Temperature Transmitter, Range: -200–0°C, Setpoint: -160°C)
        --> TIC103 (Temperature Indicating Controller)
            --> TCV103 (Cryogenic Valve, 0–100%): Adjusts LNG temperature via bypass.
  - Control Philosophy:
    - PID loop (FIC101) maintains flow at 800 kg/h to ensure stable vaporizer operation.
    - PID loop (PIC102) maintains pressure at 40 bar to prevent overpressurization.
    - PID loop (TIC103) adjusts cryogenic valve to stabilize LNG temperature.
  - Alarms:
    - FAL101 (Low Flow Alarm, Trigger: FT101 < 100 kg/h)
    - FAH102 (High Flow Alarm, Trigger: FT101 > 950 kg/h)
    - PAL103 (Low Pressure Alarm, Trigger: PT102 < 35 bar)
    - TAH104 (High Temperature Alarm, Trigger: TT103 > -150°C)

[Vaporizer V01]
  - Purpose: Convert LNG to gaseous natural gas using a shell-and-tube heat exchanger with steam heating.
  - Instrumentation:
    - TT201 (Vaporizer Outlet Temperature Transmitter, Range: 0–100°C, Setpoint: 20°C)
        --> TIC201 (Temperature Indicating Controller)
            --> TCV201 (Steam Valve, 0–100%): Controls steam flow to maintain gas outlet temperature.
    - PT202 (Vaporizer Outlet Pressure Transmitter, Range: 0–60 bar, Setpoint: 45 bar)
        --> PIC202 (Pressure Indicating Controller)
            --> PCV202 (Outlet Pressure Control Valve, 0–100%): Regulates gas outlet pressure.
    - FT203 (Gas Flow Transmitter, Range: 0–1200 kg/h, Setpoint: 800 kg/h)
        --> FIC203 (Flow Indicating Controller, Cascade)
            --> FCV101 (Inlet Flow Control Valve): Adjusts LNG inlet flow to match outlet demand.
    - LT204 (Vaporizer Liquid Level Transmitter, Range: 0–100%, Setpoint: 50%)
        --> LIC204 (Level Indicating Controller)
            --> LCV204 (Level Control Valve, 0–100%): Maintains liquid level in vaporizer.
  - Control Philosophy:
    - PID loop (TIC201) maintains outlet gas temperature at 20°C by adjusting steam flow.
    - PID loop (PIC202) regulates outlet pressure at 45 bar to ensure safe downstream delivery.
    - Cascade PID loop (FIC203) adjusts inlet flow (FCV101) based on outlet gas flow to balance throughput.
    - PID loop (LIC204) maintains vaporizer liquid level at 50% to prevent dry-out or flooding.
  - Alarms:
    - TAH201 (High Temperature Alarm, Trigger: TT201 > 30°C)
    - TAL202 (Low Temperature Alarm, Trigger: TT201 < 10°C)
    - PAH203 (High Pressure Alarm, Trigger: PT202 > 50 bar)
    - LAL204 (Low Level Alarm, Trigger: LT204 < 30%)
    - LAH205 (High Level Alarm, Trigger: LT204 > 70%)

[Steam Supply]
  - Purpose: Provide heating medium to vaporizer.
  - Instrumentation:
    - FT301 (Steam Flow Transmitter, Range: 0–500 kg/h, Setpoint: 300 kg/h)
        --> FIC301 (Flow Indicating Controller)
            --> FCV301 (Steam Flow Control Valve, 0–100%): Regulates steam supply to vaporizer.
    - TT302 (Steam Temperature Transmitter, Range: 100–250°C, Setpoint: 180°C)
        --> TIC302 (Temperature Indicating Controller)
            --> TCV302 (Steam Temperature Control Valve, 0–100%): Adjusts steam temperature.
  - Control Philosophy:
    - PID loop (FIC301) maintains steam flow to support vaporizer heating demand.
    - PID loop (TIC302) ensures steam temperature stability at 180°C.
  - Alarms:
    - FAL301 (Low Steam Flow Alarm, Trigger: FT301 < 100 kg/h)
    - TAH302 (High Steam Temperature Alarm, Trigger: TT302 > 200°C)

# Safety Interlocks and Trip Signals
  - PSH401 (High Pressure Shutdown, Trigger: PT202 > 55 bar)
      --> ESD1 (Emergency Shutdown): Closes FCV101, PCV202, and TCV201; opens vent valve XV401.
  - TSH402 (High Temperature Shutdown, Trigger: TT201 > 35°C)
      --> ESD2 (Emergency Shutdown): Closes TCV201 and FCV301; stops steam supply.
  - LSL403 (Low Level Trip, Trigger: LT204 < 20%)
      --> ESD3 (Emergency Shutdown): Closes FCV101 to prevent vaporizer dry-out.
  - FSL404 (Low Flow Trip, Trigger: FT101 < 50 kg/h for 30 seconds)
      --> A101 (Alarm) --> Auto Close FCV101: Prevents insufficient LNG supply.
  - PSH405 (High Inlet Pressure Shutdown, Trigger: PT102 > 48 bar)
      --> ESD4 (Emergency Shutdown): Closes FCV101 and PCV102; opens bypass valve XV402.
  - MS406 (Manual Emergency Stop, Trigger: Operator Input via HMI)
      --> ESD5 (Full System Shutdown): Closes all valves (FCV101, PCV102, TCV201, FCV301); opens vent XV401.

# Monitoring and Operator Interface
  - Data Logging:
    - Log all transmitters (FT101, PT102, TT103, TT201, PT202, FT203, LT204, FT301, TT302) every 10 seconds.
    - Record alarm and interlock events with timestamps.
  - HMI Displays:
    - Real-time values for flow (FT101, FT203), pressure (PT102, PT202), temperature (TT103, TT201), and level (LT204).
    - Trend graphs for outlet temperature (TT201) and pressure (PT202) over 1 hour.
    - Status indicators for control loops (FIC101, TIC201, etc.) and ESD states.
  - Operator Inputs:
    - Adjust setpoints (e.g., TIC201, FIC101) within safe limits via HMI.
    - Acknowledge alarms (e.g., FAL101, TAH201).
    - Initiate manual shutdown (MS406) or reset ESD conditions.
