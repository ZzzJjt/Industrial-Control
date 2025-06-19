IF TT_101 > 240.0 THEN
    Melt_Pump := FALSE;
    Feed_Valve := FALSE;
    Alarm_OverTemp := TRUE;
END_IF;

IF PT_102 > 2.5 THEN
    Relief_Valve := TRUE;
    Melt_Pump := FALSE;
    Heater := FALSE;
    Alarm_OverPressure := TRUE;
END_IF;

IF FT_201 < 0.85 * NominalAirFlow THEN
    Melt_Pump := FALSE;
    PrillFan := FALSE;
    Alarm_CoolingLoss := TRUE;
END_IF;

IF E_STOP THEN
    Melt_Pump := FALSE;
    Heater := FALSE;
    Feed_Valve := FALSE;
    PrillFan := FALSE;
    Relief_Valve := TRUE;
    Alarm_ESTOP := TRUE;
END_IF;
