To create a flexible and reusable timer function block in IEC 61131-3 Structured Text that can be used within PLC systems for various timing needs, follow the design below. This implementation includes configurable duration, runtime control through start and stop signals, tracking of elapsed time, and indication of timeout completion.

### 🧱 FUNCTION_BLOCK: `FB_Timer`

#### ✅ Purpose

A self-contained timer function block to provide configurable timing functionality with runtime controls (start/stop), elapsed time tracking, and timeout indication. Suitable for sequencing operations, delays, and safety checks in industrial automation applications.

---

### 📋 Interface Definition

```pascal
FUNCTION_BLOCK FB_Timer
VAR_INPUT
    Start       : BOOL; // Start signal to activate the timer
    Stop        : BOOL; // Stop signal to deactivate the timer and reset it
    PresetTime  : TIME; // Time duration set by user
END_VAR

VAR_OUTPUT
    Running     : BOOL; // TRUE when the timer is counting
    Done        : BOOL; // TRUE when elapsed time >= preset time
    RemainingTime: TIME; // Optional output showing remaining time until done
END_VAR

VAR
    StartTime   : TIME; // Internal storage for the start time
    ElapsedTime : TIME; // Internal calculation of elapsed time
    TimerActive : BOOL := FALSE; // Internal flag indicating if the timer is active
END_VAR
```

### 🚀 Implementation Logic

```pascal
// Reset on Stop signal or initialization
IF Stop THEN
    TimerActive := FALSE;
    Running := FALSE;
    Done := FALSE;
    ElapsedTime := T#0s;
END_IF;

// Start the timer
IF Start AND NOT TimerActive THEN
    StartTime := TOD_TO_TIME(TOD_NOW()); // Record current time as start time
    TimerActive := TRUE;
    Running := TRUE;
    Done := FALSE;
ELSIF NOT Start AND TimerActive THEN
    // Calculate elapsed time
    ElapsedTime := TOD_TO_TIME(TOD_NOW()) - StartTime;
    
    // Check if elapsed time has reached or exceeded preset time
    IF ElapsedTime >= PresetTime THEN
        TimerActive := FALSE;
        Running := FALSE;
        Done := TRUE;
        RemainingTime := T#0s;
    ELSE
        RemainingTime := PresetTime - ElapsedTime;
        Running := TRUE;
        Done := FALSE;
    END_IF;
ELSE
    // If neither starting nor stopping, keep previous state but update RemainingTime if not done
    IF NOT Done THEN
        ElapsedTime := TOD_TO_TIME(TOD_NOW()) - StartTime;
        RemainingTime := PresetTime - ElapsedTime;
    END_IF;
END_IF;
```

### 🛠 Example Usage in a PLC Program

```pascal
PROGRAM MainProgram
VAR
    MyTimer     : FB_Timer;
    TimerStart  : BOOL := FALSE; // Set to TRUE to start the timer
    TimerStop   : BOOL := FALSE; // Set to TRUE to stop and reset the timer
    TimerDuration: TIME := T#5s; // Preset time duration of 5 seconds
    IsRunning   : BOOL;
    IsDone      : BOOL;
    TimeLeft    : TIME;
END_VAR

// Call the timer function block
MyTimer(
    Start := TimerStart,
    Stop := TimerStop,
    PresetTime := TimerDuration,
    Running => IsRunning,
    Done => IsDone,
    RemainingTime => TimeLeft
);

// Use IsRunning, IsDone, and TimeLeft in your control logic as needed
```

### 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Configurable Duration** | Users can set any desired time duration using `PresetTime`. |
| **Runtime Control** | Provides `Start` and `Stop` signals for controlling the timer dynamically during execution. |
| **Elapsed Time Tracking** | Internally tracks elapsed time and indicates whether the timer is running. |
| **Timeout Indication** | Outputs `Done` flag when the preset time is reached, useful for triggering subsequent actions. |
| **Remaining Time Output** | Optionally provides `RemainingTime`, allowing external monitoring of how much time is left. |

This function block ensures flexibility and maintainability in PLC-based automation projects by offering precise control over timing mechanisms. It's designed to be easily integrated into larger programs and adjusted according to specific application requirements.
