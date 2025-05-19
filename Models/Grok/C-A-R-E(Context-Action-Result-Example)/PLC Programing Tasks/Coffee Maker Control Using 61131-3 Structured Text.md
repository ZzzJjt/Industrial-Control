(* IEC 61131-3 Structured Text: CoffeeMachineControl Function Block *)
(* Purpose: Controls a single-station coffee machine with safety interlocks *)

FUNCTION_BLOCK CoffeeMachineControl
VAR_INPUT
    EmergencyStop : BOOL;           (* TRUE to stop all operations *)
    Start : BOOL;                   (* TRUE to begin coffee-making process *)
    CoffeeMilk : BOOL;              (* TRUE to select coffee with milk mode *)
    CoffeeOnly : BOOL;              (* TRUE to select coffee-only mode *)
    MixerLevelFull : BOOL;          (* TRUE when mixer tank reaches 130ml *)
END_VAR
VAR_OUTPUT
    CoffeeValve : BOOL;             (* TRUE to open coffee valve *)
    MilkValve : BOOL;               (* TRUE to open milk valve *)
    OutputValve : BOOL;             (* TRUE to open output valve *)
    Mixer : BOOL;                   (* TRUE to activate mixer *)
    State : INT;                    (* Current state: 0=Idle, 1=Filling, 2=Mixing, 3=Dispensing, 4=Emergency *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    MixTimer : TON;                 (* Timer for 4-second mixing *)
    LastEmergencyStop : BOOL;       (* Previous EmergencyStop state *)
    LastStart : BOOL;               (* Previous Start state *)
    LastCoffeeMilk : BOOL;          (* Previous CoffeeMilk state *)
    LastCoffeeOnly : BOOL;          (* Previous CoffeeOnly state *)
    LastMixerLevelFull : BOOL;      (* Previous MixerLevelFull state *)
    LastState : INT;                (* Previous state for logging *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Fill mixer tank based on mode (CoffeeOnly: CoffeeValve, CoffeeMilk: CoffeeValve + MilkValve)
   - Stop filling at 130ml (MixerLevelFull)
   - Mix for 4 seconds
   - Dispense via OutputValve
   - EmergencyStop halts all operations, resets to Emergency state
   - State machine ensures clear transitions and safety interlocks
*)

(* Step 1: Handle Emergency Stop *)
(* EmergencyStop overrides all operations, halts system *)
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    Mixer := FALSE;
    OutputValve := FALSE;
    MixTimer.IN := FALSE; (* Reset timer *)
    State := 4; (* Emergency state *)
    IF State <> LastState AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:05:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Emergency Stop Activated');
    END_IF;
ELSE
    (* Step 2: State machine execution *)
    CASE State OF
        0: (* Idle: Waiting for Start and mode selection *)
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := FALSE;
            OutputValve := FALSE;
            MixTimer.IN := FALSE;
            IF Start AND NOT CoffeeMilk AND NOT CoffeeOnly THEN
                (* No mode selected *)
                IF LogCount < 50 AND (Start <> LastStart) THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 12:05:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Select CoffeeMilk or CoffeeOnly');
                END_IF;
            ELSIF Start AND CoffeeMilk AND CoffeeOnly THEN
                (* Invalid: Both modes selected *)
                IF LogCount < 50 AND (Start <> LastStart OR CoffeeMilk <> LastCoffeeMilk OR CoffeeOnly <> LastCoffeeOnly) THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 12:05:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Cannot select both CoffeeMilk and CoffeeOnly');
                END_IF;
            ELSIF Start AND CoffeeMilk THEN
                State := 1; (* Transition to Filling *)
                CoffeeValve := TRUE;
                MilkValve := TRUE;
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 12:05:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Filling Started: CoffeeMilk Mode');
                END_IF;
            ELSIF Start AND CoffeeOnly THEN
                State := 1; (* Transition to Filling *)
                CoffeeValve := TRUE;
                MilkValve := FALSE;
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 12:05:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Filling Started: CoffeeOnly Mode');
                END_IF;
            END_IF;

        1: (* Filling: Filling mixer tank *)
            IF MixerLevelFull THEN
                State := 2; (* Transition to Mixing *)
                CoffeeValve := FALSE;
                MilkValve := FALSE;
                Mixer := TRUE;
                MixTimer(IN := TRUE, PT := T#4s);
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 12:05:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Mixing Started: Tank Full');
                END_IF;
            END_IF;

        2: (* Mixing: Mixing for 4 seconds *)
            IF MixTimer.Q THEN
                State := 3; (* Transition to Dispensing *)
                Mixer := FALSE;
                OutputValve := TRUE;
                MixTimer.IN := FALSE;
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 12:05:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Dispensing Started');
                END_IF;
            END_IF;

        3: (* Dispensing: Dispensing beverage *)
            (* Assume dispensing completes instantly for simplicity *)
            (* In practice, add timer or sensor for dispensing completion *)
            State := 0; (* Return to Idle *)
            OutputValve := FALSE;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:05:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Dispensing Completed: Returning to Idle');
            END_IF;

        4: (* Emergency: All operations halted *)
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := FALSE;
            OutputValve := FALSE;
            MixTimer.IN := FALSE;
            IF NOT EmergencyStop THEN
                State := 0; (* Transition to Idle when EmergencyStop cleared *)
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 12:05:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Emergency Cleared: Returning to Idle');
                END_IF;
            END_IF;
    END_CASE;
END_IF;

(* Step 3: Update last states for logging *)
LastEmergencyStop := EmergencyStop;
LastStart := Start;
LastCoffeeMilk := CoffeeMilk;
LastCoffeeOnly := CoffeeOnly;
LastMixerLevelFull := MixerLevelFull;
LastState := State;

(* Step 4: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Controls coffee machine with tanks, valves, mixer, and safety features.
   - Inputs:
     - EmergencyStop: BOOL, TRUE to halt all operations.
     - Start: BOOL, TRUE to begin process.
     - CoffeeMilk: BOOL, TRUE for coffee with milk.
     - CoffeeOnly: BOOL, TRUE for coffee only.
     - MixerLevelFull: BOOL, TRUE at 130ml.
   - Outputs:
     - CoffeeValve: BOOL, TRUE to open coffee valve.
     - MilkValve: BOOL, TRUE to open milk valve.
     - OutputValve: BOOL, TRUE to dispense.
     - Mixer: BOOL, TRUE to activate mixer.
     - State: INT, current state (0=Idle, 1=Filling, 2=Mixing, 3=Dispensing, 4=Emergency).
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Idle: Wait for Start and mode (CoffeeMilk or CoffeeOnly).
     - Filling: Open valves based on mode, stop at MixerLevelFull.
     - Mixing: Run mixer for 4 seconds.
     - Dispensing: Open OutputValve, return to Idle.
     - Emergency: Halt all, reset to Emergency state.
   - State Machine:
     - Idle (0): Ready for start.
     - Filling (1): Filling tank.
     - Mixing (2): Mixing for 4 seconds.
     - Dispensing (3): Dispensing beverage.
     - Emergency (4): Operations halted.
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, deterministic execution.
     - Minimal memory (~4 KB for logs, scalars for states).
   - Safety:
     - EmergencyStop halts all operations immediately.
     - Validates mode selection (mutually exclusive).
     - Logs state changes for traceability.
   - Usage:
     - Coffee vending: Fills, mixes, dispenses based on mode, halts on emergency.
     - Example: Start=TRUE, CoffeeMilk=TRUE → CoffeeValve=TRUE, MilkValve=TRUE.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], BOOL, INT, TON; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
