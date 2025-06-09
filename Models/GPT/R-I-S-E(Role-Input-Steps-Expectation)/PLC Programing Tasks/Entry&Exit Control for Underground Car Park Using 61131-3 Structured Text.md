FUNCTION_BLOCK ParkingPassageControl
VAR_INPUT
    X1 : BOOL; // Photoelectric sensor at ground floor
    X2 : BOOL; // Photoelectric sensor at basement
    M1 : BOOL; // Car enters from ground floor via X1
    M2 : BOOL; // Car enters from basement via X1
    M3 : BOOL; // Car enters from basement via X2
    M4 : BOOL; // Car enters from ground floor via X2
END_VAR

VAR_OUTPUT
    Y1 : BOOL := FALSE; // Red light (stop)
    Y2 : BOOL := TRUE;  // Green light (go)
END_VAR

VAR
    M20 : BOOL := FALSE; // Passage occupied from ground floor
    M30 : BOOL := FALSE; // Passage occupied from basement
END_VAR

// === Detect passage entry ===
IF M1 OR M4 THEN
    M20 := TRUE; // Car entering from ground floor
END_IF;

IF M2 OR M3 THEN
    M30 := TRUE; // Car entering from basement
END_IF;

// === Detect passage exit ===
IF M3 OR M4 THEN
    M20 := FALSE; // Ground-origin car has exited
END_IF;

IF M1 OR M2 THEN
    M30 := FALSE; // Basement-origin car has exited
END_IF;

// === Traffic light control ===
IF M20 OR M30 THEN
    // Passage is occupied
    Y1 := TRUE;  // Red ON
    Y2 := FALSE; // Green OFF
ELSE
    // Passage is free
    Y1 := FALSE; // Red OFF
    Y2 := TRUE;  // Green ON
END_IF;
