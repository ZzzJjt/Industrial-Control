```
PROGRAM PneumaticSystemControl
VAR
    // Inputs
    PressureSensor : REAL; // Pressure sensor reading in bar
    FlowSensor : REAL; // Flow sensor reading in standard liters per minute (SLPM)
    
    // Outputs
    CompressorControl : BOOL := FALSE; // Control signal for compressor
    ValveControl : BOOL := FALSE; // Control signal for flow control valve
    
    // Intermediate Variables
    DesiredPressure : REAL := 5.75; // Desired pressure in bar (mid-point of range)
    DesiredFlowRate : REAL := 50.0; // Desired flow rate in SLPM
    PressureTolerance : REAL := 0.25; // Pressure tolerance band around the desired pressure
    FlowRateTolerance : REAL := 5.0; // Flow rate tolerance band around the desired flow rate
    ControlLoopFrequency : TIME := T#100ms; // Control loop frequency
    CompressorTimer : TIME := T#0s; // Timer for compressor control delay
    ValveTimer : TIME := T#0s; // Timer for valve control delay
    CompressorDelay : TIME := T#2s; // Delay before compressor activates
    ValveDelay : TIME := T#1s; // Delay before valve adjusts
END_VAR

// Main control logic executed every 100 ms
IF TCU(ControlLoopFrequency) THEN
    // Pressure Control
    IF PressureSensor < DesiredPressure - PressureTolerance THEN
        // Pressure is too low, activate compressor
        CompressorControl := TRUE;
        CompressorTimer := T#0s;
    ELSIF PressureSensor > DesiredPressure + PressureTolerance THEN
        // Pressure is too high, deactivate compressor
        CompressorControl := FALSE;
        CompressorTimer := T#0s;
    ELSE
        // Pressure is within tolerance, maintain current state
        CompressorControl := CompressorControl;
    END_IF;
    
    // Flow Rate Control
    IF FlowSensor < DesiredFlowRate - FlowRateTolerance THEN
        // Flow rate is too low, open valve more
        ValveControl := TRUE;
        ValveTimer := T#0s;
    ELSIF FlowSensor > DesiredFlowRate + FlowRateTolerance THEN
        // Flow rate is too high, close valve more
        ValveControl := FALSE;
        ValveTimer := T#0s;
    ELSE
        // Flow rate is within tolerance, maintain current state
        ValveControl := ValveControl;
    END_IF;
    
    // Safety Checks
    IF PressureSensor < 5.5 OR PressureSensor > 6.0 THEN
        // Pressure out of safe range, shut down system
        CompressorControl := FALSE;
        ValveControl := FALSE;
    END_IF;
    
    // Timers for delayed actions
    IF CompressorControl AND CompressorTimer < CompressorDelay THEN
        CompressorTimer := CompressorTimer + T#100ms;
    END_IF;
    
    IF ValveControl AND ValveTimer < ValveDelay THEN
        ValveTimer := ValveTimer + T#100ms;
    END_IF;
    
    // Debugging outputs (for simulation purposes)
    // Example: Write("Desired Pressure", DesiredPressure);
    // Example: Write("Desired Flow Rate", DesiredFlowRate);
    // Example: Write("Pressure Sensor", PressureSensor);
    // Example: Write("Flow Sensor", FlowSensor);
    // Example: Write("Compressor Control", CompressorControl);
    // Example: Write("Valve Control", ValveControl);
    // Example: Write("Compressor Timer", CompressorTimer);
    // Example: Write("Valve Timer", ValveTimer);
END_IF;

END_PROGRAM
```
