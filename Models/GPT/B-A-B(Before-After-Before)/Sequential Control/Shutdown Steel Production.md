VAR
    shutdownPhase      : INT := 4;

    tempCurrent        : REAL;       // °C
    gasFlowCurrent     : REAL;       // m³/h
    oxygenFlowCurrent  : REAL;       // m³/h

    tempSetpoint       : REAL := 200.0;
    gasFlowInitial     : REAL := 3000.0;  // example start
    shutdownStartTime  : TIME;
    timeNow            : TIME;

    gasRampFB          : GasRampDown;
    oxyControlFB       : OxygenBalancer;

    phaseTimer         : TON;
    shutdownComplete   : BOOL := FALSE;
END_VAR

FUNCTION_BLOCK GasRampDown
VAR_INPUT
    gasStart      : REAL;
    duration      : TIME;
    t0            : TIME;
    tNow          : TIME;
END_VAR
VAR_OUTPUT
    gasSetpoint   : REAL;
    done          : BOOL;
END_VAR
VAR
    elapsed       : TIME;
    rate          : REAL;
END_VAR

elapsed := tNow - t0;
IF elapsed >= duration THEN
    gasSetpoint := 0.0;
    done := TRUE;
ELSE
    rate := gasStart / REAL_TO_TIME(duration);
    gasSetpoint := gasStart - rate * REAL_TO_TIME(elapsed);
END_IF;

