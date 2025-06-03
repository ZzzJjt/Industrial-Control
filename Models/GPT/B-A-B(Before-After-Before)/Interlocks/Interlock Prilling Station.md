FUNCTION_BLOCK FB_PrillingInterlockSystem
VAR_INPUT
    Temp_Tower           : REAL;  // Tower temperature (°C)
    Pressure_Process     : REAL;  // Process pressure (bar)
    Flow_CoolingAir      : REAL;  // Cooling air flow (m³/h)
    Level_Bucket         : REAL;  // Prill bucket level (%)
    MeltPump_OK          : BOOL;  // TRUE = pump operational
    Scrubber_OK          : BOOL;  // TRUE = scrubber operational
    PowerSupply_OK       : BOOL;  // TRUE = power is on
    InertGas_OK          : BOOL;  // TRUE = N2/inert gas available
    ExplosionVent        : BOOL;  // TRUE = explosion vent triggered
    E_STOP               : BOOL;  // Manual emergency stop
END_VAR

VAR_OUTPUT
    Shutdown_Tower       : BOOL;  // Shut down prill tower
    Stop_MeltFeed        : BOOL;  // Stop feeding melt to prilling head
    Alarm                : BOOL;  // Global alarm trigger
    CauseCode            : INT;   // Error code for diagnostics
END_VAR

VAR
    Latch_Estop          : BOOL := FALSE;
    Latch_Explosion      : BOOL := FALSE;
    Latch_Overtemp       : BOOL := FALSE;
    Latch_Overpress      : BOOL := FALSE;
    Latch_LowFlow        : BOOL := FALSE;
    Latch_HighLevel      : BOOL := FALSE;
    Latch_PumpFail       : BOOL := FALSE;
    Latch_ScrubberFail   : BOOL := FALSE;
    Latch_PowerFail      : BOOL := FALSE;
    Latch_GasLoss        : BOOL := FALSE;
END_VAR

// --- Latching Conditions ---
Latch_Overtemp     := Latch_Overtemp OR (Temp_Tower > 220.0);            // Overtemperature
Latch_Overpress    := Latch_Overpress OR (Pressure_Process > 15.0);      // Overpressure
Latch_LowFlow      := Latch_LowFlow OR (Flow_CoolingAir < 5000.0);       // Insufficient cooling air
Latch_HighLevel    := Latch_HighLevel OR (Level_Bucket > 95.0);          // Bucket overfilled
Latch_PumpFail     := Latch_PumpFail OR (NOT MeltPump_OK);               // Melt pump failure
Latch_ScrubberFail := Latch_ScrubberFail OR (NOT Scrubber_OK);           // Scrubber malfunction
Latch_PowerFail    := Latch_PowerFail OR (NOT PowerSupply_OK);           // Power loss
Latch_GasLoss      := Latch_GasLoss OR (NOT InertGas_OK);                // Inert gas loss
Latch_Explosion    := Latch_Explosion OR ExplosionVent;                  // Explosion vent triggered
Latch_Estop        := Latch_Estop OR E_STOP;                             // Manual emergency stop

// --- Determine Shutdown Actions ---
Shutdown_Tower := Latch_Overtemp OR Latch_Overpress OR Latch_ScrubberFail OR 
                  Latch_PowerFail OR Latch_GasLoss OR Latch_Estop OR Latch_Explosion;

Stop_MeltFeed  := Latch_LowFlow OR Latch_PumpFail OR Shutdown_Tower;

// --- Global Alarm ---
Alarm := Shutdown_Tower OR Stop_MeltFeed;

// --- Diagnostic Cause Code (priority-based) ---
IF Latch_Estop         THEN CauseCode := 1;
ELSIF Latch_Explosion  THEN CauseCode := 2;
ELSIF Latch_Overtemp   THEN CauseCode := 3;
ELSIF Latch_Overpress  THEN CauseCode := 4;
ELSIF Latch_LowFlow    THEN CauseCode := 5;
ELSIF Latch_HighLevel  THEN CauseCode := 6;
ELSIF Latch_PumpFail   THEN CauseCode := 7;
ELSIF Latch_ScrubberFail THEN CauseCode := 8;
ELSIF Latch_PowerFail  THEN CauseCode := 9;
ELSIF Latch_GasLoss    THEN CauseCode := 10;
ELSE CauseCode := 0;
END_IF;
