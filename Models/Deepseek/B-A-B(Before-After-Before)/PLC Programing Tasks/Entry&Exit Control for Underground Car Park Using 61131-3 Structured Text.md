PROGRAM PLC_PRG
TITLE 'Single-Lane Car Park Traffic Control'

(*
    Description:
    Controls traffic flow through a single-lane underground car park.
    
    Features:
    - Monitors photoelectric sensors at both ends (X1, X2)
    - Uses scan-cycle flags (M1–M4) to detect movement direction
    - Tracks current lane occupancy with M20 (ground entry) and M30 (basement entry)
    - Controls red (Y1) and green (Y2) traffic lights
    - Ensures only one vehicle occupies the passage at any time
    
    Safety:
    - Red light ON blocks both sides when passage is occupied
    - Green light ON only when passage is completely free
*)

VAR
    // Inputs: Sensors
    X1 : BOOL := FALSE;   // Entry sensor – Ground Floor
    X2 : BOOL := FALSE;   // Entry sensor – Basement

    // Internal Flags: Scan Pulse Flags (used to detect motion direction)
    M1 : BOOL := FALSE;   // Rising edge detected on X1
    M2 : BOOL := FALSE;   // Falling edge detected on X1
    M3 : BOOL := FALSE;   // Falling edge detected on X2
    M4 : BOOL := FALSE;   // Rising edge detected on X2

    // Internal Flags: Passage Occupancy
    M20 : BOOL := FALSE;  // Passage occupied by vehicle entering from Ground
    M30 : BOOL := FALSE;  // Passage occupied by vehicle entering from Basement

    // Outputs: Lights
    Y1 : BOOL := FALSE;   // Red Light – Passage Occupied
    Y2 : BOOL := TRUE;    // Green Light – Passage Free
END_VAR

// === MAIN LOGIC ===

// Step 1: Detect vehicle entry/exit using edge transitions

// Ground Floor side (X1)
IF X1 THEN
    M1 := NOT M1;  // Toggle M1 on rising edge of X1
ELSE
    M2 := NOT M2;  // Toggle M2 on falling edge of X1
END_IF;

// Basement side (X2)
IF X2 THEN
    M4 := NOT M4;  // Toggle M4 on rising edge of X2
ELSE
    M3 := NOT M3;  // Toggle M3 on falling edge of X2
END_IF;

// Step 2: Set passage occupancy flags
// A car enters from Ground if M1 or M4 is triggered
IF M1 OR M4 THEN
    M20 := TRUE;
END_IF;

// A car enters from Basement if M2 or M3 is triggered
IF M2 OR M3 THEN
    M30 := TRUE;
END_IF;

// Step 3: Clear passage occupancy flags when vehicle exits
// Vehicle exits toward Basement if M3 or M4 is triggered
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

// Vehicle exits toward Ground if M1 or M2 is triggered
IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;

// Step 4: Control traffic lights based on passage status
IF M20 OR M30 THEN
    Y1 := TRUE;   // Red Light ON – passage is occupied
    Y2 := FALSE;  // Green Light OFF
ELSE
    Y1 := FALSE;  // Red Light OFF – passage is free
    Y2 := TRUE;   // Green Light ON
END_IF;

END_PROGRAM
