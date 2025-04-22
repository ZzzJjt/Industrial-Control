```
FUNCTION_BLOCK DigitalCounter
VAR_INPUT
    mode : BOOL; // TRUE for UP, FALSE for DOWN
    enable : BOOL; // Enable signal to trigger counting
    initialValue : INT; // Initial value of the counter
    stepSize : INT; // Step size for increment/decrement
    minValue : INT; // Minimum value before underflow
    maxValue : INT; // Maximum value before overflow
END_VAR
VAR_OUTPUT
    currentValue : INT; // Current value of the counter
END_VAR
VAR
    firstCall : BOOL := TRUE; // Flag to set initial value only once
END_VAR

// On the first call, set the initial value
IF firstCall THEN
    currentValue := initialValue;
    firstCall := FALSE;
END_IF;

// Handle counting based on mode and enable signal
IF enable THEN
    IF mode THEN // UP mode
        IF currentValue < maxValue THEN
            currentValue := currentValue + stepSize;
        ELSE
            // Handle overflow
            currentValue := minValue;
        END_IF;
    ELSE // DOWN mode
        IF currentValue > minValue THEN
            currentValue := currentValue - stepSize;
        ELSE
            // Handle underflow
            currentValue := maxValue;
        END_IF;
    END_IF;
END_IF;

RETURN;

END_FUNCTION_BLOCK

PROGRAM CounterExample
VAR
    mode : BOOL := TRUE; // Start in UP mode
    enable : BOOL := TRUE; // Enable counting
    initialValue : INT := 0; // Initial value of the counter
    stepSize : INT := 1; // Step size for increment/decrement
    minValue : INT := 0; // Minimum value before underflow
    maxValue : INT := 100; // Maximum value before overflow
    currentValue : INT; // Variable to store the current value of the counter
BEGIN
    // Initialize the counter
    DigitalCounter(mode:=mode, enable:=enable, initialValue:=initialValue,
                   stepSize:=stepSize, minValue:=minValue, maxValue:=maxValue,
                   currentValue:=currentValue);

    // Optionally, update mode and enable based on some condition
    IF /* some condition */ THEN
        mode := NOT mode; // Toggle mode
    END_IF;

    // Optionally, update enable based on some condition
    enable := /* some condition */;

    // Output the current value of the counter
    WRITE("Current Value: ", currentValue);
END_PROGRAM
```
