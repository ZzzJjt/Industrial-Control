```
PROGRAM UreaFertilizerBatchControl
VAR
    // State variables
    currentState: INT := 0; // Initial state
    
    // Timers
    timerHeating: TON;
    timerCooling: TON;
    timerPressureRegulation: TON;
    
    // Process parameters
    heatingTime: TIME := T#10m; // Heating time
    coolingTime: TIME := T#5m; // Cooling time
    pressureRegulationTime: TIME := T#3m; // Pressure regulation time
    reactionTemp: REAL := 180.0; // Reaction temperature (°C)
    maxPressure: REAL := 140.0; // Maximum allowed pressure (bar)
    
    // Flags
    isHeatingComplete: BOOL := FALSE;
    isCoolingComplete: BOOL := FALSE;
    isPressureRegulationComplete: BOOL := FALSE;
    
    // Sensors
    reactorTemp: REAL; // Reactor temperature sensor
    reactorPressure: REAL; // Reactor pressure sensor
    
    // Actuators
    heater: BOOL;
    cooler: BOOL;
    pressureValve: BOOL;
    
END_VAR

METHOD HeatReactor: BOOL (heatingTime: TIME, targetTemp: REAL)
    heater := TRUE;
    timerHeating(IN:=TRUE, PT:=heatingTime);
    WHILE timerHeating.Q = FALSE OR reactorTemp < targetTemp DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    heater := FALSE;
    isHeatingComplete := TRUE;
    RETURN TRUE;
END_METHOD

METHOD CoolReactor: BOOL (coolingTime: TIME, targetTemp: REAL)
    cooler := TRUE;
    timerCooling(IN:=TRUE, PT:=coolingTime);
    WHILE timerCooling.Q = FALSE OR reactorTemp > targetTemp DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    cooler := FALSE;
    isCoolingComplete := TRUE;
    RETURN TRUE;
END_METHOD

METHOD RegulatePressure: BOOL (pressureRegulationTime: TIME, maxPressure: REAL)
    pressureValve := TRUE;
    timerPressureRegulation(IN:=TRUE, PT:=pressureRegulationTime);
    WHILE timerPressureRegulation.Q = FALSE OR reactorPressure > maxPressure DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    pressureValve := FALSE;
    isPressureRegulationComplete := TRUE;
    RETURN TRUE;
END_METHOD

(* Main control logic *)
CASE currentState OF
    0: (* Initial state - Load materials *)
        currentState := 1; // Proceed to heating
        
    1: (* Heating phase *)
        IF NOT isHeatingComplete THEN
            IF HeatReactor(heatingTime, reactionTemp) THEN
                currentState := 2; // Proceed to pressure regulation
            END_IF;
        END_IF;
        
    2: (* Pressure regulation phase *)
        IF NOT isPressureRegulationComplete THEN
            IF RegulatePressure(pressureRegulationTime, maxPressure) THEN
                currentState := 3; // Proceed to cooling
            END_IF;
        END_IF;
        
    3: (* Cooling phase *)
        IF NOT isCoolingComplete THEN
            IF CoolReactor(coolingTime, 150.0) THEN // Target temp after reaction
                currentState := 4; // Offloading
            END_IF;
        END_IF;
        
    4: (* Offloading *)
        currentState := 0; // Return to initial state
END_CASE;

(* Additional logic can be added here for monitoring and alarms *)

END_PROGRAM
```
