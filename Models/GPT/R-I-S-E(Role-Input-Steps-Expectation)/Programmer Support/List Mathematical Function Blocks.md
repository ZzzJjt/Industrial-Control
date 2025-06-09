VAR
    Angle_Deg     : REAL := 30.0;       // Flow angle in degrees
    Angle_Rad     : REAL;               // Converted angle in radians
    Flow_Magnitude: REAL := 100.0;      // Magnitude of the flow vector
    Flow_X        : REAL;               // X component of flow
    Flow_Y        : REAL;               // Y component of flow

    // Safety limits
    Limit_Min     : REAL := -120.0;
    Limit_Max     : REAL := 120.0;
END_VAR

// Convert degrees to radians
Angle_Rad := DEG_TO_RAD(Angle_Deg);

// Compute vector components
Flow_X := COS(Angle_Rad) * Flow_Magnitude;
Flow_Y := SIN(Angle_Rad) * Flow_Magnitude;

// Clamp output using OSCAT LIMIT function
Flow_X := LIMIT(Limit_Min, Flow_X, Limit_Max);
Flow_Y := LIMIT(Limit_Min, Flow_Y, Limit_Max);
