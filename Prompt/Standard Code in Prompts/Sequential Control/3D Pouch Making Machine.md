```
PROGRAM InitializeMachine
VAR
    // Initial Parameters
    initHeatingTemp : REAL := 120.0; // °C
    initCoolingTemp : REAL := 25.0; // °C
    initFeederSpeed : REAL := 0.0; // m/min
END_VAR

METHOD InitializeParameters : BOOL
    FOR i := 1 TO 8 DO
        SetHeatingStationTemp(i, initHeatingTemp);
        SetCoolingStationTemp(i, initCoolingTemp);
    END_FOR;
    SetFeederUnitSpeed(1, initFeederSpeed);
    SetFeederUnitSpeed(2, initFeederSpeed);
    MoveHorizontalCutterToHomePosition();
    MoveVerticalCutterToHomePosition();
    RETURN TRUE;
END_METHOD

METHOD StartFeeders : BOOL (targetSpeed : REAL)
    FOR i := 1 TO 2 DO
        rampUpSpeed := RampFunction(0, targetSpeed, 10); // Ramp up over 10 seconds
        SetFeederUnitSpeed(i, rampUpSpeed);
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD HeatStations : BOOL (targetTemp : REAL)
    FOR i := 1 TO 8 DO
        SetHeatingStationTemp(i, targetTemp);
        WHILE NOT IsHeatingStationAtTemp(i, targetTemp) DO
            WAIT 1; // Wait for 1 second
        END_WHILE;
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD CoolStations : BOOL (targetTemp : REAL)
    FOR i := 1 TO 8 DO
        SetCoolingStationTemp(i, targetTemp);
        WHILE NOT IsCoolingStationAtTemp(i, targetTemp) DO
            WAIT 1; // Wait for 1 second
        END_WHILE;
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD ActivateCutters : BOOL
    MoveHorizontalCutterToOperationalPosition();
    MoveVerticalCutterToOperationalPosition();
    ActivateHorizontalCutter();
    ActivateVerticalCutter();
    RETURN TRUE;
END_METHOD

METHOD StartupRoutine : BOOL
    InitializeParameters();
    StartFeeders(50.0); // Target speed of 50 m/min
    HeatStations(120.0); // Heating temperature of 120°C
    CoolStations(25.0); // Cooling temperature of 25°C
    ActivateCutters();
    RETURN TRUE;
END_METHOD

METHOD DeactivateCutters : BOOL
    DeactivateHorizontalCutter();
    DeactivateVerticalCutter();
    MoveHorizontalCutterToHomePosition();
    MoveVerticalCutterToHomePosition();
    RETURN TRUE;
END_METHOD

METHOD StopFeeders : BOOL
    FOR i := 1 TO 2 DO
        rampDownSpeed := RampFunction(50.0, 0.0, 10); // Ramp down over 10 seconds
        SetFeederUnitSpeed(i, rampDownSpeed);
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD CoolDownHeatingStations : BOOL (coolingTemp : REAL)
    FOR i := 1 TO 8 DO
        SetHeatingStationTemp(i, coolingTemp);
        WHILE NOT IsHeatingStationAtTemp(i, coolingTemp) DO
            WAIT 1; // Wait for 1 second
        END_WHILE;
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD CoolDownCoolingStations : BOOL (roomTemp : REAL)
    FOR i := 1 TO 8 DO
        SetCoolingStationTemp(i, roomTemp);
        WHILE NOT IsCoolingStationAtTemp(i, roomTemp) DO
            WAIT 1; // Wait for 1 second
        END_WHILE;
    END_FOR;
    RETURN TRUE;
END_METHOD

METHOD ShutdownRoutine : BOOL
    DeactivateCutters();
    StopFeeders();
    CoolDownHeatingStations(25.0); // Cooling temperature of 25°C
    CoolDownCoolingStations(25.0); // Room temperature of 25°C
    RETURN TRUE;
END_METHOD
```
