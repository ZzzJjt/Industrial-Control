```
PROGRAM CarParkControl
VAR
    // Sensors
    X1 : BOOL; // Photoelectric switch at the ground floor entry/exit
    X2 : BOOL; // Photoelectric switch at the basement entry/exit
    
    // Intermediate variables
    M1 : BOOL := FALSE; // ON for one scan cycle when a car from the ground floor passes X1
    M2 : BOOL := FALSE; // ON for one scan cycle when a car from the basement passes X1
    M3 : BOOL := FALSE; // ON for one scan cycle when a car from the basement passes X2
    M4 : BOOL := FALSE; // ON for one scan cycle when a car from the ground floor passes X2
    M20 : BOOL := FALSE; // ON during the process of a car entering from the ground floor
    M30 : BOOL := FALSE; // ON during the process of a car entering from the basement
    
    // Output devices
    Y1 : BOOL := FALSE; // Red lights at the entry/exit of the ground floor and the basement
    Y2 : BOOL := TRUE; // Green lights at the entry/exit of the ground floor and the basement
END_VAR

// Main control logic
IF X1 AND NOT M1 THEN
    // A car is entering from the ground floor
    M1 := TRUE;
    M20 := TRUE;
    Y1 := TRUE;
    Y2 := FALSE;
ELSIF X2 AND NOT M4 THEN
    // A car is exiting to the ground floor
    M4 := TRUE;
    M20 := FALSE;
    Y1 := FALSE;
    Y2 := TRUE;
ELSIF X1 AND NOT M3 THEN
    // A car is exiting to the basement
    M3 := TRUE;
    M30 := FALSE;
    Y1 := FALSE;
    Y2 := TRUE;
ELSIF X2 AND NOT M2 THEN
    // A car is entering from the basement
    M2 := TRUE;
    M30 := TRUE;
    Y1 := TRUE;
    Y2 := FALSE;
ELSIF M1 AND NOT X1 THEN
    // A car has passed through from the ground floor
    M1 := FALSE;
    M20 := FALSE;
    Y1 := FALSE;
    Y2 := TRUE;
ELSIF M2 AND NOT X1 THEN
    // A car has passed through from the basement
    M2 := FALSE;
    M30 := FALSE;
    Y1 := FALSE;
    Y2 := TRUE;
ELSIF M3 AND NOT X2 THEN
    // A car has passed through to the basement
    M3 := FALSE;
    M30 := FALSE;
    Y1 := FALSE;
    Y2 := TRUE;
ELSIF M4 AND NOT X2 THEN
    // A car has passed through to the ground floor
    M4 := FALSE;
    M20 := FALSE;
    Y1 := FALSE;
    Y2 := TRUE;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("X1", X1);
// Example: Write("X2", X2);
// Example: Write("M1", M1);
// Example: Write("M2", M2);
// Example: Write("M3", M3);
// Example: Write("M4", M4);
// Example: Write("M20", M20);
// Example: Write("M30", M30);
// Example: Write("Y1", Y1);
// Example: Write("Y2", Y2);

END_PROGRAM
```
