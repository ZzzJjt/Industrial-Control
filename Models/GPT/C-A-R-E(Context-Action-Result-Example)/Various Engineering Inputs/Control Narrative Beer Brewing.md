FUNCTION_BLOCK FB_Lautering
VAR_INPUT
    Start : BOOL;
    Reset : BOOL;
    Turbidity : REAL;              // NTU
    WortFlow : REAL;              // L/min
    SpargeTemp : REAL;            // °C
    TunLevel : REAL;              // %
    KettleVolume : REAL;          // L
    TargetVolume : REAL := 100.0; // L
    SpargeWaterReady : BOOL;
END_VAR

VAR_OUTPUT
    Recirculating : BOOL;
    TransferToKettle : BOOL;
    SpargeActive : BOOL;
    DivertToWaste : BOOL;
    Done : BOOL;
END_VAR

VAR
    State : INT := 0;
    RakeActivated : BOOL := FALSE;
END_VAR

// State Machine
IF Reset THEN
    State := 0;
    Recirculating := FALSE;
    TransferToKettle := FALSE;
    SpargeActive := FALSE;
    DivertToWaste := FALSE;
    Done := FALSE;
    RakeActivated := FALSE;

ELSIF Start THEN
    CASE State OF

    0: // Init
        IF NOT Reset THEN
            Recirculating := TRUE;
            State := 1;
        END_IF

    1: // Recirculation (Vorlauf)
        IF Turbidity < 200.0 THEN
            Recirculating := FALSE;
            TransferToKettle := TRUE;
            State := 2;
        END_IF

    2: // Begin Wort Transfer
        IF TunLevel < 30.0 AND SpargeWaterReady THEN
            SpargeActive := TRUE;
            State := 3;
        END_IF

    3: // Sparging
        IF WortFlow < 1.0 THEN
            RakeActivated := TRUE; // Simplified rake control trigger
        END_IF

        IF KettleVolume >= TargetVolume THEN
            SpargeActive := FALSE;
            TransferToKettle := FALSE;

            IF Turbidity > 200.0 THEN
                DivertToWaste := TRUE;
            ELSE
                Done := TRUE;
            END_IF

            State := 4;
        END_IF

    4: // Finished
        // All actions complete

    END_CASE
END_IF
