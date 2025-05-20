FUNCTION_BLOCK CoffeeMakerController
VAR_INPUT
    EmergencyStop : BOOL; (* TRUE if emergency stop is pressed *)
    Start : BOOL; (* TRUE to initiate brewing process *)
    CoffeeMilk : BOOL; (* TRUE to select coffee with milk *)
    CoffeeOnly : BOOL; (* TRUE to select coffee only *)
    MixerLevelFull : BOOL; (* TRUE when mixer tank is full (130ml) *)
END_VAR
VAR_OUTPUT
    CoffeeValve : BOOL; (* TRUE to open coffee valve *)
    MilkValve : BOOL; (* TRUE to open milk valve *)
    Mixer : BOOL; (* TRUE to run mixer motor *)
    OutputValve : BOOL; (* TRUE to open output valve *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid mode selection *)
END_VAR
VAR
    State : INT := 0; (* State machine: 0=Idle, 1=Filling, 2=Mixing, 3=Dispensing *)
    MixTimer : TON; (* Timer for 4-second mixing phase *)
    PrevStart : BOOL; (* Tracks previous Start state for edge detection *)
END_VAR

(* Emergency stop override: immediate shutdown *)
IF EmergencyStop THEN
    (* Stop all operations and reset to idle *)
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    Mixer := FALSE;
    OutputValve := FALSE;
    MixTimer(IN := FALSE); (* Reset timer *)
    State := 0;
    ErrorCode := 0;
    RETURN;
END_IF;

(* Detect rising edge of Start *)
IF Start AND NOT PrevStart THEN
    (* Validate mode selection *)
    IF CoffeeMilk AND CoffeeOnly THEN
        (* Invalid: both modes selected *)
        ErrorCode := 1;
        State := 0;
    ELSIF NOT CoffeeMilk AND NOT CoffeeOnly THEN
        (* Invalid: no mode selected *)
        ErrorCode := 1;
        State := 0;
    ELSE
        ErrorCode := 0;
    END_IF;
END_IF;

(* State machine *)
CASE State OF
    0: (* Idle *)
        (* Wait for Start and valid mode *)
        IF Start AND NOT PrevStart AND ErrorCode = 0 THEN
            (* Open valves based on mode *)
            IF CoffeeMilk THEN
                CoffeeValve := TRUE;
                MilkValve := TRUE;
            ELSIF CoffeeOnly THEN
                CoffeeValve := TRUE;
                MilkValve := FALSE;
            END_IF;
            State := 1; (* Transition to Filling *)
        END_IF;

    1: (* Filling *)
        (* Wait for mixer tank to fill *)
        IF MixerLevelFull THEN
            (* Close all valves and start mixer *)
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := TRUE;
            MixTimer(IN := TRUE, PT := T#4s); (* Start 4-second timer *)
            State := 2; (* Transition to Mixing *)
        END_IF;

    2: (* Mixing *)
        (* Wait for mixing to complete *)
        IF MixTimer.Q THEN
            (* Stop mixer and open output valve *)
            Mixer := FALSE;
            OutputValve := TRUE;
            MixTimer(IN := FALSE); (* Reset timer *)
            State := 3; (* Transition to Dispensing *)
        END_IF;

    3: (* Dispensing *)
        (* Assume dispensing completes when mixer is empty *)
        (* In practice, add a sensor or timer *)
        IF NOT MixerLevelFull THEN
            (* Close output valve and return to idle *)
            OutputValve := FALSE;
            State := 0;
        END_IF;

    ELSE
        (* Invalid state: reset to idle *)
        CoffeeValve := FALSE;
        MilkValve := FALSE;
        Mixer := FALSE;
        OutputValve := FALSE;
        MixTimer(IN := FALSE);
        State := 0;
END_CASE;

(* Update previous Start state *)
PrevStart := Start;

END_FUNCTION_BLOCK
