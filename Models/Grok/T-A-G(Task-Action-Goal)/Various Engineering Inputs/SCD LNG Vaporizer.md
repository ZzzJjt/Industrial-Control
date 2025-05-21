# SCD: LNG Vaporizer System - 100,000 kg/h
# Notation: --> (process flow), | (instrumentation), [] (control loop), () (equipment tag)
# Tag Format: NORSOK I-005 (e.g., 21-FT-201 = Flow Transmitter 201, System 21 = LNG Vaporizer)
# System: 21 (LNG Vaporizer), Location: Offshore LNG Terminal

# 1. Process Flow
  (TK-101: LNG Storage Tank, 200,000 L, -162°C, 5 bar)
    | 21-LT-101 (Level Transmitter, 0–100%, Setpoint: 70%)
    | 21-LIC-101 [Controls 21-LCV-101]
    --> 21-LCV-101 (Level Control Valve, 0–100%, Setpoint: 50%) --> 
    | 21-FT-201 (LNG Inlet Flow, 0–120,000 kg/h, Setpoint: 100,000 kg/h)
    | 21-FIC-201 [Controls 21-FCV-201]
    --> 21-FCV-201 (LNG Flow Control Valve, 0–100%, Setpoint: 80%) --> 
    | 21-TT-101 (LNG Inlet Temperature, -170 to -150°C, Setpoint: -162°C)
    | 21-PT-102 (LNG Inlet Pressure, 0–10 bar, Setpoint: 5 bar)
  (VP-101: Shell-and-Tube Vaporizer, Steam-Heated)
    | 21-TT-201 (NG Outlet Temperature, 0–50°C, Setpoint: 10°C)
    | 21-TIC-201 [Controls 21-TCV-201]
    | 21-PT-202 (NG Outlet Pressure, 0–10 bar, Setpoint: 4.5 bar)
    | 21-PIC-202 [Controls 21-PCV-202]
    --> 21-TCV-201 (Steam Control Valve, 0–100%, Setpoint: 60%) --> 
    --> 21-PCV-202 (Pressure Control Valve, 0–100%, Setpoint: 50%) --> 
    | 21-FT-203 (NG Outlet Flow, 0–120,000 kg/h, Setpoint: 100,000 kg/h)
  (TK-102: Feedwater Tank, 10,000 L, 150°C, 6 bar)
    | 21-LT-301 (Level Transmitter, 0–100%, Setpoint: 60%)
    | 21-LIC-301 [Controls 21-LCV-301]
    --> 21-LCV-301 (Feedwater Control Valve, 0–100%, Setpoint: 50%) --> 
    | 21-FT-302 (Feedwater Flow, 0–5000 L/min, Setpoint: 4000 L/min)
    | 21-FIC-302 [Controls 21-FCV-302]
    --> 21-FCV-302 (Feedwater Flow Valve, 0–100%, Setpoint: 70%) --> 
    | 21-TT-303 (Feedwater Temperature, 0–200°C, Setpoint: 150°C)
  (P-101: Feedwater Pump)
    | 21-XI-304 (Pump Run Status, 0/1, Setpoint: 1)
    | 21-YO-304 [Pump Start Command, 0/1]
    | 21-PI-305 (Discharge Pressure, 0–10 bar, Setpoint: 6 bar)
    --> (VP-101: Shell-and-Tube Vaporizer) [Feedwater recycle loop]
  (TK-103: NG Outlet Buffer Tank, 50,000 L, 10°C, 4.5 bar)
    | 21-LT-401 (Level Transmitter, 0–100%, Setpoint: 50%)
    | 21-PT-402 (Pressure Transmitter, 0–10 bar, Setpoint: 4.5 bar)
    | 21-TT-403 (Temperature Transmitter, 0–50°C, Setpoint: 10°C)
    --> Pipeline Distribution

# 2. Control Loops
  - Flow Control Loop:
    | 21-FT-201 (LNG Inlet Flow) --> 21-FIC-201 (PID Controller)
    | 21-FIC-201 [Outputs to 21-FCV-201]
    | Setpoint: 100,000 kg/h, Range: 0–120,000 kg/h
    | Control: Adjust 21-FCV-201 (0–100%) to maintain LNG flow
  - Temperature Control Loop:
    | 21-TT-201 (NG Outlet Temperature) --> 21-TIC-201 (PID Controller)
    | 21-TIC-201 [Outputs to 21-TCV-201]
    | Setpoint: 10°C, Range: 0–50°C
    | Control: Adjust 21-TCV-201 (0–100%) to control steam flow for heating
  - Pressure Control Loop:
    | 21-PT-202 (NG Outlet Pressure) --> 21-PIC-202 (PID Controller)
    | 21-PIC-202 [Outputs to 21-PCV-202]
    | Setpoint: 4.5 bar, Range: 0–10 bar
    | Control: Adjust 21-PCV-202 (0–100%) to maintain outlet pressure
  - Feedwater Level Control Loop:
    | 21-LT-301 (Feedwater Tank Level) --> 21-LIC-301 (PID Controller)
    | 21-LIC-301 [Outputs to 21-LCV-301]
    | Setpoint: 60%, Range: 0–100%
    | Control: Adjust 21-LCV-301 (0–100%) to maintain tank level
  - Feedwater Flow Control Loop:
    | 21-FT-302 (Feedwater Flow) --> 21-FIC-302 (PID Controller)
    | 21-FIC-302 [Outputs to 21-FCV-302]
    | Setpoint: 4000 L/min, Range: 0–5000 L/min
    | Control: Adjust 21-FCV-302 (0–100%) to maintain feedwater flow

# 3. Safety Logic and Interlocks
  - Overpressure Interlock:
    | 21-PSH-301 (High Pressure Switch, Trip: > 5.2 bar)
    | [Triggers ESD1: Shut Heater Steam]
    | Action: Close 21-TCV-201 (0%), Open 21-PCV-202 (100%)
    | Alarm: "High Pressure Trip – ESD1 Activated"
  - Overtemperature Interlock:
    | 21-TSH-302 (High Temperature Switch, Trip: > 20°C)
    | [Triggers ESD2: Block LNG Inlet]
    | Action: Close 21-FCV-201 (0%), Close 21-LCV-101 (0%)
    | Alarm: "High Temperature Trip – ESD2 Activated"
  - Low Feedwater Level Interlock:
    | 21-LSL-303 (Low Level Switch, Trip: < 30%)
    | [Stops P-101]
    | Action: Set 21-YO-304 = 0 (Pump Stop)
    | Alarm: "Low Feedwater Level – Pump Stopped"
  - Pump Failure Interlock:
    | 21-XI-304 (Pump Run Status = 0 when YO-304 = 1)
    | [Triggers Alarm]
    | Action: Close 21-FCV-302 (0%)
    | Alarm: "Feedwater Pump Failure"
  - Emergency Shutdown:
    | 21-XI-401 (Emergency Stop Button, Trip: ON)
    | [Triggers Full ESD]
    | Action: Close 21-FCV-201, 21-TCV-201, 21-PCV-202, 21-LCV-301, 21-FCV-302 (0%), Stop P-101 (YO-304 = 0)
    | Alarm: "Emergency Shutdown Activated"

# Notes
- Flow direction: --> indicates process flow (LNG → NG → pipeline).
- Tags: System 21 (LNG Vaporizer), e.g., 21-FT-201 (Flow), 21-TIC-201 (Temperature Controller).
- Control: [] denotes controller output to valve/pump (e.g., TIC201 [Controls TCV201]).
- Setpoints: Reflect steady-state at 100,000 kg/h (e.g., 10°C NG, 4.5 bar, 4000 L/min feedwater).
- Safety: Interlocks (PSH-301, TSH-302) ensure protection against overpressure/overtemperature.
- Compliance: NORSOK I-005 for tag format, control philosophy, and safety logic.
