VAR
    // === Phase Step Trackers ===
    Phase_IngredientAdd : INT := 0;
    Phase_HeatBlend     : INT := 0;

    // === Process Completion Flags ===
    IngredientsAdded    : BOOL := FALSE;
    HeatingComplete     : BOOL := FALSE;
    BlendingComplete    : BOOL := FALSE;

    // === Setpoints and Parameters ===
    TargetTemp          : REAL := 70.0;             // °C
    StirSpeed           : INT := 200;               // RPM
    BlendDuration       : TIME := T#10m;            // Blending time

    // === Ingredient Quantities (kg) ===
    QtyMilk             : REAL := 60.0;
    QtyWater            : REAL := 20.0;
    QtySugar            : REAL := 15.0;
    QtyCocoa            : REAL := 5.0;

    // === Process Values ===
    CurrentTemp         : REAL;
    TemperatureReached  : BOOL := FALSE;
    MixerRunning        : BOOL := FALSE;

    // === Timers ===
    BlendTimer          : TON;
    BlendTimerStart     : BOOL := FALSE;

    // === Function/Method Stubs ===
    // PROCEDURE AddIngredient(Name: STRING, Amount: REAL);
    // PROCEDURE StartHeating(TargetTemp: REAL);
    // PROCEDURE StartMixing(Speed: INT);
    // PROCEDURE StopMixing();
END_VAR

// === PHASE 1: INGREDIENT ADDITION ===
CASE Phase_IngredientAdd OF

    0: // Add milk
        AddIngredient(Name := 'Milk', Amount := QtyMilk);
        Phase_IngredientAdd := 1;

    1: // Add water
        AddIngredient(Name := 'Water', Amount := QtyWater);
        Phase_IngredientAdd := 2;

    2: // Add liquid sugar
        AddIngredient(Name := 'LiquidSugar', Amount := QtySugar);
        Phase_IngredientAdd := 3;

    3: // Add cocoa
        AddIngredient(Name := 'Cocoa', Amount := QtyCocoa);
        Phase_IngredientAdd := 4;

    4: // Complete ingredient phase
        IngredientsAdded := TRUE;

END_CASE

// === PHASE 2: HEATING AND BLENDING ===
IF IngredientsAdded THEN
    CASE Phase_HeatBlend OF

        0: // Start heating
            StartHeating(TargetTemp := TargetTemp);
            IF CurrentTemp >= TargetTemp THEN
                TemperatureReached := TRUE;
                Phase_HeatBlend := 1;
            END_IF;

        1: // Start mixing
            StartMixing(Speed := StirSpeed);
            MixerRunning := TRUE;
            Phase_HeatBlend := 2;

        2: // Hold blending for 10 minutes
            IF NOT BlendTimerStart THEN
                BlendTimer(IN := TRUE, PT := BlendDuration);
                BlendTimerStart := TRUE;
            END_IF;

            IF BlendTimer.Q THEN
                StopMixing();
                MixerRunning := FALSE;
                BlendingComplete := TRUE;
                Phase_HeatBlend := 3;
            END_IF;

        3: // Blending complete
            // Optional: Transfer to next unit or hold
    END_CASE;
END_IF
