PROGRAM BatchControl
VAR
    State : INT := 0; // Initial state
    Timer : TON;
    Temperature : REAL;
    Pressure : REAL;
END_VAR

// Method to update temperatures and pressures based on current state
METHOD UpdateTemperaturesAndPressures
VAR_INPUT
    CurrentState : INT;
END_VAR
BEGIN
    CASE CurrentState OF
        1:
            Temperature := 250.0; // Set initial reactor temperature
            Pressure := 5.0;      // Set initial reactor pressure
        2:
            Temperature := 300.0; // Increase temperature for polymerization
            Pressure := 7.0;      // Increase pressure for polymerization
        3:
            Temperature := 280.0; // Cool down after polymerization
            Pressure := 4.0;      // Reduce pressure after polymerization
        ELSE
            Temperature := 20.0;  // Default room temperature
            Pressure := 1.0;      // Default atmospheric pressure
    END_CASE;
END_METHOD

// Main execution loop
CASE State OF
    0: // Initialization
        Timer(IN := FALSE); // Reset timer
        State := 1;         // Transition to next state
    1: // Heating Phase
        UpdateTemperaturesAndPressures(State);
        Timer(IN := TRUE, PT := T#5m); // Start timer for 5 minutes
        IF Timer.Q THEN
            Timer(IN := FALSE); // Reset timer
            State := 2;         // Transition to next state
        END_IF;
    2: // Polymerization Phase
        UpdateTemperaturesAndPressures(State);
        Timer(IN := TRUE, PT := T#10m); // Start timer for 10 minutes
        IF Timer.Q THEN
            Timer(IN := FALSE); // Reset timer
            State := 3;         // Transition to next state
        END_IF;
    3: // Cooling Phase
        UpdateTemperaturesAndPressures(State);
        Timer(IN := TRUE, PT := T#3m); // Start timer for 3 minutes
        IF Timer.Q THEN
            Timer(IN := FALSE); // Reset timer
            State := 0;         // Return to initialization state
        END_IF;
    ELSE
        State := 0; // Handle unexpected states by resetting to initialization
END_CASE



