FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[1..64] OF REAL;  // Real input signal
    Execute     : BOOL;                  // Trigger to start FFT
END_VAR
VAR_OUTPUT
    RealOut     : ARRAY[1..64] OF REAL;  // Real part of FFT output
    ImagOut     : ARRAY[1..64] OF REAL;  // Imaginary part
    Magnitude   : ARRAY[1..64] OF REAL;  // Magnitude spectrum
    Done        : BOOL;                  // Set TRUE when complete
    Error       : BOOL;                  // Set TRUE on failure
    ErrorID     : INT;                   // Diagnostic code
END_VAR
VAR
    TempReal    : ARRAY[1..64] OF REAL;
    TempImag    : ARRAY[1..64] OF REAL;
    N           : INT := 64;             // FFT size (must be power of 2)
    i, j, k, m  : INT;
    stage, stages, span, halfspan : INT;
    angle, tw_re, tw_im : REAL;
    ur, ui, vr, vi : REAL;
    bitrev     : ARRAY[1..64] OF INT;
    Pi         : REAL := 3.14159265;
    executing  : BOOL := FALSE;
END_VAR

// ---------------------------------------------
// Bit reversal function for reordering inputs
// ---------------------------------------------
FOR i := 0 TO N - 1 DO
    bitrev[i + 1] := 0;
    FOR j := 0 TO 5 DO // log2(64) = 6
        IF (i AND (1 << j)) <> 0 THEN
            bitrev[i + 1] := bitrev[i + 1] + (1 << (5 - j));
        END_IF
    END_FOR
END_FOR

// Start computation on rising edge
IF Execute AND NOT executing THEN
    executing := TRUE;
    Done := FALSE;
    Error := FALSE;

    // Load inputs into reordered arrays
    FOR i := 1 TO N DO
        TempReal[i] := InputSignal[bitrev[i] + 1];
        TempImag[i] := 0.0;
    END_FOR

    // Cooley-Tukey FFT
    stages := 6; // log2(64)
    FOR stage := 1 TO stages DO
        span := 1 << stage;
        halfspan := span / 2;

        FOR i := 0 TO (N - 1) BY span DO
            FOR j := 0 TO halfspan - 1 DO
                k := i + j + 1;
                m := k + halfspan;

                angle := -2.0 * Pi * j / span;
                tw_re := COS(angle);
                tw_im := SIN(angle);

                ur := TempReal[k];
                ui := TempImag[k];
                vr := TempReal[m] * tw_re - TempImag[m] * tw_im;
                vi := TempReal[m] * tw_im + TempImag[m] * tw_re;

                TempReal[k] := ur + vr;
                TempImag[k] := ui + vi;
                TempReal[m] := ur - vr;
                TempImag[m] := ui - vi;
            END_FOR
        END_FOR
    END_FOR

    // Output results
    FOR i := 1 TO N DO
        RealOut[i] := TempReal[i];
        ImagOut[i] := TempImag[i];
        Magnitude[i] := SQRT(TempReal[i]*TempReal[i] + TempImag[i]*TempImag[i]);
    END_FOR

    Done := TRUE;
    executing := FALSE;
END_IF
