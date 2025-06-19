Here is a **self-contained, scan-cycle-safe PID controller function block** written in **IEC 61131-3 Structured Text**, designed for industrial automation applications such as temperature, pressure, or position control.

This implementation includes:

✅ Tunable **Kp, Ki, Kd gains**  
✅ **Anti-windup** to prevent integral term saturation  
✅ Output **clamping** between user-defined limits  
✅ Support for **reset and enable control**  
✅ **Derivative on measurement** (not error) to reduce setpoint kick  
✅ Clear internal state tracking and comments

---

## 🧱 FUNCTION_BLOCK: `FB_PID_Controller`

### ✅ Purpose

Provide a robust, reusable, and configurable PID control algorithm for PLC-based control loops in IEC 61131-3 environments.

---

### 📋 Interface Definition

```pascal
FUNCTION_BLOCK FB_PID_Controller
VAR_INPUT
    Setpoint        : REAL; // Desired value
    ProcessVariable : REAL; // Measured process value
    Kp              : REAL; // Proportional gain
    Ki              : REAL; // Integral gain
    Kd              : REAL; // Derivative gain
    Enable          : BOOL; // Enables the PID output
    Reset           : BOOL; // Resets internal integrator and previous values
    MinOutput       : REAL; // Minimum output limit
    MaxOutput       : REAL; // Maximum output limit
    DeltaT          : REAL; // Time since last call (seconds)
END_VAR

VAR_OUTPUT
    ControlOutput   : REAL; // Final control output
    Error           : REAL; // Current error (for monitoring)
    IntegralTerm    : REAL; // Accumulated integral term
    DerivativeTerm  : REAL; // Calculated derivative term
END_VAR

VAR
    prevError         : REAL := 0.0;
    integrator        : REAL := 0.0;
    prevProcessVar    : REAL := 0.0;
    outputUnlimited   : REAL := 0.0;
    outputLimited     : REAL := 0.0;
    clampHi           : BOOL := FALSE;
    clampLo           : BOOL := FALSE;
END_VAR
```

---

### 🚀 Implementation Logic

```pascal
// PID Controller Function Block
// Implements standard PID with anti-windup, clamping, and reset logic

// Reset internal states if requested
IF Reset THEN
    prevError := 0.0;
    integrator := 0.0;
    prevProcessVar := ProcessVariable;
    ControlOutput := 0.0;
    Error := 0.0;
    IntegralTerm := 0.0;
    DerivativeTerm := 0.0;
    EXIT; // Skip calculation if reset
END_IF;

// Only perform PID calculation if enabled
IF Enable THEN

    // Calculate current error
    Error := Setpoint - ProcessVariable;

    // Compute integrator term with anti-windup via clamping
    integrator := integrator + Ki * Error * DeltaT;

    // Clamp integrator to avoid windup
    IF integrator > MaxOutput THEN
        integrator := MaxOutput;
    ELSIF integrator < MinOutput THEN
        integrator := MinOutput;
    END_IF;

    // Compute derivative term using change in process variable (not error)
    DerivativeTerm := Kd * (prevProcessVar - ProcessVariable) / DeltaT;

    // Save previous process variable for next derivative calculation
    prevProcessVar := ProcessVariable;

    // Compute full output before clamping
    outputUnlimited := Kp * Error + integrator + DerivativeTerm;

    // Apply output limits
    IF outputUnlimited > MaxOutput THEN
        outputLimited := MaxOutput;
        clampHi := TRUE;
        clampLo := FALSE;
    ELSIF outputUnlimited < MinOutput THEN
        outputLimited := MinOutput;
        clampHi := FALSE;
        clampLo := TRUE;
    ELSE
        outputLimited := outputUnlimited;
        clampHi := FALSE;
        clampLo := FALSE;
    END_IF;

    // Assign final output
    ControlOutput := outputLimited;

    // Store terms for output visibility
    IntegralTerm := integrator;
    DerivativeTerm := DerivativeTerm;

ELSE
    // If disabled, freeze outputs and retain state
    // Optional: set output to 0 or hold last value depending on application
    ControlOutput := 0.0;
END_IF;
```

---

### 🛠 Example Usage in a PLC Program

```pascal
PROGRAM PLC_PRG
VAR
    MyPID       : FB_PID_Controller;
    SP          : REAL := 100.0; // Setpoint
    PV          : REAL := 85.0;  // Process Variable
    CtrlOut     : REAL;
    Err         : REAL;
    IntegTerm   : REAL;
    DerivTerm   : REAL;
    SampleTime  : REAL := 0.1;   // 100 ms
    ResetPID    : BOOL := FALSE;
END_VAR

MyPID(
    Setpoint := SP,
    ProcessVariable := PV,
    Kp := 2.0,
    Ki := 0.5,
    Kd := 0.1,
    Enable := TRUE,
    Reset := ResetPID,
    MinOutput := 0.0,
    MaxOutput := 100.0,
    DeltaT := SampleTime,
    ControlOutput => CtrlOut,
    Error => Err,
    IntegralTerm => IntegTerm,
    DerivativeTerm => DerivTerm
);

// Use CtrlOut to drive valve, motor, heater, etc.
```

---

### 📌 Key Features & Benefits

| Feature | Description |
|--------|-------------|
| **Scan-Cycle Safe** | Designed to be called once per scan cycle |
| **Anti-Windup Logic** | Prevents integrator saturation using clamping |
| **Derivative on Measurement** | Reduces "derivative kick" from setpoint changes |
| **Output Clamping** | Ensures output stays within safe physical bounds |
| **Reset Capability** | Useful for startup, mode switching, or tuning |
| **Modular Design** | Reusable across different control loops |

---

### ⚠️ Notes on Practical Use

| Consideration | Recommendation |
|--------------|----------------|
| **Sample Time (`DeltaT`)** | Must be consistent and accurate; use system clock or fixed task interval |
| **Integral Windup** | Monitor clamp flags if further diagnostics are needed |
| **Derivative Filtering** | Add low-pass filter on derivative term if noisy sensor data |
| **Tuning** | Use Ziegler-Nichols method or auto-tune logic if available |
| **Nonlinear Systems** | Consider adaptive PID or gain scheduling for nonlinear processes |

---

### 📥 Optional Extensions

Would you like this function block extended to include:

✅ **Auto-tune mode** using relay feedback or step response  
✅ **Derivative filtering** (e.g., first-order LPF on derivative term)  
✅ **Gain scheduling** based on operating region  
✅ **Integral freezing** during output saturation  
✅ **Logging of historical values** for tuning or diagnostics  

Let me know how you'd like to enhance it!
