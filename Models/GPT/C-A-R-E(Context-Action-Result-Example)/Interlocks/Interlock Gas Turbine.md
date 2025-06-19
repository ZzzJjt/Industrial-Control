(* Gas Turbine Interlock Logic *)

IF Exhaust_Temp > 650.0 THEN
    Trip_Turbine := TRUE;
    Alarm_OverTemp := TRUE;
END_IF;

IF Rotor_Speed > 1.05 * Rated_Speed THEN
    Trip_Turbine := TRUE;
    Alarm_Overspeed := TRUE;
END_IF;

IF Oil_Pressure < 1.5 THEN
    Trip_Turbine := TRUE;
    Alarm_LowOilPressure := TRUE;
END_IF;

IF Emergency_Stop = TRUE THEN
    Trip_Turbine := TRUE;
    Fuel_Valve := FALSE;
    Alarm_EmergencyStop := TRUE;
END_IF;
