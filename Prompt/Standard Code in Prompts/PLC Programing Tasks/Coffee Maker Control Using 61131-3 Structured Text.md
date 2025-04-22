```
PROGRAM CoffeeMachineControl
VAR
    TankLevel : INT := 0; // Level of the mixer tank (0-130 ml)
    MixerRunning : BOOL := FALSE; // Indicates if the mixer is running
    MixingTime : TIME := T#4s; // Duration for the mixer to run
    MixTimer : TIME := T#0s; // Timer for the mixing process
    OutputOpen : BOOL := FALSE; // Indicates if the output valve is open
    CoffeeValveOpen : BOOL := FALSE; // Indicates if the coffee valve is open
    MilkValveOpen : BOOL := FALSE; // Indicates if the milk valve is open
    EmergencyStop : BOOL := FALSE; // Emergency stop button status
    StartButton : BOOL := FALSE; // Start button status
    CoffeeMilkButton : BOOL := FALSE; // Coffee and milk button status
    CoffeeOnlyButton : BOOL := FALSE; // Coffee only button status
    IsMixing : BOOL := FALSE; // Indicates if the system is currently mixing
    IsDispensing : BOOL := FALSE; // Indicates if the system is dispensing
END_VAR

// Main control logic
IF EmergencyStop THEN
    // If emergency stop is pressed, halt all operations and reset the system
    CoffeeValveOpen := FALSE;
    MilkValveOpen := FALSE;
    MixerRunning := FALSE;
    OutputOpen := FALSE;
    MixTimer := T#0s;
    TankLevel := 0;
ELSIF StartButton THEN
    // If start button is pressed, begin the coffee-making process
    IF CoffeeMilkButton THEN
        // If coffee and milk button is pressed, prepare coffee with milk
        CoffeeValveOpen := TRUE;
        MilkValveOpen := TRUE;
    ELSIF CoffeeOnlyButton THEN
        // If coffee only button is pressed, prepare coffee without milk
        CoffeeValveOpen := TRUE;
        MilkValveOpen := FALSE;
    END_IF;
END_IF;

// Fill the mixer tank
IF CoffeeValveOpen OR MilkValveOpen THEN
    IF TankLevel < 130 THEN
        // Increment the tank level
        TankLevel := TankLevel + 1;
    ELSE
        // If the tank is full, close the valves and start mixing
        CoffeeValveOpen := FALSE;
        MilkValveOpen := FALSE;
        MixerRunning := TRUE;
        IsMixing := TRUE;
        MixTimer := T#0s;
    END_IF;
END_IF;

// Mixing process
IF IsMixing THEN
    MixTimer := MixTimer + T#1s;
    IF MixTimer >= MixingTime THEN
        // If mixing is complete, open the output valve
        MixerRunning := FALSE;
        OutputOpen := TRUE;
        IsMixing := FALSE;
        IsDispensing := TRUE;
    END_IF;
END_IF;

// Dispensing process
IF IsDispensing THEN
    // After dispensing, reset the system
    IF TankLevel > 0 THEN
        TankLevel := TankLevel - 1;
    ELSE
        OutputOpen := FALSE;
        IsDispensing := FALSE;
    END_IF;
END_IF;

// Safety features
IF EmergencyStop THEN
    // If emergency stop is pressed, ensure all operations cease
    CoffeeValveOpen := FALSE;
    MilkValveOpen := FALSE;
    MixerRunning := FALSE;
    OutputOpen := FALSE;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("Tank Level", TankLevel);
// Example: Write("Coffee Valve Open", CoffeeValveOpen);
// Example: Write("Milk Valve Open", MilkValveOpen);
// Example: Write("Output Open", OutputOpen);
// Example: Write("Emergency Stop", EmergencyStop);
// Example: Write("Start Button", StartButton);
// Example: Write("Coffee and Milk Button", CoffeeMilkButton);
// Example: Write("Coffee Only Button", CoffeeOnlyButton);
// Example: Write("Is Mixing", IsMixing);
// Example: Write("Is Dispensing", IsDispensing);

END_PROGRAM
```
