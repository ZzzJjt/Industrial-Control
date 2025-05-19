Technical Documentation for UreaReactionControl
1. Purpose
The UreaReactionControl program is an IEC 61131-3 Structured Text (ST) implementation designed to automate a two-phase urea synthesis process in an industrial reactor. The program controls the loading of raw materials (ammonia and CO₂) into the reactor and monitors the reaction process to ensure safe and efficient urea production. It performs the following key functions:

Material Loading Phase: Opens ammonia and CO₂ inlet valves to load raw materials, confirming valve status before proceeding.
Reaction Phase: Monitors pressure and temperature conditions, maintains them within safe thresholds, and times the reaction to completion.
Safety and Completion: Closes valves upon reaction completion or fault detection, setting a finished flag for downstream processes.

The program ensures precise control, compliance with safety standards, and reliable operation under varying reactor conditions, making it suitable for chemical processing plants.
2. Variable List
Below is a comprehensive list of variables used in the UreaReactionControl program, including their data types, roles, and descriptions. Variables are categorized as inputs, outputs, internal control variables, and reaction parameters.
Inputs



Variable Name
Type
Description



Ammonia_Valve_Status
BOOL
Status of ammonia inlet valve (TRUE = open, FALSE = closed).


CO2_Valve_Status
BOOL
Status of CO₂ inlet valve (TRUE = open, FALSE = closed).


Pressure_PV
REAL
Measured reactor pressure (bar), from pressure sensor.


Temperature_PV
REAL
Measured reactor temperature (°C), from temperature sensor.


Start_Command
BOOL
Operator or upstream signal to initiate the reaction process (TRUE = start).


Outputs



Variable Name
Type
Description



Ammonia_Valve_Cmd
BOOL
Command to ammonia inlet valve (TRUE = open, FALSE = close).


CO2_Valve_Cmd
BOOL
Command to CO₂ inlet valve (TRUE = open, FALSE = close).


Reaction_Finished
BOOL
Flag indicating reaction completion (TRUE = finished, FALSE = in progress).


Fault_Alarm
BOOL
Alarm indicating unsafe conditions or process failure (TRUE = fault).


Internal Control Variables



Variable Name
Type
Description



Reaction_State
INT
State variable for control flow (0 = Idle, 1 = Loading, 2 = Reacting, 3 = Complete, 4 = Fault).


Phase_Timer
TON
Timer instance to track reaction duration.


Timer_Running
BOOL
Flag indicating if the reaction timer is active (TRUE = running).


Prev_Start_Command
BOOL
Previous state of Start_Command for edge detection.


Reaction Parameters



Variable Name
Type
Description



Pressure_Min
REAL
Minimum allowable pressure (e.g., 150.0 bar).


Pressure_Max
REAL
Maximum allowable pressure (e.g., 200.0 bar).


Temperature_Min
REAL
Minimum allowable temperature (e.g., 180.0°C).


Temperature_Max
REAL
Maximum allowable temperature (e.g., 220.0°C).


Reaction_Duration
TIME
Required reaction time (e.g., T#300s for 5 minutes).


Pressure_Tolerance
REAL
Acceptable pressure deviation (e.g., 5.0 bar) for stability checks.


Temperature_Tolerance
REAL
Acceptable temperature deviation (e.g., 5.0°C) for stability checks.


3. Logic Flow
The UreaReactionControl program operates as a finite state machine (FSM) with distinct states to manage the urea synthesis process. The control flow is executed within each PLC scan cycle, ensuring compatibility with cyclic execution. Below is a step-by-step explanation of the logic, aligned with the provided steps.
State Machine Overview
The program uses Reaction_State (INT) to track progress:

0: Idle – Waiting for Start_Command to initiate the process.
1: Loading – Opens ammonia and CO₂ valves, waits for confirmation.
2: Reacting – Monitors pressure/temperature, runs the reaction timer.
3: Complete – Closes valves, sets Reaction_Finished.
4: Fault – Closes valves, sets Fault_Alarm on unsafe conditions.

Step-by-Step Control Behavior

Idle State (Reaction_State = 0):

Condition: Program starts here or returns after completion/fault.
Actions:
All outputs are off: Ammonia_Valve_Cmd := FALSE, CO2_Valve_Cmd := FALSE, Reaction_Finished := FALSE, Fault_Alarm := FALSE.
Resets internal variables: Phase_Timer.IN := FALSE, Timer_Running := FALSE, Integral := 0.0.
Monitors Start_Command for a rising edge (using Prev_Start_Command).


Transition:
If Start_Command = TRUE and Prev_Start_Command = FALSE, move to Loading state (Reaction_State := 1).
Update Prev_Start_Command := Start_Command.




Loading State (Reaction_State = 1):

Actions:
Opens valves: Ammonia_Valve_Cmd := TRUE, CO2_Valve_Cmd := TRUE.
Checks valve status: Ammonia_Valve_Status and CO2_Valve_Status.


Transition:
If both Ammonia_Valve_Status = TRUE and CO2_Valve_Status = TRUE, move to Reacting state (Reaction_State := 2).
If valves fail to open within a timeout (e.g., 10 s, using a separate TON), move to Fault state (Reaction_State := 4, Fault_Alarm := TRUE).




Reacting State (Reaction_State = 2):

Actions:
Verifies pressure and temperature are within safe ranges:
Pressure_Min ≤ Pressure_PV ≤ Pressure_Max
Temperature_Min ≤ Temperature_PV ≤ Temperature_Max


Starts the reaction timer: Phase_Timer.IN := TRUE, Phase_Timer.PT := Reaction_Duration.
Monitors timer: Timer_Running := TRUE while Phase_Timer.Q = FALSE.
Checks condition stability (optional):
Ensures ABS(Pressure_PV - Target_Pressure) ≤ Pressure_Tolerance.
Ensures ABS(Temperature_PV - Target_Temperature) ≤ Temperature_Tolerance.




Transition:
If Phase_Timer.Q = TRUE (reaction time complete) and conditions are stable, move to Complete state (Reaction_State := 3).
If pressure or temperature exceeds safe limits, move to Fault state (Reaction_State := 4, Fault_Alarm := TRUE).
If conditions become unstable (outside tolerance), may pause timer or trigger fault (configurable).




Complete State (Reaction_State = 3):

Actions:
Closes valves: Ammonia_Valve_Cmd := FALSE, CO2_Valve_Cmd := FALSE.
Sets completion flag: Reaction_Finished := TRUE.
Stops timer: Phase_Timer.IN := FALSE, Timer_Running := FALSE.


Transition:
If Start_Command = FALSE (operator resets), return to Idle state (Reaction_State := 0).
Remains in Complete until reset to allow downstream processes to act.




Fault State (Reaction_State = 4):

Actions:
Closes valves: Ammonia_Valve_Cmd := FALSE, CO2_Valve_Cmd := FALSE.
Sets fault flag: Fault_Alarm := TRUE.
Stops timer: Phase_Timer.IN := FALSE, Timer_Running := FALSE.


Transition:
If Start_Command = FALSE and fault is cleared (e.g., operator acknowledgment), return to Idle state (Reaction_State := 0, Fault_Alarm := FALSE).





4. Timer Logic
The reaction timing is managed using a TON (Timer On Delay) instance (Phase_Timer), which tracks the duration of the reaction process in the Reacting state. Key aspects of the timer logic include:

Initialization:

In the Idle and Complete states, the timer is disabled: Phase_Timer.IN := FALSE.
The preset time is set to Reaction_Duration (e.g., T#300s for 5 minutes) upon entering the Reacting state.


Operation:

In the Reacting state, the timer is activated: Phase_Timer.IN := TRUE, Phase_Timer.PT := Reaction_Duration.
The timer runs continuously, incrementing its elapsed time (Phase_Timer.ET) each scan cycle until Phase_Timer.Q = TRUE (indicating ET ≥ PT).
The Timer_Running flag is set to TRUE while the timer is active, aiding in state monitoring.


Completion:

When Phase_Timer.Q = TRUE, the reaction is considered complete (assuming pressure/temperature conditions are met), triggering a transition to the Complete state.
The timer is stopped (Phase_Timer.IN := FALSE) to reset ET and prepare for the next cycle.


Fault Handling:

If a fault occurs (e.g., pressure/temperature out of range), the timer is stopped (Phase_Timer.IN := FALSE) to pause the process, ensuring safety.
The timer state is preserved across scans, as Phase_Timer is a persistent instance.


Usage:

The timer’s elapsed time (Phase_Timer.ET) can be monitored via HMI for real-time progress tracking.
The CURRENT_TIME mentioned in the input likely refers to the timer's internal clock or Phase_Timer.ET, used to measure the reaction duration against Reaction_Duration.



5. Valve Triggering
Valve control is critical to the urea reaction process. The program triggers the ammonia and CO₂ inlet valves as follows:

Ammonia Valve (Ammonia_Valve_Cmd):

On: Set to TRUE in the Loading state to open the valve and allow ammonia inflow.
Off: Set to FALSE in the Complete, Fault, or Idle states to close the valve, stopping inflow.
Confirmation: The program checks Ammonia_Valve_Status = TRUE to confirm the valve is open before proceeding to the Reacting state.


CO₂ Valve (CO2_Valve_Cmd):

On: Set to TRUE in the Loading state to open the valve and allow CO₂ inflow.
Off: Set to FALSE in the Complete, Fault, or Idle states to close the valve, stopping inflow.
Confirmation: The program checks CO2_Valve_Status = TRUE to confirm the valve is open before proceeding.


Trigger Conditions:

Valves are opened simultaneously in the Loading state upon receiving Start_Command.
Valves are closed immediately in the Fault state to ensure safety or in the Complete state to finalize the process.
A timeout mechanism (not detailed in the code but recommended) prevents indefinite waiting if a valve fails to open.


Safety Notes:

Valve commands are boolean, assuming digital outputs to actuators.
The program assumes reliable valve feedback (Ammonia_Valve_Status, CO2_Valve_Status). Additional logic (e.g., timeout or fault detection) may be needed for robustness.



6. Reaction Success Criteria
The urea reaction is deemed successful when the following conditions are met in the Reacting state:

Pressure Conditions:

Pressure_PV must remain within Pressure_Min (e.g., 150.0 bar) and Pressure_Max (e.g., 200.0 bar).
Optional stability check: ABS(Pressure_PV - Target_Pressure) ≤ Pressure_Tolerance (e.g., 5.0 bar) ensures consistent pressure.


Temperature Conditions:

Temperature_PV must remain within Temperature_Min (e.g., 180.0°C) and Temperature_Max (e.g., 220.0°C).
Optional stability check: ABS(Temperature_PV - Target_Temperature) ≤ Temperature_Tolerance (e.g., 5.0°C) ensures consistent temperature.


Timing:

The reaction must complete the full duration specified by Reaction_Duration (e.g., T#300s).
The Phase_Timer.Q = TRUE indicates the timer has reached or exceeded Reaction_Duration without interruption.


Fault Absence:

No faults (e.g., valve failure, pressure/temperature out of range) occur during the Reacting state.
If any condition violates the thresholds, the program transitions to the Fault state, setting Fault_Alarm := TRUE.


Outcome:

Upon meeting all criteria, the program transitions to the Complete state, sets Reaction_Finished := TRUE, and closes both valves.
The Reaction_Finished flag signals downstream processes (e.g., product extraction) to proceed.



7. Additional Notes

Safety Considerations:

The program includes fault detection for out-of-range pressure/temperature and valve failures, ensuring compliance with chemical process safety standards.
Engineers should verify sensor calibration (pressure, temperature) and valve reliability before deployment.
Consider adding a manual override or emergency stop input for operator intervention.


Extensibility:

The FSM structure allows easy addition of states (e.g., pre-heating, cooling) or conditions (e.g., flow rate monitoring).
Parameters like Reaction_Duration, Pressure_Min/Max, and Temperature_Min/Max can be adjusted via HMI for process optimization.


Maintenance:

Regularly check Phase_Timer accuracy against PLC clock drift.
Monitor Fault_Alarm logs to identify recurring issues (e.g., valve sticking, sensor drift).
Document any changes to thresholds or timings for traceability.


Compliance:

The program supports safety and quality compliance by enforcing strict pressure/temperature thresholds and providing clear fault indicators.
Aligns with IEC 61131-3 standards for PLC programming, ensuring portability across compliant platforms.



8. Conclusion
The UreaReactionControl program provides a robust, safe, and efficient solution for automating the urea synthesis process. Its finite state machine design, clear variable structure, and precise timer logic ensure reliable operation, while fault detection and output control enhance safety. This documentation serves as a ready-to-use reference for engineers, supporting maintenance, verification, and future development of the urea reaction control system.
For further details or to review the actual ST code, please provide the program source or specific implementation questions.
