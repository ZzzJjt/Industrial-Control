PROGRAM SteelProductionShutdownSequence
VAR
    // State management variables
    currentState : INT := 0; // 0: Idle, 1: Initiate Shutdown, 2: Furnace Cooldown, 3: Gas Flow Reduction, 4: Oxygen Level Management, 5: Safety Checks, 6: Final Shutdown

    // Parameters for Furnace Cooldown Phase
    TargetCooldownTemperature : REAL := 100.0; // Target temperature in Celsius
    CooldownRate : REAL := 1.0; // Rate of temperature decrease per minute in Celsius

    // Parameters for Gas Flow Reduction Phase
    InitialFuelFlow : REAL := 100.0; // Initial fuel gas flow rate in m³/min
    FinalFuelFlow : REAL := 0.0; // Final fuel gas flow rate in m³/min
    TotalReductionTime : TIME := T#12h; // Total time for reduction in hours
    ReductionInterval : TIME := T#15m; // Interval for checking fuel flow reduction

    // Parameters for Oxygen Level Management Phase
    FuelAirRatioTarget : REAL := 2.5; // Target fuel-to-air ratio
    MonitoringInterval : TIME := T#5m; // Interval for monitoring and adjusting oxygen supply

    // Timers
    CooldownTimer : TON;
    ReductionTimer : TON;
    MonitoringTimer : TON;
    SafetyCheckTimer : TON;

    // Flags
    CooldownComplete : BOOL := FALSE;
    ReductionComplete : BOOL := FALSE;
    MonitoringComplete : BOOL := FALSE;
    SafetyChecksPassed : BOOL := FALSE;

    // Interlocks
    EmergencyStop : BOOL := FALSE; // Emergency stop button status
    TemperatureInterlock : BOOL := FALSE; // Temperature interlock status
    FlowInterlock : BOOL := FALSE; // Flow interlock status

    // Sensors
    CurrentTemperature : REAL := 800.0; // Current temperature sensor value in Celsius
    CurrentFuelFlow : REAL := InitialFuelFlow; // Current fuel gas flow rate in m³/min
    CurrentOxygenConcentration : REAL := 21.0; // Current oxygen concentration in %
END_VAR

// Function block declarations
FUNCTION_BLOCK RampDownFuelFlow
VAR_INPUT
    InitialFlow : REAL;
    FinalFlow : REAL;
    TotalTime : TIME;
    Interval : TIME;
END_VAR

VAR_OUTPUT
    ReducedFlow : REAL;
    Complete : BOOL;
END_VAR

VAR
    StepCount : INT := 0;
    StepsPerInterval : INT := 0;
    StepSize : REAL := 0.0;
    ReductionTimer : TON;
END_VAR

METHOD Execute(this : REFERENCE TO RampDownFuelFlow) : BOOL
BEGIN
    IF NOT this.ReductionTimer.Q THEN
        this.ReductionTimer(IN := TRUE, PT := this.Interval);
    END_IF;

    IF this.ReductionTimer.Q THEN
        this.StepCount := this.StepCount + 1;
        this.ReducedFlow := this.InitialFlow - (this.StepSize * this.StepCount);

        IF this.ReducedFlow <= this.FinalFlow THEN
            this.ReducedFlow := this.FinalFlow;
            this.Complete := TRUE;
        END_IF;

        this.ReductionTimer(IN := FALSE);
    END_IF;

    RETURN this.Complete;
END_METHOD

FUNCTION_BLOCK ManageOxygenSupply
VAR_INPUT
    CurrentFuelFlow : REAL;
    RatioTarget : REAL;
    MonitoringInterval : TIME;
END_VAR

VAR_OUTPUT
    AdjustedOxygenFlow : REAL;
END_VAR

VAR
    MonitoringTimer : TON;
END_VAR

METHOD Execute(this : REFERENCE TO ManageOxygenSupply) : BOOL
BEGIN
    IF NOT this.MonitoringTimer.Q THEN
        this.MonitoringTimer(IN := TRUE, PT := this.MonitoringInterval);
    END_IF;

    IF this.MonitoringTimer.Q THEN
        this.AdjustedOxygenFlow := (this.CurrentFuelFlow / this.RatioTarget); // Calculate adjusted oxygen flow based on target ratio
        this.MonitoringTimer(IN := FALSE);
    END_IF;

    RETURN TRUE;
END_METHOD

// Main control loop
CASE currentState OF
    0: // Idle state
        IF StartShutdown AND NOT EmergencyStop THEN
            currentState := 1; // Transition to Initiate Shutdown state
        END_IF;

    1: // Initiate Shutdown state
        currentState := 2; // Transition to Furnace Cooldown state

    2: // Furnace Cooldown state
        CASE currentState OF
            2: // Starting Furnace Cooldown sub-state
                WHILE CurrentTemperature > TargetCooldownTemperature DO
                    CurrentTemperature := CurrentTemperature - CooldownRate;
                    IF CurrentTemperature <= TargetCooldownTemperature THEN
                        CooldownComplete := TRUE;
                        currentState := 3; // Transition to Gas Flow Reduction state
                        EXIT;
                    END_IF;
                END_WHILE;

            ELSE:
                currentState := 2; // Reset to Starting Furnace Cooldown sub-state in case of unexpected state
        END_CASE;

    3: // Gas Flow Reduction state
        VAR
            RampDownBlock : RampDownFuelFlow;
        END_VAR

        RampDownBlock.InitialFlow := InitialFuelFlow;
        RampDownBlock.FinalFlow := FinalFuelFlow;
        RampDownBlock.TotalTime := TotalReductionTime;
        RampDownBlock.Interval := ReductionInterval;

        IF NOT ReductionTimer.Q THEN
            ReductionTimer(IN := TRUE, PT := ReductionInterval);
        END_IF;

        IF ReductionTimer.Q THEN
            RampDownBlock.Execute();
            CurrentFuelFlow := RampDownBlock.ReducedFlow;
            ReductionTimer(IN := FALSE);

            IF RampDownBlock.Complete THEN
                ReductionComplete := TRUE;
                currentState := 4; // Transition to Oxygen Level Management state
            END_IF;
        END_IF;

    4: // Oxygen Level Management state
        VAR
            OxygenManagementBlock : ManageOxygenSupply;
        END_VAR

        OxygenManagementBlock.CurrentFuelFlow := CurrentFuelFlow;
        OxygenManagementBlock.RatioTarget := FuelAirRatioTarget;
        OxygenManagementBlock.MonitoringInterval := MonitoringInterval;

        IF NOT MonitoringTimer.Q THEN
            MonitoringTimer(IN := TRUE, PT := MonitoringInterval);
        END_IF;

        IF MonitoringTimer.Q THEN
            OxygenManagementBlock.Execute();
            CurrentOxygenConcentration := OxygenManagementBlock.AdjustedOxygenFlow;
            MonitoringTimer(IN := FALSE);
        END_IF;

    5: // Safety Checks state
        IF NOT SafetyCheckTimer.Q THEN
            SafetyCheckTimer(IN := TRUE, PT := T#1h);
        END_IF;

        IF SafetyCheckTimer.Q THEN
            IF CurrentTemperature <= TargetCooldownTemperature AND CurrentFuelFlow <= FinalFuelFlow AND CurrentOxygenConcentration <= 21.0 THEN
                SafetyChecksPassed := TRUE;
                currentState := 6; // Transition to Final Shutdown state
            END_IF;
            SafetyCheckTimer(IN := FALSE);
        END_IF;

    6: // Final Shutdown state
        // All checks passed, perform final shutdown procedures
        currentState := 0; // Transition back to Idle state

    ELSE:
        currentState := 0; // Reset to Idle state in case of unexpected state
END_CASE;

// In-line comments explaining transitions and control logic
// - The program starts in the Idle state and transitions to Initiate Shutdown when StartShutdown is triggered and there is no emergency stop.
// - During Initiate Shutdown, the program immediately transitions to Furnace Cooldown.
// - During Furnace Cooldown, the reactor temperature is gradually reduced to the target cooldown temperature (100°C). If the target temperature is reached, it transitions to Gas Flow Reduction.
// - During Gas Flow Reduction, the fuel gas flow is ramped down from the initial flow rate to zero over 12 hours with regular checks every 15 minutes. If the reduction is complete, it transitions to Oxygen Level Management.
// - During Oxygen Level Management, the oxygen supply is continuously adjusted to maintain a 1:2.5 fuel-to-air ratio. Adjustments are made every 5 minutes. Once adjustments are stable, it transitions to Safety Checks.
// - During Safety Checks, various safety conditions are verified every hour. If all conditions are met, it transitions to Final Shutdown.
// - Once all checks are passed, the program performs final shutdown procedures and transitions back to the Idle state.
