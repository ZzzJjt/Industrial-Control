```
PROGRAM CocoaMilkProduction
VAR
    // States for the batch process
    state: INT := 0;
    
    // Timers
    timerHeat: TON;
    timerBlend: TON;
    
    // Process parameters
    heatTemp: REAL := 60.0; // °C
    blendTime: TIME := T#5m; // Blending time
    
    // Flags
    isHeatingComplete: BOOL := FALSE;
    isBlendingComplete: BOOL := FALSE;
    
    // Temperature sensor
    mixTemp: REAL;
    
    // Actuators
    heater: BOOL;
    stirrer: BOOL;
    
    // Flow meters
    milkFlow: REAL;
    waterFlow: REAL;
    sugarFlow: REAL;
    cocoaFlow: REAL;
    
    // Required quantities
    milkQty: REAL := 80.0; // kg
    waterQty: REAL := 15.0; // kg
    sugarQty: REAL := 4.0; // kg
    cocoaQty: REAL := 1.0; // kg
END_VAR

METHOD ChargeIngredients: BOOL (milkQty: REAL, waterQty: REAL)
    milkFlow := milkQty;
    waterFlow := waterQty;
    WHILE milkFlow > 0 OR waterFlow > 0 DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    RETURN TRUE;
END_METHOD

METHOD HeatMixture: BOOL (targetTemp: REAL)
    heater := TRUE;
    WHILE mixTemp < targetTemp DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    heater := FALSE;
    RETURN TRUE;
END_METHOD

METHOD AddSweetener: BOOL (sugarQty: REAL)
    sugarFlow := sugarQty;
    WHILE sugarFlow > 0 DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    RETURN TRUE;
END_METHOD

METHOD AddCocoa: BOOL (cocoaQty: REAL)
    cocoaFlow := cocoaQty;
    WHILE cocoaFlow > 0 DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    RETURN TRUE;
END_METHOD

METHOD BlendMixture: BOOL (blendDuration: TIME)
    stirrer := TRUE;
    WHILE timerBlend.Q = FALSE DO
        WAIT 1; // Wait for 1 second
    END_WHILE;
    stirrer := FALSE;
    RETURN TRUE;
END_METHOD

(* Main control logic *)
CASE state OF
    0: (* Start the process *)
        state := 1;
        
    1: (* Charge ingredients *)
        IF NOT ChargeIngredients(milkQty, waterQty) THEN
            state := 2;
        END_IF;
        
    2: (* Heat mixture *)
        IF NOT isHeatingComplete THEN
            IF HeatMixture(heatTemp) THEN
                isHeatingComplete := TRUE;
                state := 3;
            END_IF;
        END_IF;
        
    3: (* Add sweetener *)
        IF NOT AddSweetener(sugarQty) THEN
            state := 4;
        END_IF;
        
    4: (* Add cocoa *)
        IF NOT AddCocoa(cocoaQty) THEN
            state := 5;
        END_IF;
        
    5: (* Blend mixture *)
        timerBlend(IN:=TRUE, PT:=blendTime);
        IF NOT isBlendingComplete THEN
            IF BlendMixture(blendTime) THEN
                isBlendingComplete := TRUE;
                state := 6;
            END_IF;
        END_IF;
        
    6: (* Cool and finish *)
        state := 0; // Complete and return to start
END_CASE;

(* Additional logic can be added here for monitoring and alarms *)

END_PROGRAM
```
