PROGRAM PouchMachineControl
VAR
    // State machine
    machineState : INT := 0;

    // Inputs
    StartBtn : BOOL;
    StopBtn  : BOOL;
    TensionOK : BOOL;
    FlowOK : BOOL;

    // Outputs
    HeatersOn : BOOL := FALSE;
    CoolersOn : BOOL := FALSE;
    FeedersOn : BOOL := FALSE;
    HorizontalCutterOn : BOOL := FALSE;
    VerticalCutterOn : BOOL := FALSE;

    // Timers
    tPreheat : TON;
    tCool : TON;
    tCutterDelay : TON;

    // Internal flags
    preheatDone : BOOL := FALSE;
    coolDone : BOOL := FALSE;
    cutterReady : BOOL := FALSE;

END_VAR

// Main logic
tPreheat(IN := machineState = 1, PT := T#60s);
tCool(IN := machineState = 6, PT := T#30s);
tCutterDelay(IN := machineState = 4, PT := T#5s);

CASE machineState OF
    0: // Idle
        IF StartBtn THEN
            machineState := 1;
        ELSIF StopBtn THEN
            // Already idle
            machineState := 0;
        END_IF;

    1: // Preheating
        HeatersOn := TRUE;
        IF tPreheat.Q THEN
            preheatDone := TRUE;
            machineState := 2;
        END_IF;

    2: // Start Coolers
        CoolersOn := TRUE;
        machineState := 3;

    3: // Start Feeders (tension check)
        IF TensionOK THEN
            FeedersOn := TRUE;
            machineState := 4;
        END_IF;

    4: // Wait for FlowOK and tension before cutters
        IF TensionOK AND FlowOK THEN
            tCutterDelay(IN := TRUE);
            IF tCutterDelay.Q THEN
                cutterReady := TRUE;
                machineState := 5;
            END_IF;
        ELSE
            tCutterDelay(IN := FALSE);
        END_IF;

    5: // Start Cutters
        IF cutterReady THEN
            HorizontalCutterOn := TRUE;
            VerticalCutterOn := TRUE;
        END_IF;

        IF StopBtn THEN
            machineState := 6; // Shutdown
        END_IF;

    6: // Shutdown Sequence
        HorizontalCutterOn := FALSE;
        VerticalCutterOn := FALSE;
        FeedersOn := FALSE;

        tCool(IN := TRUE);
        IF tCool.Q THEN
            CoolersOn := FALSE;
            HeatersOn := FALSE;
            machineState := 0;
        END_IF;

END_CASE;
