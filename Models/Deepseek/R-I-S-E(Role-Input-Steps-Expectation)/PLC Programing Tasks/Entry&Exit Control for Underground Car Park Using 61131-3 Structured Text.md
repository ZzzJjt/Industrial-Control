FUNCTION_BLOCK FB_ParkingGaragePassageControl
VAR_INPUT
    // Input Sensors / Pulse Flags
    X1 : BOOL; // Ground floor photoelectric switch
    X2 : BOOL; // Basement photoelectric switch
    M1 : BOOL; // Car enters from ground through X1 (pulse)
    M2 : BOOL; // Car enters from basement through X1 (pulse)
    M3 : BOOL; // Car enters from basement through X2 (pulse)
    M4 : BOOL; // Car enters from ground through X2 (pulse)
END_VAR

VAR_OUTPUT
    // Traffic Lights
    Y1 : BOOL := FALSE; // Red light (blocked)
    Y2 : BOOL := TRUE;  // Green light (allowed)
END_VAR

VAR
    // Intermediate States
    M20 : BOOL := FALSE; // Passage occupied by car from ground
    M30 : BOOL := FALSE; // Passage occupied by car from basement
END_VAR

// --- STEP 1: Detect vehicle entry and set passage state ---
// Entry from ground floor
IF M1 OR M4 THEN
    M20 := TRUE;
END_IF;

// Entry from basement
IF M2 OR M3 THEN
    M30 := TRUE;
END_IF;

// --- STEP 2: Detect vehicle exit and reset passage state ---
// Exit to basement from ground
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

// Exit to ground from basement
IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;

// --- STEP 3: Control traffic lights based on passage status ---
IF M20 OR M30 THEN
    // Passage is occupied — block further entry
    Y1 := TRUE;
    Y2 := FALSE;
ELSE
    // Passage is free — allow entry
    Y1 := FALSE;
    Y2 := TRUE;
END_IF;

PROGRAM PLC_PRG
VAR
    GaragePassage : FB_ParkingGaragePassageControl;

    // Simulated inputs
    SensorX1 : BOOL := FALSE;
    SensorX2 : BOOL := FALSE;
    PulseM1 : BOOL := FALSE;
    PulseM2 : BOOL := FALSE;
    PulseM3 : BOOL := FALSE;
    PulseM4 : BOOL := FALSE;

    // Outputs
    RedLight : BOOL;
    GreenLight : BOOL;
END_VAR

// Call the function block
GaragePassage(
    X1 := SensorX1,
    X2 := SensorX2,
    M1 := PulseM1,
    M2 := PulseM2,
    M3 := PulseM3,
    M4 := PulseM4
);

// Map outputs
RedLight := GaragePassage.Y1;
GreenLight := GaragePassage.Y2;
