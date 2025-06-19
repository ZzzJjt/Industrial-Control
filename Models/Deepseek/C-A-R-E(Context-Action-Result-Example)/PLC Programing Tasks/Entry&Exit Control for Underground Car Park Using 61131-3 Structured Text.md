PROGRAM CarParkTrafficControl
VAR_INPUT
    X1 : BOOL; // Photoelectric switch at ground floor (ON when car passes)
    X2 : BOOL; // Photoelectric switch at basement (ON when car passes)
END_VAR

VAR_MEMORY
    M20 : BOOL; // Passage occupied by a car entering from the ground floor
    M30 : BOOL; // Passage occupied by a car entering from the basement
END_VAR

VAR_OUTPUT
    Y1 : BOOL; // Red lights (ON = stop)
    Y2 : BOOL; // Green lights (ON = go)
END_VAR

VAR
    M1 : BOOL; // Pulse flag for X1 rising edge
    M2 : BOOL; // Pulse flag for X1 falling edge
    M3 : BOOL; // Pulse flag for X2 rising edge
    M4 : BOOL; // Pulse flag for X2 falling edge
    PrevX1 : BOOL; // Previous state of X1
    PrevX2 : BOOL; // Previous state of X2
END_VAR

// Detect rising and falling edges for X1 and X2
M1 := X1 AND NOT PrevX1; // Rising edge of X1
M2 := NOT X1 AND PrevX1; // Falling edge of X1
M3 := X2 AND NOT PrevX2; // Rising edge of X2
M4 := NOT X2 AND PrevX2; // Falling edge of X2

// Update previous states
PrevX1 := X1;
PrevX2 := X2;

// Set passage occupancy based on rising edges
IF M1 THEN
    M20 := TRUE; // Ground floor entry
END_IF;

IF M3 THEN
    M30 := TRUE; // Basement entry
END_IF;

// Clear passage occupancy based on falling edges
IF M2 THEN
    M20 := FALSE; // Ground floor exit
END_IF;

IF M4 THEN
    M30 := FALSE; // Basement exit
END_IF;

// Light control
IF M20 OR M30 THEN
    Y1 := TRUE;  // Red light ON
    Y2 := FALSE; // Green light OFF
ELSE
    Y1 := FALSE; // Red light OFF
    Y2 := TRUE;  // Green light ON
END_IF;

// Inline comments explaining the logic:
// The program uses photoelectric sensors to detect vehicle entry and exit.
// Memory flags (M20 and M30) track whether the passage is occupied by cars from either direction.
// Actuators (Y1 and Y2) control red and green lights to manage safe access.
// The system allows only one car at a time to use the passage and resets to a default "ready" state when the passage becomes empty.



