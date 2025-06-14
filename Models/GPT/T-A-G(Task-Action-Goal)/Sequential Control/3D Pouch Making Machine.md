VAR
    // Operator commands
    Start_Sequence : BOOL;
    Shutdown_Sequence : BOOL;

    // Component outputs
    Heater[1..8] : BOOL;
    Cooler[1..8] : BOOL;
    Feeder_A_Enabled : BOOL;
    Feeder_B_Enabled : BOOL;
    Cutter_Horizontal : BOOL;
    Cutter_Vertical : BOOL;

    // Control
    SequenceStep : INT := 0;
    SeqTimer : TON;
    TimerStart : BOOL;

    // Parameters
    HeatUpTime : TIME := T#5s;
    CoolDownTime : TIME := T#3s;
    FeedDelay : TIME := T#2s;
    CutterSyncDelay : TIME := T#1s;
END_VAR

// Control timer triggering
SeqTimer(IN := TimerStart, PT := HeatUpTime);

// START-UP SEQUENCE
IF Start_Sequence AND NOT Shutdown_Sequence THEN
    CASE SequenceStep OF
        0: // Start heaters
            Heater[1] := TRUE;
            TimerStart := TRUE;
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := 1;
            END_IF

        1..7: // Continue heating sequence
            Heater[SequenceStep + 1] := TRUE;
            TimerStart := TRUE;
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := SequenceStep + 1;
            END_IF

        8: // All heaters done – now start cooling
            Cooler[1] := TRUE;
            TimerStart := TRUE;
            SeqTimer(PT := CoolDownTime);
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := 9;
            END_IF

        9..15:
            Cooler[SequenceStep - 7] := TRUE;
            TimerStart := TRUE;
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := SequenceStep + 1;
            END_IF

        16: // Enable feeders
            TimerStart := TRUE;
            SeqTimer(PT := FeedDelay);
            IF SeqTimer.Q THEN
                Feeder_A_Enabled := TRUE;
                Feeder_B_Enabled := TRUE;
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := 17;
            END_IF

        17: // Synchronize cutters
            TimerStart := TRUE;
            SeqTimer(PT := CutterSyncDelay);
            IF SeqTimer.Q THEN
                Cutter_Horizontal := TRUE;
                Cutter_Vertical := TRUE;
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := 100; // Ready
            END_IF

        100: // MACHINE FULLY STARTED
            ; // Idle in final state
    END_CASE

// SHUTDOWN SEQUENCE
ELSIF Shutdown_Sequence THEN
    CASE SequenceStep OF
        100: // Cutters off
            Cutter_Horizontal := FALSE;
            Cutter_Vertical := FALSE;
            TimerStart := TRUE;
            SeqTimer(PT := CutterSyncDelay);
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := 101;
            END_IF

        101: // Feeders off
            Feeder_A_Enabled := FALSE;
            Feeder_B_Enabled := FALSE;
            TimerStart := TRUE;
            SeqTimer(PT := FeedDelay);
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := 102;
            END_IF

        102..108: // Coolers off in reverse
            Cooler[109 - SequenceStep] := FALSE;
            TimerStart := TRUE;
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := SequenceStep + 1;
            END_IF

        109..116: // Heaters off in reverse
            Heater[117 - SequenceStep] := FALSE;
            TimerStart := TRUE;
            IF SeqTimer.Q THEN
                TimerStart := FALSE;
                SeqTimer(IN := FALSE);
                SequenceStep := SequenceStep + 1;
            END_IF

        117: // Shutdown complete
            SequenceStep := 0;
    END_CASE
END_IF
