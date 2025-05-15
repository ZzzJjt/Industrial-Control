Certainly! Below is a self-contained timer function block in IEC 61131-3 Structured Text that meets the specified requirements. This function block includes configurable time duration, start and stop inputs, elapsed time tracking, and output flags for running and done states. It also optionally provides remaining time.

### Flexible and Configurable Timer Function Block

```st
[<title="FlexibleTimerFunctionBlock">]
FUNCTION_BLOCK FlexibleTimerFB
VAR_INPUT
    Start : BOOL;           // Signal to start the timer
    Stop : BOOL;            // Signal to stop the timer
    PresetTime : TIME;      // Time duration for the timer
END_VAR

VAR_OUTPUT
    Running : BOOL := FALSE; // TRUE when the timer is counting
    Done : BOOL := FALSE;    // TRUE when the timer completes
    RemainingTime : TIME;    // Optional: Remaining time until completion
END_VAR

VAR
    StartTime : TIME;        // Time when the timer started
    ElapsedTime : TIME;      // Elapsed time since the timer started
BEGIN
METHOD ExecuteTimer : VOID
BEGIN
    IF Start AND NOT Stop THEN
        IF NOT Running THEN
            StartTime := T#0s;
            Running := TRUE;
        END_IF;

        ElapsedTime := TON(TIME(), StartTime).ET;

        IF ElapsedTime >= PresetTime THEN
            Done := TRUE;
            Running := FALSE;
        ELSE
            Done := FALSE;
        END_IF;

        RemainingTime := PresetTime - ElapsedTime;
    ELSIF Stop THEN
        Running := FALSE;
        Done := FALSE;
        ElapsedTime := T#0s;
        RemainingTime := PresetTime;
    ELSE
        Running := FALSE;
        Done := FALSE;
        ElapsedTime := T#0s;
        RemainingTime := PresetTime;
    END_IF;
END_METHOD

// Main execution method
ExecuteTimer();
END_FUNCTION_BLOCK
```

### Explanation of the Code

#### Inputs
- **Start**: A boolean signal to start the timer.
- **Stop**: A boolean signal to stop the timer.
- **PresetTime**: The configured time duration for the timer.

#### Outputs
- **Running**: A boolean flag indicating if the timer is currently counting.
- **Done**: A boolean flag indicating if the timer has completed.
- **RemainingTime**: The remaining time until the timer completes (optional).

#### Internal Variables
- **StartTime**: The time when the timer started.
- **ElapsedTime**: The elapsed time since the timer started.

#### Method: ExecuteTimer
- **Start Logic**:
  - If `Start` is `TRUE` and `Stop` is `FALSE`, and the timer is not already running (`Running` is `FALSE`), it initializes `StartTime` to `T#0s` and sets `Running` to `TRUE`.
  - It calculates `ElapsedTime` using the `TON` (On-Delay Timer) function.
  - If `ElapsedTime` reaches or exceeds `PresetTime`, it sets `Done` to `TRUE` and `Running` to `FALSE`.
  - Otherwise, it keeps `Done` as `FALSE`.
  - It calculates `RemainingTime` as `PresetTime - ElapsedTime`.
  
- **Stop Logic**:
  - If `Stop` is `TRUE`, it resets `Running` to `FALSE`, `Done` to `FALSE`, and `ElapsedTime` to `T#0s`. It also resets `RemainingTime` to `PresetTime`.

- **Idle Logic**:
  - If neither `Start` nor `Stop` is active, it ensures `Running` and `Done` are `FALSE` and resets `ElapsedTime` and `RemainingTime`.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its `ExecuteTimer` method:

```st
PROGRAM MainProgram
VAR
    MyTimer : FlexibleTimerFB;
    StartSignal : BOOL := FALSE; // Example start signal
    StopSignal : BOOL := FALSE;  // Example stop signal
    PresetDuration : TIME := T#5s; // Example preset time duration
    TimerRunning : BOOL;
    TimerDone : BOOL;
    TimerRemaining : TIME;
BEGIN
    // Configure the timer
    MyTimer.Start := StartSignal;
    MyTimer.Stop := StopSignal;
    MyTimer.PresetTime := PresetDuration;

    // Execute the timer
    MyTimer();

    // Retrieve the results
    TimerRunning := MyTimer.Running;
    TimerDone := MyTimer.Done;
    TimerRemaining := MyTimer.RemainingTime;

    // Output the results
    IF TimerDone THEN
        // Timer has completed
    ELSIF TimerRunning THEN
        // Timer is running
    ELSE
        // Timer is idle
    END_IF;
END_PROGRAM
```

### Key Features
- **Dynamic Control**: Supports dynamic start and stop signals.
- **Scan-Cycle Accuracy**: Uses scan-cycle timing with `TON` function for accurate elapsed time calculation.
- **Optional Remaining Time**: Provides optional remaining time output.
- **Modular Design**: Clear inputs and outputs for easy integration into broader control programs.
- **Robust Handling**: Properly handles start/stop transitions and ensures no race conditions.

This function block provides a flexible and reusable timer mechanism suitable for various automation tasks, ensuring accurate timing and dynamic control capabilities.
