// === Input Sensors and Flags ===
VAR
    X1, X2 : BOOL;           // Photoelectric sensors (detect presence)
    M1, M2, M3, M4 : BOOL;   // Scan flags (detect direction & location)
END_VAR

// === Internal State Flags ===
VAR
    M20 : BOOL := FALSE;     // Passage occupancy flag from ground floor
    M30 : BOOL := FALSE;     // Passage occupancy flag from basement
END_VAR

// === Output Traffic Lights ===
VAR
    Y1 : BOOL := FALSE;      // Red light (ON = stop)
    Y2 : BOOL := TRUE;       // Green light (ON = go)
END_VAR

// === Entry Detection ===
// Ground floor entry (M1 or M4): entering the passage
IF M1 OR M4 THEN
    M20 := TRUE;
END_IF;

// Basement entry (M2 or M3): entering the passage
IF M2 OR M3 THEN
    M30 := TRUE;
END_IF;

// === Exit Detection ===
// Ground floor exit (M3 or M4): passage clear from ground side
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

// Basement exit (M1 or M2): passage clear from basement side
IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;

// === Traffic Light Control ===
// If either side is using the passage, show RED
IF M20 OR M30 THEN
    Y1 := TRUE;   // Red ON
    Y2 := FALSE;  // Green OFF
ELSE
    Y1 := FALSE;  // Red OFF
    Y2 := TRUE;   // Green ON
END_IF;
