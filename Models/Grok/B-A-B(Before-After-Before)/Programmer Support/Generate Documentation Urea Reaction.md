UreaReactionControl Program Documentation
Purpose
The UreaReactionControl program is an IEC 61131-3 Structured Text PLC program designed to automate a two-step urea synthesis process in a chemical reactor. It controls the injection of raw materials (ammonia and CO₂), monitors reactor conditions (pressure and temperature), and manages reaction timing to ensure safe, efficient, and high-quality urea production. The program ensures precise valve control, adheres to safety limits, and uses real-time clock data (CURRENT_TIME) for accurate reaction timing.
Program Overview
The urea synthesis process involves:

Loading Raw Materials: Opening valves to inject ammonia and CO₂ into the reactor.
Monitoring and Controlling the Reaction: Maintaining pressure and temperature within specified tolerances while timing the reaction duration.
Completion: Closing valves and finalizing the reaction once conditions are met.

The program uses a state machine to manage these steps, with safety checks to prevent over-pressurization, overheating, or incomplete reactions.
Variable Definitions
Inputs



Variable
Type
Description



Pressure_PV
REAL
Measured reactor pressure in bar (from pressure sensor).


Temp_PV
REAL
Measured reactor temperature in °C (from temperature sensor).


Start_Button
BOOL
TRUE when operator initiates the reaction cycle.


Outputs



Variable
Type
Description



Ammonia_Valve
BOOL
TRUE to open ammonia injection valve, FALSE to close.


CO2_Valve
BOOL
TRUE to open CO₂ injection valve, FALSE to close.


Reaction_Complete
BOOL
TRUE when reaction cycle is successfully completed.


Internal Variables



Variable
Type
Description



Reaction_State
INT
State machine variable: 0=Idle, 1=Loading, 2=Reaction, 3=Complete.


Reaction_Timer
TON
Timer for tracking reaction duration.


Start_Time
TIME
Stores reaction start time using CURRENT_TIME.


Elapsed_Time
TIME
Calculated time since reaction start.


Pressure_Fault
BOOL
TRUE if pressure is outside safe range.


Temp_Fault
BOOL
TRUE if temperature is outside safe range.


Constants



Parameter
Type
Value
Description



Pressure_SP
REAL
150.0
Target pressure setpoint in bar.


Pressure_Tolerance
REAL
5.0
Allowable pressure deviation in bar.


Temp_SP
REAL
180.0
Target temperature setpoint in °C.


Temp_Tolerance
REAL
10.0
Allowable temperature deviation in °C.


Reaction_Duration
TIME
T#300s
Required reaction time (5 minutes).


Max_Pressure
REAL
160.0
Maximum safe pressure in bar.


Min_Pressure
REAL
140.0
Minimum safe pressure in bar.


Max_Temp
REAL
200.0
Maximum safe temperature in °C.


Min_Temp
REAL
160.0
Minimum safe temperature in °C.


Process Flow Description
Step 1: Loading Raw Materials

Trigger: Operator presses Start_Button, and Reaction_State is 0 (Idle).
Actions:
Transition to Reaction_State := 1 (Loading).
Open Ammonia_Valve and CO2_Valve to inject raw materials.
Capture Start_Time using CURRENT_TIME.


Conditions:
Valves remain open until Pressure_PV reaches Pressure_SP ± Pressure_Tolerance (150.0 ± 5.0 bar).
Safety check: If Pressure_PV > Max_Pressure or Temp_PV > Max_Temp, close both valves and set Pressure_Fault or Temp_Fault.


Transition: Once pressure is within tolerance, move to Reaction_State := 2 (Reaction).

Step 2: Monitoring and Controlling the Reaction

Actions:
Close Ammonia_Valve and CO2_Valve to stop material injection.
Start Reaction_Timer (IN := TRUE, PT := Reaction_Duration).
Continuously monitor Pressure_PV and Temp_PV against Pressure_SP ± Pressure_Tolerance and Temp_SP ± Temp_Tolerance.
Calculate Elapsed_Time := CURRENT_TIME - Start_Time for reaction timing.


Conditions:
If Pressure_PV or Temp_PV exceeds Max_Pressure or Max_Temp, set Pressure_Fault or Temp_Fault, close valves, and halt the reaction.
If Pressure_PV or Temp_PV falls below Min_Pressure or Min_Temp, flag a fault but allow the timer to continue unless critical.


Transition: When Reaction_Timer.Q is TRUE (after 5 minutes) and conditions are within tolerances, move to Reaction_State := 3 (Complete).

Step 3: Completion

Actions:
Set Reaction_Complete := TRUE to signal successful reaction.
Ensure Ammonia_Valve and CO2_Valve are closed.
Reset Reaction_Timer and clear PedestrianRequest.
Return to Reaction_State := 0 (Idle) for the next cycle.


Conditions:
If faults (Pressure_Fault or Temp_Fault) are active, Reaction_Complete remains FALSE, and the system stays in Idle until reset by the operator.


Safety: Valves remain closed to prevent further material injection.

Valve Control Logic

Ammonia_Valve and CO2_Valve:
Open (TRUE) only in Reaction_State = 1 (Loading) when Pressure_PV < Pressure_SP + Pressure_Tolerance and no faults.
Closed (FALSE) in all other states or if Pressure_Fault, Temp_Fault, or Pressure_PV > Max_Pressure.


Safety Shutoff:
Immediate closure if Pressure_PV > 160.0 bar or Temp_PV > 200.0°C.
Valves remain closed until the system is reset and faults are cleared.



Timing and Tolerance Logic

Reaction Timing:
Start_Time is captured using CURRENT_TIME when entering Reaction_State = 1.
Elapsed_Time is computed as CURRENT_TIME - Start_Time to track total reaction duration.
Reaction_Timer enforces a fixed 5-minute (T#300s) reaction period in Reaction_State = 2.


Pressure Tolerance:
Target: Pressure_SP = 150.0 bar.
Acceptable range: 145.0–155.0 bar (Pressure_SP ± Pressure_Tolerance).
Fault if Pressure_PV < 140.0 or > 160.0 bar.


Temperature Tolerance:
Target: Temp_SP = 180.0°C.
Acceptable range: 170.0–190.0°C (Temp_SP ± Temp_Tolerance).
Fault if Temp_PV < 160.0 or > 200.0°C.



Safety Logic

Fault Detection:
Pressure_Fault := TRUE if Pressure_PV < Min_Pressure or > Max_Pressure.
Temp_Fault := TRUE if Temp_PV < Min_Temp or > Max_Temp.


Response:
Close Ammonia_Valve and CO2_Valve.
Halt reaction progress (stay in current state or revert to Idle).
Prevent Reaction_Complete until faults are resolved.


Operator Reset:
Faults must be acknowledged, and Start_Button pressed again to restart.



Process Flow Summary

Idle (Reaction_State = 0):
Wait for Start_Button.
All valves closed, outputs reset.


Loading (Reaction_State = 1):
Open Ammonia_Valve and CO2_Valve.
Monitor Pressure_PV until within 145.0–155.0 bar.
Capture Start_Time.


Reaction (Reaction_State = 2):
Close valves, start Reaction_Timer.
Monitor Pressure_PV (140.0–160.0 bar) and Temp_PV (160.0–200.0°C).
Track Elapsed_Time for diagnostics.


Complete (Reaction_State = 3):
Set Reaction_Complete := TRUE.
Reset timer and return to Idle.



Notes for Developers

Scalability: Adjust Pressure_SP, Temp_SP, Reaction_Duration, or tolerances for different reactor designs.
Diagnostics: Add HMI to display Pressure_PV, Temp_PV, Elapsed_Time, Pressure_Fault, Temp_Fault, and Reaction_State.
Safety Enhancements: Integrate alarms for Pressure_Fault and Temp_Fault with operator notifications.
Maintenance: Log Start_Time and fault events for process audits.
Assumptions: Pressure_PV and Temp_PV are provided by analog sensors; CURRENT_TIME is a PLC system function.

