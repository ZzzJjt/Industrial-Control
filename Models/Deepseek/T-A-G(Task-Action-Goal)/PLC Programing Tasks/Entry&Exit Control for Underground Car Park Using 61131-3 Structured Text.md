PROGRAM TrafficLightControl
VAR_INPUT
    X1 : BOOL; // Detects car passage at the ground floor
    X2 : BOOL; // Detects car passage at the basement
END_VAR

VAR_OUTPUT
    Y1 : BOOL; // Red light output
    Y2 : BOOL; // Green light output
END_VAR

VAR
    M1 : BOOL; // One-scan memory flag for X1 rising edge
    M2 : BOOL; // One-scan memory flag for X1 falling edge
    M3 : BOOL; // One-scan memory flag for X2 rising edge
    M4 : BOOL; // One-scan memory flag for X2 falling edge
    M20 : BOOL; // Set when a car is in the passage from the ground floor
    M30 : BOOL; // Set when a car is in the passage from the basement
END_VAR

// Initialize system on power-up
Y1 := FALSE;
Y2 := TRUE;

// Edge detection for X1
IF X1 AND NOT M1 THEN
    M1 := TRUE;
    M2 := FALSE;
ELSIF NOT X1 AND M1 THEN
    M1 := FALSE;
    M2 := TRUE;
END_IF;

// Edge detection for X2
IF X2 AND NOT M3 THEN
    M3 := TRUE;
    M4 := FALSE;
ELSIF NOT X2 AND M3 THEN
    M3 := FALSE;
    M4 := TRUE;
END_IF;

// Update memory flags based on sensor inputs
IF M1 OR M4 THEN
    M20 := TRUE; // Car from ground floor
END_IF;

IF M2 OR M3 THEN
    M30 := TRUE; // Car from basement
END_IF;

IF M3 OR M4 THEN
    M20 := FALSE; // Car from ground floor has exited
END_IF;

IF M1 OR M2 THEN
    M30 := FALSE; // Car from basement has exited
END_IF;

// Update output lights based on memory flags
IF M20 OR M30 THEN
    Y1 := TRUE;  // Red light ON
    Y2 := FALSE; // Green light OFF
ELSE
    Y1 := FALSE; // Red light OFF
    Y2 := TRUE;  // Green light ON
END_IF;

// Additional comments for clarity
// - On power-up, initialize the system with green lights enabled for free traffic flow.
// - Monitor sensors X1 and X2 for car passage.
// - Use one-scan memory flags (M1-M4) to detect rising and falling edges of X1 and X2.
// - Set memory flags M20 and M30 to indicate car presence in the passage from ground floor or basement.
// - Control red (Y1) and green (Y2) lights based on the presence of cars in the passage.
// - Ensure only one vehicle uses the passage at a time and prevent simultaneous entry from both directions.



