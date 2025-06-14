PROGRAM PVCProductionControl
VAR
    // Inputs
    StartButton : BOOL; // Start button input
    StopButton : BOOL;  // Stop button input
    EmergencyStop : BOOL; // Emergency stop input

    // Outputs
    VacuumPumpActive : BOOL; // Vacuum pump output
    ValveDemineralizedWater : BOOL; // Valve for demineralized water
    ValveSurfactants : BOOL; // Valve for surfactants
    ValveVCM : BOOL; // Valve for VCM
    ValveCatalyst : BOOL; // Valve for catalyst
    HeaterActive : BOOL; // Heater output
    CoolerActive : BOOL; // Cooler output
    AgitatorSpeed : INT; // Agitator speed output

    // Internal Variables
    CurrentPhase : INT := 0; // Phase tracker: 0 = Idle, 1 = Polymerize, 2 = Decover, 3 = Dry
    Running : BOOL := FALSE; // Machine running status
    EmergencyStopped : BOOL := FALSE; // Emergency stop status

    // Process Parameters
    TargetTemperatureMin : REAL := 55.0; // Minimum target temperature for polymerization
    TargetTemperatureMax : REAL := 60.0; // Maximum target temperature for polymerization
    TargetAgitatorSpeed : INT := 200; // Target agitator speed
    PressureSetpoint : REAL := 1.0; // Pressure setpoint for monitoring

    // Timers
    TimerEvacuation : TON; // Timer for evacuation phase
    TimerCharging : TON; // Timer for charging phase
    TimerPolymerization : TON; // Timer for polymerization phase
    TimerDecovering : TON; // Timer for decovering phase
    TimerDrying : TON; // Timer for drying phase

    // Sensors
    CurrentTemperature : REAL := 0.0; // Current temperature reading from sensor
    CurrentPressure : REAL := 0.0; // Current pressure reading from sensor

    // Methods
    PROCEDURE EvacuateReactor();
    PROCEDURE AddDemineralizedWater();
    PROCEDURE AddSurfactants();
    PROCEDURE AddVCMAndCatalyst();
    PROCEDURE MaintainTemperature();
    PROCEDURE DecoverSystem();
    PROCEDURE DryMaterial();
END_VAR

// Initialize variables
VacuumPumpActive := FALSE;
ValveDemineralizedWater := FALSE;
ValveSurfactants := FALSE;
ValveVCM := FALSE;
ValveCatalyst := FALSE;
HeaterActive := FALSE;
CoolerActive := FALSE;
AgitatorSpeed := 0;

// Main control loop
IF EmergencyStop THEN
    EmergencyStopped := TRUE;
    Running := FALSE;
    CurrentPhase := 0; // Reset phase
ELSE
    IF StartButton AND NOT Running AND NOT EmergencyStopped THEN
        Running := TRUE;
        CurrentPhase := 1; // Initiate Polymerize phase
    ELSIF StopButton AND Running THEN
        Running := FALSE;
        CurrentPhase := 0; // Reset phase
    END_IF;
END_IF;

// Phase Control Logic
IF Running THEN
    CASE CurrentPhase OF
        1: // Polymerize phase
            EvacuateReactor();
            IF TimerEvacuation.Q THEN
                AddDemineralizedWater();
                IF TimerCharging.Q THEN
                    AddSurfactants();
                    IF TimerCharging.Q THEN
                        AddVCMAndCatalyst();
                        IF TimerCharging.Q THEN
                            MaintainTemperature();
                            IF TimerPolymerization.Q THEN
                                CurrentPhase := 2; // Transition to Decover phase
                            END_IF;
                        END_IF;
                    END_IF;
                END_IF;
            END_IF;
        2: // Decover phase
            DecoverSystem();
            IF TimerDecovering.Q THEN
                CurrentPhase := 3; // Transition to Dry phase
            END_IF;
        3: // Dry phase
            DryMaterial();
            IF TimerDrying.Q THEN
                Running := FALSE; // End of process
                CurrentPhase := 0; // Reset phase
            END_IF;
        ELSE
            CurrentPhase := 0; // Reset phase
    END_CASE;
END_IF;

// Emergency Stop Handling
IF EmergencyStopped THEN
    // Ensure all components are deactivated
    VacuumPumpActive := FALSE;
    ValveDemineralizedWater := FALSE;
    ValveSurfactants := FALSE;
    ValveVCM := FALSE;
    ValveCatalyst := FALSE;
    HeaterActive := FALSE;
    CoolerActive := FALSE;
    AgitatorSpeed := 0;
    Running := FALSE;
    EmergencyStopped := FALSE; // Reset emergency stop flag after handling
END_IF;

// Modular Methods
PROCEDURE EvacuateReactor()
    VacuumPumpActive := TRUE;
    TimerEvacuation(IN := TRUE, PT := T#30m); // Evacuate for 30 minutes
END_PROCEDURE

PROCEDURE AddDemineralizedWater()
    ValveDemineralizedWater := TRUE;
    TimerCharging(IN := TRUE, PT := T#10m); // Charge for 10 minutes
    IF TimerCharging.Q THEN
        ValveDemineralizedWater := FALSE;
    END_IF;
END_PROCEDURE

PROCEDURE AddSurfactants()
    ValveSurfactants := TRUE;
    TimerCharging(IN := TRUE, PT := T#10m); // Charge for 10 minutes
    IF TimerCharging.Q THEN
        ValveSurfactants := FALSE;
    END_IF;
END_PROCEDURE

PROCEDURE AddVCMAndCatalyst()
    ValveVCM := TRUE;
    ValveCatalyst := TRUE;
    TimerCharging(IN := TRUE, PT := T#10m); // Charge for 10 minutes
    IF TimerCharging.Q THEN
        ValveVCM := FALSE;
        ValveCatalyst := FALSE;
    END_IF;
END_PROCEDURE

PROCEDURE MaintainTemperature()
    HeaterActive := TRUE;
    AgitatorSpeed := TargetAgitatorSpeed;
    TimerPolymerization(IN := TRUE, PT := T#2h); // Polymerize for 2 hours
    // Simulate temperature control
    IF CurrentTemperature < TargetTemperatureMin THEN
        HeaterActive := TRUE;
        CoolerActive := FALSE;
    ELSIF CurrentTemperature > TargetTemperatureMax THEN
        HeaterActive := FALSE;
        CoolerActive := TRUE;
    ELSE
        HeaterActive := FALSE;
        CoolerActive := FALSE;
    END_IF;
    // Monitor pressure drop
    IF CurrentPressure > PressureSetpoint THEN
        // Handle pressure issue
        Running := FALSE; // Stop process due to excessive pressure
    END_IF;
END_PROCEDURE

PROCEDURE DecoverSystem()
    AgitatorSpeed := 0;
    TimerDecovering(IN := TRUE, PT := T#15m); // Decover for 15 minutes
END_PROCEDURE

PROCEDURE DryMaterial()
    HeaterActive := TRUE;
    AgitatorSpeed := TargetAgitatorSpeed;
    TimerDrying(IN := TRUE, PT := T#1h); // Dry for 1 hour
    // Simulate temperature control
    IF CurrentTemperature < TargetTemperatureMin THEN
        HeaterActive := TRUE;
        CoolerActive := FALSE;
    ELSIF CurrentTemperature > TargetTemperatureMax THEN
        HeaterActive := FALSE;
        CoolerActive := TRUE;
    ELSE
        HeaterActive := FALSE;
        CoolerActive := FALSE;
    END_IF;
END_PROCEDURE



