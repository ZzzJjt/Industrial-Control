PROGRAM TrafficLightControl
VAR
    // Inputs (1-scan memory flags from sensors)
    M1, M2, M3, M4 : BOOL;       // Movement flags from sensors (1 scan pulse)
    // Passage occupancy memory
    M20 : BOOL := FALSE;         // TRUE when car is in passage from ground floor
    M30 : BOOL := FALSE;         // TRUE when car is in passage from basement
    // Outputs
    Y1 : BOOL := FALSE;          // Red light (ON = Stop)
    Y2 : BOOL := TRUE;           // Green light (ON = Proceed)
END_VAR

// --- Car Enters from Ground Floor ---
IF M1 OR M4 THEN
    M20 := TRUE;
END_IF;

// --- Car Enters from Basement ---
IF M2 OR M3 THEN
    M30 := TRUE;
END_IF;

// --- Car Exits from Ground Floor Side ---
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

// --- Car Exits from Basement Side ---
IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;

// --- Traffic Light Control Logic ---
IF M20 OR M30 THEN
    Y1 := TRUE;     // Red ON – passage occupied
    Y2 := FALSE;    // Green OFF
ELSE
    Y1 := FALSE;    // Red OFF – safe to enter
    Y2 := TRUE;     // Green ON
END_IF;
