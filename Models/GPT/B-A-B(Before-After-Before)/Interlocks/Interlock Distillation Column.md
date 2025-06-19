                          +---------------------------+
                          |                           |
                          |        C-101              |
                          |  Distillation Column      |
                          |                           |
        Feed ------------>+                           +-------------> Distillate
            (FV-101)      |                           |
                          |                           |
                     [LT-101]                        [TT-101]
                 Level Transmitter             Top Temperature
                          |                           |
                          +------------+--------------+
                                       |
                                    [PT-101]
                                 Pressure Transmitter
                                       |
                                  PRV-101 (to flare)
                                       |
                           -------------------------------
                          |                               |
                          V                               V
                    E-101 (Reboiler)                E-102 (Condenser)
                          |                               |
                   Reboiler Supply Valve             Cooling Water




                   PROGRAM ColumnInterlocks
VAR_INPUT
    PT_101 : REAL;       // Column pressure in psi
    TT_101 : REAL;       // Column top temperature in deg C
    LT_101 : REAL;       // Column level in %
END_VAR

VAR_IN_OUT
    FV_101         : BOOL;   // Feed valve
    PRV_101        : BOOL;   // Pressure relief valve (digital trigger)
    REBOILER_VALVE : BOOL;   // Reboiler supply valve
    HI_LVL_ALARM   : BOOL;   // High level alarm
END_VAR

VAR CONSTANT
    PT_LOW_LIMIT   : REAL := 50.0;     // psi
    PT_HIGH_LIMIT  : REAL := 120.0;    // psi
    TT_HIGH_LIMIT  : REAL := 180.0;    // deg C
    LT_HIGH_LIMIT  : REAL := 90.0;     // %
END_VAR

// Safety Interlock Logic

// Pressure low: shut feed
IF PT_101 < PT_LOW_LIMIT THEN
    FV_101 := FALSE;
END_IF

// Pressure high: trigger PRV
IF PT_101 > PT_HIGH_LIMIT THEN
    PRV_101 := TRUE;
ELSE
    PRV_101 := FALSE;
END_IF

// Temperature high: shut reboiler steam supply
IF TT_101 > TT_HIGH_LIMIT THEN
    REBOILER_VALVE := FALSE;
END_IF

// High level: raise alarm
IF LT_101 > LT_HIGH_LIMIT THEN
    HI_LVL_ALARM := TRUE;
ELSE
    HI_LVL_ALARM := FALSE;
END_IF
