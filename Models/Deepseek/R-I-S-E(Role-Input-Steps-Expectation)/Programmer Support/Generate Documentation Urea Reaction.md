AmmoniaValveOpen := TRUE;
CO2ValveOpen := TRUE;

IF AmmoniaValveConfirmed AND CO2ValveConfirmed THEN
    ValvesOpen := TRUE;
END_IF;

IF (PressureSensor >= MinPressure AND PressureSensor <= MaxPressure) AND
   (TempSensor >= MinTemp AND TempSensor <= MaxTemp) THEN
    ConditionsStable := TRUE;
END_IF;
