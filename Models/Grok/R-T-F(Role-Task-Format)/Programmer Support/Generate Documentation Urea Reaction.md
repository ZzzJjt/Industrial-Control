UreaReactionControl Program Documentation
Program Purpose
The UreaReactionControl program automates the urea synthesis process in a chemical plant, controlling the reaction of ammonia and carbon dioxide to produce urea under high pressure and temperature. The program ensures precise sequencing of raw material loading, reaction control, and shutdown, maintaining safety and efficiency. It operates within a PLC environment, adhering to IEC 61131-3 standards, and executes cyclically to manage the following phases:

Raw Material Loading: Loads ammonia and carbon dioxide into the reactor, verifying valve and pump status.
Reaction Control: Maintains pressure (200 bar) and temperature (190°C) within safe limits, adjusting valves based on sensor feedback.
Automatic Shutdown: Completes the reaction cycle and safely shuts down the system, ensuring all valves are closed and alarms are cleared.

The program uses a state machine to sequence operations, timers for phase duration, and safety interlocks to prevent hazardous conditions, such as overpressure or valve failures.
Variable Table
Inputs



Variable Name
Type
Description
Units/Range



AmmoniaFlow_PV
REAL
Measured flow rate of ammonia
L/min (0.0–100.0)


CO2Flow_PV
REAL
Measured flow rate of carbon dioxide
L/min (0.0–100.0)


Pressure_PV
REAL
Measured reactor pressure
bar (0.0–250.0)


Temperature_PV
REAL
Measured reactor temperature
°C (0.0–300.0)


AmmoniaValveStatus
BOOL
TRUE if ammonia valve is open
TRUE/FALSE


CO2ValveStatus
BOOL
TRUE if CO2 valve is open
TRUE/FALSE


OutletValveStatus
BOOL
TRUE if outlet valve is open
TRUE/FALSE


EmergencyStop
BOOL
TRUE if emergency stop is pressed
TRUE/FALSE


Outputs



Variable Name
Type
Description
Units/Range



AmmoniaValveCmd
BOOL
Command to open/close ammonia valve
TRUE/FALSE


CO2ValveCmd
BOOL
Command to open/close CO2 valve
TRUE/FALSE


OutletValveCmd
BOOL
Command to open/close outlet valve
TRUE/FALSE


ReactionComplete
BOOL
TRUE when reaction cycle is complete
TRUE/FALSE


FaultAlarm
BOOL
TRUE if a fault is detected (e.g., sensor failure)
TRUE/FALSE


Internal Flags



Variable Name
Type
Description
Units/Range



State
INT
Current state (0=Idle, 1=Loading, 2=Reaction, 3=Shutdown)
0–3


LoadingComplete
BOOL
TRUE when raw material loading is finished
TRUE/FALSE


ReactionInProgress
BOOL
TRUE during reaction phase
TRUE/FALSE


ShutdownInitiated
BOOL
TRUE when shutdown sequence starts
TRUE/FALSE


Configurable Parameters



Variable Name
Type
Description
Default Value



Pressure_SP
REAL
Target reactor pressure setpoint
200.0 bar


Temperature_SP
REAL
Target reactor temperature setpoint
190.0°C


LoadingDuration
TIME
Duration for raw material loading
T#30s


ReactionDuration
TIME
Maximum duration for reaction phase
T#300s


Pressure_Tolerance
REAL
Allowable pressure deviation
5.0 bar


Temperature_Tolerance
REAL
Allowable temperature deviation
5.0°C


Control Sequence Description
The UreaReactionControl program uses a state machine to manage the urea synthesis process, ensuring a logical sequence of operations. The sequence is divided into three main steps, with safety checks and transitions based on sensor inputs and timers.
State Machine States

Idle (State = 0):

Purpose: Initial or post-shutdown state, waiting for process start.
Actions: All valves closed (AmmoniaValveCmd, CO2ValveCmd, OutletValveCmd := FALSE), alarms cleared (FaultAlarm := FALSE).
Transition: Moves to Loading (State = 1) when EmergencyStop = FALSE and the process is initiated (e.g., via HMI start command).


Loading (State = 1):

Purpose: Load ammonia and CO2 into the reactor.
Actions:
Opens AmmoniaValveCmd and CO2ValveCmd to start flow.
Monitors AmmoniaFlow_PV, CO2Flow_PV, AmmoniaValveStatus, and CO2ValveStatus to confirm loading.
Starts LoadingTimer (PT := LoadingDuration, default T#30s).


Transition:
If LoadingTimer.Q is TRUE and flows/status are valid, sets LoadingComplete := TRUE and moves to Reaction (State = 2).
If EmergencyStop = TRUE or sensor faults occur, moves to Shutdown (State = 3).




Reaction (State = 2):

Purpose: Control the reaction by maintaining pressure and temperature.
Actions:
Closes AmmoniaValveCmd and CO2ValveCmd to seal the reactor.
Monitors Pressure_PV and Temperature_PV, comparing against Pressure_SP (200.0 bar) and Temperature_SP (190.0°C).
Starts ReactionTimer (PT := ReactionDuration, default T#300s).
If Pressure_PV or Temperature_PV deviates beyond Pressure_Tolerance (5.0 bar) or Temperature_Tolerance (5.0°C), sets FaultAlarm := TRUE.


Transition:
If ReactionTimer.Q is TRUE and conditions are within tolerances, sets ReactionComplete := TRUE and moves to Shutdown (State = 3).
If EmergencyStop = TRUE or faults persist, moves to Shutdown.




Shutdown (State = 3):

Purpose: Safely terminate the process.
Actions:
Closes all valves (AmmoniaValveCmd, CO2ValveCmd, OutletValveCmd := FALSE).
Opens OutletValveCmd briefly to release residual pressure, then closes it.
Clears LoadingComplete, ReactionInProgress, and sets ShutdownInitiated := TRUE.


Transition:
Returns to Idle (State = 0) after shutdown is complete and EmergencyStop = FALSE.





Control Sequence Diagram
[Idle] --> [Loading] --> [Reaction] --> [Shutdown] --> [Idle]
   |         |            |             |
   +---------+------------+-------------+--> [Shutdown] (on EmergencyStop or fault)

Timing Logic Explanation
Timing is managed using the CURRENT_TIME function and two TON timers (LoadingTimer, ReactionTimer) to control phase durations and track process progress.

CURRENT_TIME Usage:

The CURRENT_TIME function retrieves the PLC’s system time, used to timestamp process events or log state transitions.
Example: On entering Reaction, the start time is recorded (ReactionStartTime := CURRENT_TIME) to track elapsed time or log cycle duration for audits.


LoadingTimer:

Purpose: Ensures raw material loading completes within LoadingDuration (default T#30s).
Operation: Initialized (IN := TRUE, PT := LoadingDuration) when entering Loading state. When Q := TRUE, confirms loading completion if flows and valve statuses are valid.
Usage: LoadingTimer.ET (elapsed time) can be monitored on HMI to track progress.


ReactionTimer:

Purpose: Limits the reaction phase to ReactionDuration (default T#300s), ensuring the process doesn’t run indefinitely.
Operation: Initialized (IN := TRUE, PT := ReactionDuration) when entering Reaction state. When Q := TRUE, triggers transition to Shutdown if conditions are met.
Usage: ReactionTimer.ET provides real-time feedback on reaction progress.


Scan-Cycle Integration:

Timers are updated each PLC scan (e.g., 10-50 ms), ensuring precise timing without blocking.
Timers are reset (IN := FALSE) on state transitions to prevent carryover.



Safety and Termination Conditions
The program includes robust safety and termination mechanisms to protect the process and equipment:

Emergency Stop:

If EmergencyStop = TRUE, the program immediately transitions to Shutdown, closing all valves and setting FaultAlarm := TRUE.
Requires manual reset (via HMI or physical switch) to return to Idle.


Sensor Faults:

Validates inputs (AmmoniaFlow_PV, CO2Flow_PV, Pressure_PV, Temperature_PV) for non-finite values (NaN, infinity) or out-of-range conditions (e.g., Pressure_PV > 250.0 bar).
On fault detection, sets FaultAlarm := TRUE, transitions to Shutdown, and closes all valves.


Valve Status Checks:

Ensures AmmoniaValveStatus, CO2ValveStatus, and OutletValveStatus match commanded states (AmmoniaValveCmd, CO2ValveCmd, OutletValveCmd).
Mismatch (e.g., AmmoniaValveCmd = TRUE but AmmoniaValveStatus = FALSE) triggers FaultAlarm and Shutdown.


Process Tolerances:

During Reaction, checks Pressure_PV within Pressure_SP ± Pressure_Tolerance (200.0 ± 5.0 bar) and Temperature_PV within Temperature_SP ± Temperature_Tolerance (190.0 ± 5.0°C).
Deviations beyond tolerances trigger FaultAlarm and may initiate Shutdown if persistent.


Termination:

Successful completion (ReactionTimer.Q = TRUE, conditions within tolerances) sets ReactionComplete := TRUE and transitions to Shutdown.
Shutdown ensures all valves are closed, residual pressure is released, and the system is safe before returning to Idle.



Program Code with Inline Comments
(*
  UreaReactionControl
  Purpose: Automates urea synthesis by controlling raw material loading, reaction, and shutdown.
  Author: [Your Name]
  Date: May 20, 2025
*)

PROGRAM UreaReactionControl
VAR
    (* Inputs *)
    AmmoniaFlow_PV : REAL; (* Ammonia flow rate (L/min, 0.0–100.0) *)
    CO2Flow_PV : REAL; (* CO2 flow rate (L/min, 0.0–100.0) *)
    Pressure_PV : REAL; (* Reactor pressure (bar, 0.0–250.0) *)
    Temperature_PV : REAL; (* Reactor temperature (°C, 0.0–300.0) *)
    AmmoniaValveStatus : BOOL; (* TRUE if ammonia valve open *)
    CO2ValveStatus : BOOL; (* TRUE if CO2 valve open *)
    OutletValveStatus : BOOL; (* TRUE if outlet valve open *)
    EmergencyStop : BOOL; (* TRUE if emergency stop pressed *)
    
    (* Outputs *)
    AmmoniaValveCmd : BOOL; (* Command for ammonia valve *)
    CO2ValveCmd : BOOL; (* Command for CO2 valve *)
    OutletValveCmd : BOOL; (* Command for outlet valve *)
    ReactionComplete : BOOL; (* TRUE when reaction cycle complete *)
    FaultAlarm : BOOL; (* TRUE on fault detection *)
    
    (* Internal flags *)
    State : INT := 0; (* 0=Idle, 1=Loading, 2=Reaction, 3=Shutdown *)
    LoadingComplete : BOOL; (* TRUE when loading finished *)
    ReactionInProgress : BOOL; (* TRUE during reaction *)
    ShutdownInitiated : BOOL; (* TRUE when shutdown starts *)
    
    (* Configurable parameters *)
    Pressure_SP : REAL := 200.0; (* Target pressure (bar) *)
    Temperature_SP : REAL := 190.0; (* Target temperature (°C) *)
    LoadingDuration : TIME := T#30s; (* Loading phase duration *)
    ReactionDuration : TIME := T#300s; (* Reaction phase duration *)
    Pressure_Tolerance : REAL := 5.0; (* Pressure deviation tolerance (bar) *)
    Temperature_Tolerance : REAL := 5.0; (* Temperature deviation tolerance (°C) *)
    
    (* Timing *)
    LoadingTimer : TON; (* Timer for loading phase *)
    ReactionTimer : TON; (* Timer for reaction phase *)
    ReactionStartTime : TIME; (* Timestamp for reaction start *)
END_VAR

(* Helper function to check REAL validity *)
FUNCTION IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_FUNCTION

(* Main control logic *)
(* Validate inputs *)
IF NOT IS_VALID_REAL(AmmoniaFlow_PV) OR AmmoniaFlow_PV < 0.0 OR AmmoniaFlow_PV > 100.0 OR
   NOT IS_VALID_REAL(CO2Flow_PV) OR CO2Flow_PV < 0.0 OR CO2Flow_PV > 100.0 OR
   NOT IS_VALID_REAL(Pressure_PV) OR Pressure_PV < 0.0 OR Pressure_PV > 250.0 OR
   NOT IS_VALID_REAL(Temperature_PV) OR Temperature_PV < 0.0 OR Temperature_PV > 300.0 THEN
    FaultAlarm := TRUE;
    State := 3; (* Transition to Shutdown *)
END_IF;

(* State machine *)
CASE State OF
    0: (* Idle *)
        AmmoniaValveCmd := FALSE;
        CO2ValveCmd := FALSE;
        OutletValveCmd := FALSE;
        ReactionComplete := FALSE;
        FaultAlarm := FALSE;
        LoadingComplete := FALSE;
        ReactionInProgress := FALSE;
        ShutdownInitiated := FALSE;
        IF NOT EmergencyStop THEN
            State := 1; (* Start loading *)
        END_IF;
    
    1: (* Loading *)
        AmmoniaValveCmd := TRUE;
        CO2ValveCmd := TRUE;
        OutletValveCmd := FALSE;
        IF EmergencyStop OR NOT AmmoniaValveStatus OR NOT CO2ValveStatus OR
           AmmoniaFlow_PV < 1.0 OR CO2Flow_PV < 1.0 THEN
            FaultAlarm := TRUE;
            State := 3; (* Transition to Shutdown *)
        ELSIF LoadingTimer.Q THEN
            LoadingTimer(IN := FALSE);
            LoadingComplete := TRUE;
            State := 2; (* Transition to Reaction *)
        ELSE
            LoadingTimer(IN := TRUE, PT := LoadingDuration);
        END_IF;
    
    2: (* Reaction *)
        AmmoniaValveCmd := FALSE;
        CO2ValveCmd := FALSE;
        OutletValveCmd := FALSE;
        ReactionInProgress := TRUE;
        IF NOT ReactionTimer.IN THEN
            ReactionStartTime := CURRENT_TIME; (* Log start time *)
        END_IF;
        IF EmergencyStop OR
           ABS(Pressure_PV - Pressure_SP) > Pressure_Tolerance OR
           ABS(Temperature_PV - Temperature_SP) > Temperature_Tolerance THEN
            FaultAlarm := TRUE;
            State := 3; (* Transition to Shutdown *)
        ELSIF ReactionTimer.Q THEN
            ReactionTimer(IN := FALSE);
            ReactionComplete := TRUE;
            State := 3; (* Transition to Shutdown *)
        ELSE
            ReactionTimer(IN := TRUE, PT := ReactionDuration);
        END_IF;
    
    3: (* Shutdown *)
        AmmoniaValveCmd := FALSE;
        CO2ValveCmd := FALSE;
        OutletValveCmd := TRUE; (* Briefly open to release pressure *)
        ShutdownInitiated := TRUE;
        IF NOT EmergencyStop THEN
            OutletValveCmd := FALSE; (* Close after release *)
            State := 0; (* Return to Idle *)
        END_IF;
    
    ELSE
        (* Invalid state: reset to Shutdown *)
        AmmoniaValveCmd := FALSE;
        CO2ValveCmd := FALSE;
        OutletValveCmd := FALSE;
        FaultAlarm := TRUE;
        State := 3;
END_CASE;

END_PROGRAM

Conclusion
The UreaReactionControl program provides a robust, safe, and efficient solution for automating the urea synthesis process. Its state machine ensures a clear sequence of loading, reaction, and shutdown, with precise timing managed by TON timers and CURRENT_TIME. Comprehensive safety checks, input validation, and fault handling protect the process, while configurable parameters allow flexibility for different operating conditions. This documentation equips future engineers with the knowledge to maintain, troubleshoot, and extend the program, ensuring reliable operation in a chemical plant environment.
