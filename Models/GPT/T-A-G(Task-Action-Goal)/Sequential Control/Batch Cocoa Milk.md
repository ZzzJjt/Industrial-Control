VAR
    // Batch control step
    Phase : INT := 0;

    // Ingredient addition flags
    Milk_Added, Water_Added, Sugar_Added, Cocoa_Added : BOOL := FALSE;

    // Heating and mixing control
    Heater : BOOL := FALSE;
    Mixer : BOOL := FALSE;
    Temp_SP : REAL := 70.0;
    Temp_Tolerance : REAL := 2.0;
    Temp_PV : REAL;
    Stirring_Speed : INT := 0;

    // Timer for mixing duration
    MixTimer : TON;
    MixTrig : BOOL := FALSE;
    Mixing_Time : TIME := T#10m;

    // Output status
    Batch_Complete : BOOL := FALSE;
END_VAR

CASE Phase OF

    0: // Add Ingredients
        IF NOT Milk_Added THEN
            AddMilk(60.0);
            Milk_Added := TRUE;
        ELSIF NOT Water_Added THEN
            AddWater(20.0);
            Water_Added := TRUE;
        ELSIF NOT Sugar_Added THEN
            AddSugar(15.0);
            Sugar_Added := TRUE;
        ELSIF NOT Cocoa_Added THEN
            AddCocoa(5.0);
            Cocoa_Added := TRUE;
        ELSE
            Phase := 1; // Proceed to Heating
        END_IF

    1: // Heat to Temperature
        Heater := TRUE;
        IF WithinTemperature(Temp_PV, Temp_SP, Temp_Tolerance) THEN
            Heater := FALSE;
            Phase := 2; // Proceed to Mixing
        END_IF

    2: // Start Mixing
        Mixer := TRUE;
        Stirring_Speed := 200;
        MixTrig := TRUE;
        MixTimer(IN := MixTrig, PT := Mixing_Time);
        IF MixTimer.Q THEN
            MixTrig := FALSE;
            MixTimer(IN := FALSE);
            Mixer := FALSE;
            Stirring_Speed := 0;
            Phase := 3; // Complete
        END_IF

    3: // Batch Complete
        Batch_Complete := TRUE;
        Phase := 4;

    4: // Idle
        ; // Wait for external reset
END_CASE

METHOD AddMilk
    VAR_INPUT Quantity : REAL; END_VAR
    // Trigger milk valve or pump with Quantity

METHOD AddWater
    VAR_INPUT Quantity : REAL; END_VAR
    // Trigger water valve or pump with Quantity

METHOD AddSugar
    VAR_INPUT Quantity : REAL; END_VAR
    // Trigger sugar dosing with Quantity

METHOD AddCocoa
    VAR_INPUT Quantity : REAL; END_VAR
    // Trigger cocoa powder feeder with Quantity

METHOD WithinTemperature : BOOL
    VAR_INPUT Actual, Setpoint, Tol : REAL;
    WithinTemperature := ABS(Actual - Setpoint) <= Tol;
END_METHOD

