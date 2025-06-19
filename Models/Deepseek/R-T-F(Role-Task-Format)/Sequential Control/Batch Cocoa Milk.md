PROGRAM CocoaMilkBatchControl
VAR
    // State management variables
    currentState : INT := 0; // 0: Idle, 1: Adding Ingredients, 2: Heating, 3: Blending, 4: Complete

    // Parameters for Ingredient Addition
    MilkQuantity : REAL := 50.0; // Quantity of milk in kg
    WaterQuantity : REAL := 30.0; // Quantity of water in kg
    LiquidSugarQuantity : REAL := 15.0; // Quantity of liquid sugar in kg
    CocoaQuantity : REAL := 5.0; // Quantity of cocoa in kg

    // Parameters for Heating Phase
    HeatingTargetTemperature : REAL := 70.0; // Target temperature in Celsius
    HeatingTime : TIME := T#10m; // Heating time

    // Parameters for Blending Phase
    BlendingSpeed : REAL := 200.0; // Stirring speed in RPM
    BlendingTime : TIME := T#10m; // Blending time

    // Timers
    HeatingTimer : TON;
    BlendingTimer : TON;

    // Flags
    IngredientsAdded : BOOL := FALSE;
    HeatingComplete : BOOL := FALSE;
    BlendingComplete : BOOL := FALSE;

    // Interlocks
    VesselInterlock : BOOL := FALSE; // Ensure vessel is empty before adding ingredients
END_VAR

// Method declarations
METHOD AddIngredient(this : REFERENCE TO CocoaMilkBatchControl, Ingredient : STRING, Quantity : REAL) : BOOL
BEGIN
    CASE Ingredient OF
        'Milk':
            // Implement milk addition logic here
            // Example: AddMilkToVessel(Quantity);
            RETURN TRUE;
        'Water':
            // Implement water addition logic here
            // Example: AddWaterToVessel(Quantity);
            RETURN TRUE;
        'LiquidSugar':
            // Implement liquid sugar addition logic here
            // Example: AddLiquidSugarToVessel(Quantity);
            RETURN TRUE;
        'Cocoa':
            // Implement cocoa addition logic here
            // Example: AddCocoaToVessel(Quantity);
            RETURN TRUE;
        ELSE:
            RETURN FALSE;
    END_CASE;
END_METHOD

METHOD StartHeating(this : REFERENCE TO CocoaMilkBatchControl) : BOOL
BEGIN
    // Implement heating logic here
    // Example: SetVesselTemperature(HeatingTargetTemperature);
    RETURN TRUE;
END_METHOD

METHOD StartBlending(this : REFERENCE TO CocoaMilkBatchControl) : BOOL
BEGIN
    // Implement blending logic here
    // Example: SetVesselMixingSpeed(BlendingSpeed);
    RETURN TRUE;
END_METHOD

// Main control loop
CASE currentState OF
    0: // Idle state
        IF StartBatch THEN
            IF NOT VesselInterlock THEN
                currentState := 1; // Transition to Adding Ingredients state
            END_IF;
        END_IF;

    1: // Adding Ingredients state
        // Add milk
        AddIngredient('Milk', MilkQuantity);
        // Add water
        AddIngredient('Water', WaterQuantity);
        // Add liquid sugar
        AddIngredient('LiquidSugar', LiquidSugarQuantity);
        // Add cocoa
        AddIngredient('Cocoa', CocoaQuantity);
        IngredientsAdded := TRUE;
        currentState := 2; // Transition to Heating state

    2: // Heating state
        StartHeating(); // Start heating process
        HeatingTimer(IN := TRUE, PT := HeatingTime);
        IF HeatingTimer.Q THEN
            HeatingTimer(IN := FALSE);
            HeatingComplete := TRUE;
            currentState := 3; // Transition to Blending state
        END_IF;

    3: // Blending state
        StartBlending(); // Start blending process
        BlendingTimer(IN := TRUE, PT := BlendingTime);
        IF BlendingTimer.Q THEN
            BlendingTimer(IN := FALSE);
            BlendingComplete := TRUE;
            currentState := 4; // Transition to Complete state
        END_IF;

    4: // Complete state
        // Batch complete, transition back to Idle
        currentState := 0;

    ELSE:
        currentState := 0; // Reset to Idle state in case of unexpected state
END_CASE;

// In-line comments explaining transitions and control logic
// - The program starts in the Idle state and transitions to Adding Ingredients when StartBatch is triggered.
// - During Ingredient Addition, specified quantities of milk, water, liquid sugar, and cocoa are added to the vessel.
// - After all ingredients are added, the program transitions to Heating, where the mixture is heated to the target temperature.
// - Once heating is complete, the program transitions to Blending, where the mixture is blended at the specified speed for the required duration.
// - Once blending is complete, the program transitions to the Complete state, indicating that the batch is finished.
// - The program then returns to the Idle state, ready for the next batch.
