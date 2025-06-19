FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[1..64] OF REAL; // Input signal array of size 64
END_VAR

VAR_OUTPUT
    MagnitudeSpectrum : ARRAY[1..32] OF REAL; // Output magnitude spectrum (half-length)
    RealComponents : ARRAY[1..64] OF REAL;     // Output real components of the FFT
    ImaginaryComponents : ARRAY[1..64] OF REAL; // Output imaginary components of the FFT
    Success : BOOL; // Flag indicating successful computation
    Error : STRING[50]; // Error message in case of failure
END_VAR

VAR
    N : INT := 64; // Size of the input array
    Log2N : INT := 6; // Log base 2 of N (since 2^6 = 64)
    i, j, k, n2, m, mmax, istep : INT;
    theta, wtemp, wr, wi, t1, t2, u1, u2 : REAL;
    tempReal, tempImag : ARRAY[1..64] OF REAL;
    BitReverseIndex : ARRAY[1..64] OF INT;
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs
    FOR i := 1 TO 64 DO
        RealComponents[i] := InputSignal[i];
        ImaginaryComponents[i] := 0.0;
    END_FOR;

    FOR i := 1 TO 32 DO
        MagnitudeSpectrum[i] := 0.0;
    END_FOR;

    Success := FALSE;
    Error := '';

    // Generate bit-reversed index array
    j := 1;
    FOR i := 1 TO 64 DO
        IF i < j THEN
            tempReal[j] := RealComponents[i];
            tempImag[j] := ImaginaryComponents[i];
            RealComponents[i] := tempReal[j];
            ImaginaryComponents[i] := tempImag[j];
        END_IF;
        m := N DIV 2;
        WHILE m < j DO
            j := j - m;
            m := m DIV 2;
        END_WHILE;
        j := j + m;
    END_FOR;

    // Perform the FFT using Cooley-Tukey radix-2 method
    mmax := 2;
    WHILE mmax <= N DO
        istep := mmax * 2;
        theta := -2.0 * PI / mmax;
        wtemp := SIN(0.5 * theta);
        wr := 1.0 - 2.0 * wtemp * wtemp;
        wi := -2.0 * SIN(theta / 2.0) * COS(theta / 2.0);
        FOR m := 1 TO mmax STEP 2 DO
            FOR i := m TO N STEP istep DO
                j := i + mmax;
                t1 := wr * RealComponents[j] - wi * ImaginaryComponents[j];
                t2 := wr * ImaginaryComponents[j] + wi * RealComponents[j];
                u1 := RealComponents[i];
                u2 := ImaginaryComponents[i];
                RealComponents[i] := u1 + t1;
                ImaginaryComponents[i] := u2 + t2;
                RealComponents[j] := u1 - t1;
                ImaginaryComponents[j] := u2 - t2;
            END_FOR;
            wr := wr * t1 - wi * t2 + wr;
            wi := wi * t1 + wr * t2 + wi;
        END_FOR;
        mmax := istep;
    END_FOR;

    // Calculate magnitude spectrum (only first half since it's symmetric)
    FOR i := 1 TO 32 DO
        MagnitudeSpectrum[i] := SQRT(SQR(RealComponents[i]) + SQR(ImaginaryComponents[i]));
    END_FOR;

    Success := TRUE;
    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
