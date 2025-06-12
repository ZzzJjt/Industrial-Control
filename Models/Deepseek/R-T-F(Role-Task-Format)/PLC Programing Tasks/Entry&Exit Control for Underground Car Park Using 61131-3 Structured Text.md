PROGRAM TrafficLightControl
VAR
    X1 : BOOL; // Photoelectric switch at ground floor (ON when a car passes)
    X2 : BOOL; // Photoelectric switch at basement
    M1 : BOOL; // ON for one scan cycle when a car passes X1 from ground floor to basement
    M2 : BOOL; // ON for one scan cycle when a car passes X2 from basement to ground floor
    M3 : BOOL; // ON for one scan cycle when a car passes X2 from basement to ground floor
    M4 : BOOL; // ON for one scan cycle when a car passes X1 from ground floor to basement
    M20 : BOOL; // TRUE if a car is in the passage from the ground floor
    M30 : BOOL; // TRUE if a car is in the passage from the basement
    Y1 : BOOL; // Red lights for both ends of the passage (ON = stop)
    Y2 : BOOL; // Green lights for both ends of the passage (ON = proceed)
END_VAR

// Initialize outputs on startup
Y1 := FALSE; // Red OFF
Y2 := TRUE;  // Green ON

// Memory flags for car entry
IF M1 OR M4 THEN
    M20 := TRUE;
END_IF;

IF M2 OR M3 THEN
    M30 := TRUE;
END_IF;

// Memory flags for car exit
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;

// Light control logic
IF M20 OR M30 THEN
    Y1 := TRUE;  // Red ON
    Y2 := FALSE; // Green OFF
ELSE
    Y1 := FALSE; // Red OFF
    Y2 := TRUE;  // Green ON
END_IF;
