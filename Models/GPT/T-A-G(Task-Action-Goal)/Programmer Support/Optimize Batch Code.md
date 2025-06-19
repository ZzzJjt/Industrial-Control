VAR
    Step : INT := 0; // Batch step
    TON_Timer : TON; // Shared timer instance
    Timer_Done : BOOL;

    // Process variables
    Temperature_Setpoint : REAL;
    Pressure_Setpoint : REAL;

    // Timer durations per step (in milliseconds)
    t_Step1 : TIME := T#30s;
    t_Step2 : TIME := T#45s;
    t_Step3 : TIME := T#60s;

    // Trigger signal
    Start_Batch : BOOL;

    // Internal flags
    StepInit : BOOL := TRUE;
END_VAR

// Method to update process conditions
METHOD UpdateTemperaturesAndPressures
VAR_INPUT
    Temp : REAL;
    Press : REAL;
END_VAR
    Temperature_Setpoint := Temp;
    Pressure_Setpoint := Press;
END_METHOD

// Main logic
CASE Step OF
    0: // IDLE
        IF Start_Batch THEN
            Step := 1;
            StepInit := TRUE;
        END_IF

    1: // Step 1 – Load Feed
        IF StepInit THEN
            UpdateTemperaturesAndPressures(150.0, 2.5);
            TON_Timer(IN := TRUE, PT := t_Step1);
            StepInit := FALSE;
        END_IF

        TON_Timer(IN := TRUE);
        Timer_Done := TON_Timer.Q;

        IF Timer_Done THEN
            Step := 2;
            StepInit := TRUE;
            TON_Timer(IN := FALSE); // Reset timer
        END_IF

    2: // Step 2 – Start Reaction
        IF StepInit THEN
            UpdateTemperaturesAndPressures(200.0, 4.0);
            TON_Timer(IN := TRUE, PT := t_Step2);
            StepInit := FALSE;
        END_IF

        TON_Timer(IN := TRUE);
        Timer_Done := TON_Timer.Q;

        IF Timer_Done THEN
            Step := 3;
            StepInit := TRUE;
            TON_Timer(IN := FALSE);
        END_IF

    3: // Step 3 – Pressure Hold
        IF StepInit THEN
            UpdateTemperaturesAndPressures(200.0, 6.0);
            TON_Timer(IN := TRUE, PT := t_Step3);
            StepInit := FALSE;
        END_IF

        TON_Timer(IN := TRUE);
        Timer_Done := TON_Timer.Q;

        IF Timer_Done THEN
            Step := 4; // Done
            StepInit := TRUE;
            TON_Timer(IN := FALSE);
        END_IF

    4: // Completed
        // Maintain safe conditions or idle
        UpdateTemperaturesAndPressures(100.0, 1.0);
END_CASE
