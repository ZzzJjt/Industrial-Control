PROGRAM PassageControl
VAR
    // Sensor Inputs (Photoelectric switches)
    X1 : BOOL; // Ground floor sensor
    X2 : BOOL; // Basement sensor

    // Pulse flags (1 scan triggers)
    M1, M2, M3, M4 : BOOL;

    // Memory flags (passage occupancy)
    M20 : BOOL := FALSE; // Passage occupied from ground floor
    M30 : BOOL := FALSE; // Passage occupied from basement

    // Light Outputs
    Y1 : BOOL := FALSE; // Red light
    Y2 : BOOL := TRUE;  // Green light
END_VAR

// === Passage Occupancy Detection ===
// Car entering from ground floor
IF M1 OR M4 THEN
    M20 := TRUE;
END_IF;

// Car entering from basement
IF M2 OR M3 THEN
    M30 := TRUE;
END_IF;

// Car exits — passage clear from ground floor direction
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

// Car exits — passage clear from basement direction
IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;

// === Light Control Logic ===
IF M20 OR M30 THEN
    // Passage is occupied — show RED lights at both ends
    Y1 := TRUE;
    Y2 := FALSE;
ELSE
    // Passage is clear — show GREEN lights
    Y1 := FALSE;
    Y2 := TRUE;
END_IF;
