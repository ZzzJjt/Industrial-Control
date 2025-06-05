PROGRAM NormalizeAndFilterSignal
VAR
    raw_signal      : REAL := 23.7;      // Raw sensor input (e.g., temperature)
    min_input       : REAL := 0.0;       // Min expected sensor value
    max_input       : REAL := 100.0;     // Max expected sensor value

    normalized      : REAL;              // Scaled 0–1 value
    filtered        : REAL;              // Filtered output
    alpha           : REAL := 0.1;       // Smoothing factor for LPF

    low_threshold   : REAL := 0.2;       // Alarm low limit
    high_threshold  : REAL := 0.8;       // Alarm high limit

    alarm_low       : BOOL;              // Alarm: value too low
    alarm_high      : BOOL;              // Alarm: value too high
END_VAR

// Normalize the signal using SCALE function
normalized := SCALE(raw_signal, min_input, max_input, 0.0, 1.0);

// Apply a low-pass filter using LP1_T1 (OSCAT LPF block)
filtered := LP1_T1(normalized, alpha);

// Check for alarms
alarm_low := (filtered < low_threshold);
alarm_high := (filtered > high_threshold);
