PROGRAM PRILLING_STATION_INTERLOCKS
VAR_INPUT
    Execute : BOOL;                      // Starts the interlock checks
    Temp_101 : REAL;                     // Temperature reading from sensor (°C)
    PT_101 : REAL;                       // Pressure reading from sensor (bar)
    Motor_Current_101 : REAL;             // Motor current reading (A)
    Air_Flow_101 : REAL;                 // Air flow reading (m³/min)
    Feed_Rate_101 : REAL;                // Feed rate reading (kg/min)
    Cool_Water_Flow_101 : REAL;           // Cooling water flow reading (m³/h)
    E_Stop_Button : BOOL;                // Emergency stop button input
END_VAR

VAR_OUTPUT
    Feed_System_Shutdown : BOOL;          // Status of feed system shutdown
    Emergency_Vent_Activated : BOOL;      // Status of emergency vent activation
    Relief_Valve_Open : BOOL;             // Status of relief valve open
    Equipment_Shutdown : BOOL;            // Status of equipment shutdown
    Backup_Fans_Activated : BOOL;         // Status of backup fans activation
    Reduce_Feed_Rate : BOOL;             // Status of reducing feed rate
    Backup_Cooling_Activated : BOOL;     // Status of backup cooling activation
    All_Systems_Shutdown : BOOL;        // Status of all systems shutdown
    Alarm_Triggered : BOOL;              // Status of alarm
    Error : BOOL;                         // General error flag
    ErrorID : DWORD;                      // Detailed error code
END_VAR

VAR
    lastExecute : BOOL := FALSE;          // Last state of the Execute input
    MAX_ALLOWED_CURRENT : REAL := 100.0;  // Maximum allowed motor current (A)
    MIN_SAFE_AIR_FLOW : REAL := 100.0;    // Minimum safe air flow (m³/min)
    MAX_ALLOWED_FEED_RATE : REAL := 1000.0;// Maximum allowed feed rate (kg/min)
    MIN_SAFE_COOL_WATER_FLOW : REAL := 10.0; // Minimum safe cooling water flow (m³/h)
END_VAR

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        // Reset outputs except latched states
        Feed_System_Shutdown := FALSE;
        Emergency_Vent_Activated := FALSE;
        Relief_Valve_Open := FALSE;
        Equipment_Shutdown := FALSE;
        Backup_Fans_Activated := FALSE;
        Reduce_Feed_Rate := FALSE;
        Backup_Cooling_Activated := FALSE;
        All_Systems_Shutdown := FALSE;
        Alarm_Triggered := FALSE;
        Error := FALSE;
        ErrorID := 0;

        // Check for overheating
        IF Temp_101 > 180.0 THEN
            Feed_System_Shutdown := TRUE;
            Emergency_Vent_Activated := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 1; // Overheating detected
        END_IF;

        // Check for overpressure
        IF PT_101 > 15.0 THEN
            Relief_Valve_Open := TRUE;
            Feed_System_Shutdown := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 2; // Overpressure detected
        END_IF;

        // Check for mechanical failure
        IF Motor_Current_101 > MAX_ALLOWED_CURRENT THEN
            Equipment_Shutdown := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 3; // Mechanical failure detected
        END_IF;

        // Check for ventilation loss
        IF Air_Flow_101 < MIN_SAFE_AIR_FLOW THEN
            Backup_Fans_Activated := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 4; // Ventilation loss detected
        END_IF;

        // Check for feed rate excess
        IF Feed_Rate_101 > MAX_ALLOWED_FEED_RATE THEN
            Reduce_Feed_Rate := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 5; // Feed rate excess detected
        END_IF;

        // Check for cooling system failure
        IF Cool_Water_Flow_101 < MIN_SAFE_COOL_WATER_FLOW THEN
            Backup_Cooling_Activated := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 6; // Cooling system failure detected
        END_IF;

        // Check for emergency stop
        IF E_Stop_Button THEN
            All_Systems_Shutdown := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 7; // Emergency stop activated
        END_IF;
    END_IF;

    lastExecute := Execute;
    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



