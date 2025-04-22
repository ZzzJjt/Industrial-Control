```
PROGRAM HeatExchangerCascadeControl
VAR
    // Inputs
    ProcessTemperature : REAL; // Temperature sensor reading in Celsius
    HeatingDemand : REAL; // Demand for heating from the process
    FlowRateSensor : REAL; // Flow rate sensor reading in liters/minute
    
    // Outputs
    TemperatureSetpoint : REAL := 50.0; // Outer loop setpoint in Celsius
    FlowRateControl : REAL := 50.0; // Inner loop output for flow control
    
    // Parameters
    KpTemp : REAL := 1.0; // Proportional gain for temperature control
    KiTemp : REAL := 0.1; // Integral gain for temperature control
    KdTemp : REAL := 0.01; // Derivative gain for temperature control
    KpFlow : REAL := 0.5; // Proportional gain for flow rate control
    KiFlow : REAL := 0.05; // Integral gain for flow rate control
    KdFlow : REAL := 0.005; // Derivative gain for flow rate control
    
    // Intermediate Variables
    ErrorTemp : REAL; // Temperature error
    IntegralTemp : REAL := 0.0; // Integral term for temperature control
    DerivativeTemp : REAL := 0.0; // Derivative term for temperature control
    ErrorFlow : REAL; // Flow rate error
    IntegralFlow : REAL := 0.0; // Integral term for flow rate control
    DerivativeFlow : REAL := 0.0; // Derivative term for flow rate control
    LastProcessTemperature : REAL := ProcessTemperature; // Previous process temperature for derivative calculation
    LastFlowRateSensor : REAL := FlowRateSensor; // Previous flow rate for derivative calculation
    SampleTime : TIME := T#1s; // Sample time for PID calculations
    
    // Timers
    PIDSampleTimer : TIME := T#0s; // Timer for sample interval
END_VAR

// Main Control Logic
IF TCU(SampleTime) THEN
    // Update last known values for derivative calculation
    LastProcessTemperature := ProcessTemperature;
    LastFlowRateSensor := FlowRateSensor;
    
    // Calculate temperature error
    ErrorTemp := TemperatureSetpoint - ProcessTemperature;
    
    // Calculate integral term for temperature control
    IntegralTemp := IntegralTemp + ErrorTemp * T#1s;
    
    // Calculate derivative term for temperature control
    DerivativeTemp := (ProcessTemperature - LastProcessTemperature) / T#1s;
    
    // Calculate flow rate setpoint based on temperature error
    FlowRateSetpoint := KpTemp * ErrorTemp + KiTemp * IntegralTemp + KdTemp * DerivativeTemp;
    
    // Calculate flow rate error
    ErrorFlow := FlowRateSetpoint - FlowRateSensor;
    
    // Calculate integral term for flow rate control
    IntegralFlow := IntegralFlow + ErrorFlow * T#1s;
    
    // Calculate derivative term for flow rate control
    DerivativeFlow := (FlowRateSensor - LastFlowRateSensor) / T#1s;
    
    // Calculate flow rate control output
    FlowRateControl := KpFlow * ErrorFlow + KiFlow * IntegralFlow + KdFlow * DerivativeFlow;
    
    // Anti-windup for integral terms
    IF ErrorTemp > 0 THEN
        IntegralTemp := MAX(IntegralTemp, 0);
    ELSE
        IntegralTemp := MIN(IntegralTemp, 0);
    END_IF;
    
    IF ErrorFlow > 0 THEN
        IntegralFlow := MAX(IntegralFlow, 0);
    ELSE
        IntegralFlow := MIN(IntegralFlow, 0);
    END_IF;
    
    // Limit the output of the flow rate control to a reasonable range
    FlowRateControl := MAX(FlowRateControl, 0);
    FlowRateControl := MIN(FlowRateControl, 100);
    
    // Reset timers
    PIDSampleTimer := T#0s;
ELSE
    PIDSampleTimer := PIDSampleTimer + T#1s;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("Temperature Setpoint", TemperatureSetpoint);
// Example: Write("Process Temperature", ProcessTemperature);
// Example: Write("Flow Rate Control Output", FlowRateControl);
// Example: Write("Flow Rate Sensor Reading", FlowRateSensor);
// Example: Write("Error Temp", ErrorTemp);
// Example: Write("Integral Temp", IntegralTemp);
// Example: Write("Derivative Temp", DerivativeTemp);
// Example: Write("Flow Rate Setpoint", FlowRateSetpoint);
// Example: Write("Error Flow", ErrorFlow);
// Example: Write("Integral Flow", IntegralFlow);
// Example: Write("Derivative Flow", DerivativeFlow);

END_PROGRAM
```
