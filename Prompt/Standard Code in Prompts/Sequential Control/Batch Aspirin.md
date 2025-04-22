```
PROGRAM AspirinProduction
VAR
    // States for the batch process
    state: INT := 0;
    
    // Timers
    timerHeat: TON;
    timerMix: TON;
    timerReact: TON;
    
    // Process parameters
    heatTemp: REAL := 70.0; // °C
    mixTime: TIME := T#10m; // Mixing time
    reactTime: TIME := T#30m; // Reaction time
    coolTemp: REAL := 20.0; // °C
    dryTemp: REAL := 90.0; // °C
    dryTime: TIME := T#1h; // Drying time
    
    // Flags
    isHeatingComplete: BOOL := FALSE;
    isMixingComplete: BOOL := FALSE;
    isReactingComplete: BOOL := FALSE;
    isCoolingComplete: BOOL := FALSE;
    isDryingComplete: BOOL := FALSE;
    
    // Temperature sensors
    reactorTemp: REAL;
    crystallizerTemp: REAL;
    dryerTemp: REAL;
    
    // Actuators
    reactorHeater: BOOL;
    reactorCooler: BOOL;
    dryerHeater: BOOL;
END_VAR

METHOD HeatReactor: BOOL (targetTemp: REAL)
    reactorHeater := TRUE;
    WHILE reactorTemp < targetTemp DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    reactorHeater := FALSE;
    RETURN TRUE;
END_METHOD

METHOD MixSolution: BOOL (mixDuration: TIME)
    WHILE timerMix.Q = FALSE DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    RETURN TRUE;
END_METHOD

METHOD ReactSolution: BOOL (reactDuration: TIME)
    WHILE timerReact.Q = FALSE DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    RETURN TRUE;
END_METHOD

METHOD CoolSolution: BOOL (targetTemp: REAL)
    reactorCooler := TRUE;
    WHILE crystallizerTemp > targetTemp DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    reactorCooler := FALSE;
    RETURN TRUE;
END_METHOD

METHOD DryCrystals: BOOL (targetTemp: REAL, duration: TIME)
    dryerHeater := TRUE;
    WHILE timerDry.Q = FALSE DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    dryerHeater := FALSE;
    RETURN TRUE;
END_METHOD

(* Main control logic *)
CASE state OF
    0: (* Start the reaction process *)
        state := 1;
        
    1: (* Heat reactor *)
        IF NOT isHeatingComplete THEN
            IF HeatReactor(heatTemp) THEN
                isHeatingComplete := TRUE;
                state := 2;
            END_IF;
        END_IF;
        
    2: (* Mix solution *)
        timerMix(IN:=TRUE, PT:=mixTime);
        IF timerMix.Q THEN
            isMixingComplete := TRUE;
            state := 3;
        END_IF;
        
    3: (* React solution *)
        timerReact(IN:=TRUE, PT:=reactTime);
        IF timerReact.Q THEN
            isReactingComplete := TRUE;
            state := 4;
        END_IF;
        
    4: (* Cool solution *)
        IF NOT isCoolingComplete THEN
            IF CoolSolution(coolTemp) THEN
                isCoolingComplete := TRUE;
                state := 5;
            END_IF;
        END_IF;
        
    5: (* Dry crystals *)
        timerDry(IN:=TRUE, PT:=dryTime);
        IF NOT isDryingComplete THEN
            IF DryCrystals(dryTemp, dryTime) THEN
                isDryingComplete := TRUE;
                state := 0; // Complete and return to start
            END_IF;
        END_IF;
END_CASE;

(* Additional logic can be added here for monitoring and alarms *)

END_PROGRAM
```
