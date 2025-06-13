PROGRAM GAS_TURBINE_INTERLOCKS
VAR_INPUT
    Execute : BOOL;          // Starts the interlock checks
    EGT : REAL;              // Exhaust Gas Temperature (°C)
    Turbine_Speed : REAL;    // Turbine Speed (RPM)
    Flame_Present : BOOL;    // Flame Presence Signal
    Lub_Oil_Pressure : REAL; // Lubrication Oil Pressure (bar)
    Cool_Water_Flow : REAL;  // Cooling Water Flow Rate (m³/h)
    Fuel_Supply_Pressure : REAL; // Fuel Supply Pressure (MPa)
    Gen_Stator_Temp : REAL;  // Generator Stator Temperature (°C)
    Vibration_Level : REAL;  // Vibration Level (mm/s RMS)
    Bearing_Temp : REAL;     // Bearing Temperature (°C)
    Emergency_Stop_Button : BOOL; // Manual Emergency Stop Button
END_VAR

VAR_OUTPUT
    Turbine_Shutdown : BOOL; // Status of Turbine Shutdown
    Fuel_Valve_Close : BOOL; // Status of Fuel Valve Closure
    Alarm_Triggered : BOOL;  // Status of Alarm
    Load_Reduction : BOOL;   // Status of Load Reduction
    Error : BOOL;            // General error flag
    ErrorID : DWORD;         // Detailed error code
END_VAR

VAR
    lastExecute : BOOL := FALSE; // Last state of the Execute input
END_VAR

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        // Reset outputs
        Turbine_Shutdown := FALSE;
        Fuel_Valve_Close := FALSE;
        Alarm_Triggered := FALSE;
        Load_Reduction := FALSE;
        Error := FALSE;
        ErrorID := 0;

        // Check for EGT overtemperature
        IF EGT > 650.0 THEN
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 1; // EGT overtemperature detected
        END_IF;

        // Check for overspeed
        IF Turbine_Speed > 3000.0 THEN
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 2; // Overspeed detected
        END_IF;

        // Check for flame failure
        IF NOT Flame_Present THEN
            Fuel_Valve_Close := TRUE;
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 3; // Flame failure detected
        END_IF;

        // Check for lubrication oil pressure low
        IF Lub_Oil_Pressure < 1.5 THEN
            Alarm_Triggered := TRUE;
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 4; // Lubrication oil pressure low
        END_IF;

        // Check for cooling water flow low
        IF Cool_Water_Flow < 10.0 THEN
            Alarm_Triggered := TRUE;
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 5; // Cooling water flow low
        END_IF;

        // Check for fuel supply pressure low
        IF Fuel_Supply_Pressure < 1.0 THEN
            Alarm_Triggered := TRUE;
            Fuel_Valve_Close := TRUE;
            Error := TRUE;
            ErrorID := 6; // Fuel supply pressure low
        END_IF;

        // Check for generator stator temperature high
        IF Gen_Stator_Temp > 120.0 THEN
            Alarm_Triggered := TRUE;
            Load_Reduction := TRUE;
            Error := TRUE;
            ErrorID := 7; // Generator stator temperature high
        END_IF;

        // Check for vibration exceedance
        IF Vibration_Level > 5.0 THEN
            Alarm_Triggered := TRUE;
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 8; // Vibration exceedance detected
        END_IF;

        // Check for bearing temperature high
        IF Bearing_Temp > 100.0 THEN
            Alarm_Triggered := TRUE;
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 9; // Bearing temperature high
        END_IF;

        // Check for emergency stop
        IF Emergency_Stop_Button THEN
            Turbine_Shutdown := TRUE;
            Error := TRUE;
            ErrorID := 10; // Emergency stop activated
        END_IF;
    END_IF;

    lastExecute := Execute;
    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



