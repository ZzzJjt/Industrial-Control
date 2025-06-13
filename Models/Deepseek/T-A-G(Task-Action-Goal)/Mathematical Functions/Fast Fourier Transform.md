FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputArray : ARRAY[1..64] OF REAL; // Input array of 64 samples
END_VAR

VAR_OUTPUT
    RealPart : ARRAY[1..64] OF REAL;   // Real part of the FFT result
    ImagPart : ARRAY[1..64] OF REAL;   // Imaginary part of the FFT result
    Converged : BOOL;                 // Indicates if the computation converged
    Error : BOOL;                     // Indicates if an error occurred
    ErrorMessage : STRING;             // Detailed error message
END_VAR

VAR
    N : INT := 64;                    // Fixed size of the input array
    M : INT := 6;                     // Log2(N), where N = 2^M
    TwiddleFactorsReal : ARRAY[1..64] OF REAL;
    TwiddleFactorsImag : ARRAY[1..64] OF REAL;
    i, j, k, n1, n2, a : INT;
    temp_real, temp_imag : REAL;
    angle_step : REAL;
    c, s : REAL;
    u_real, u_imag : REAL;
    t_real, t_imag : REAL;
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs
    FOR i := 1 TO N DO
        RealPart[i] := 0.0;
        ImagPart[i] := 0.0;
    END_FOR;
    Converged := FALSE;
    Error := FALSE;
    ErrorMessage := '';

    // Copy input array to RealPart
    FOR i := 1 TO N DO
        RealPart[i] := InputArray[i];
        ImagPart[i] := 0.0;
    END_FOR;

    // Bit-reversal permutation
    j := 1;
    FOR i := 1 TO N - 1 DO
        IF j > i THEN
            temp_real := RealPart[j];
            temp_imag := ImagPart[j];
            RealPart[j] := RealPart[i];
            ImagPart[j] := ImagPart[i];
            RealPart[i] := temp_real;
            ImagPart[i] := temp_imag;
        END_IF;
        k := N DIV 2;
        WHILE k < j DO
            j := j - k;
            k := k DIV 2;
        END_WILOOPS;
        j := j + k;
    END_FOR;

    // Precompute twiddle factors
    angle_step := -2.0 * #PI / N;
    FOR i := 1 TO N DO
        TwiddleFactorsReal[i] := COS((i - 1) * angle_step);
        TwiddleFactorsImag[i] := SIN((i - 1) * angle_step);
    END_FOR;

    // Butterfly computation
    n2 := 1;
    FOR a := 1 TO M DO
        n1 := n2;
        n2 := 2 * n1;
        angle_step := -#PI / n1;
        FOR j := 1 TO n1 DO
            c := COS((j - 1) * angle_step);
            s := SIN((j - 1) * angle_step);
            FOR k := j TO N STEP n2 DO
                i := k + n1;
                u_real := RealPart[k];
                u_imag := ImagPart[k];
                t_real := c * RealPart[i] - s * ImagPart[i];
                t_imag := s * RealPart[i] + c * ImagPart[i];
                RealPart[k] := u_real + t_real;
                ImagPart[k] := u_imag + t_imag;
                RealPart[i] := u_real - t_real;
                ImagPart[i] := u_imag - t_imag;
            END_FOR;
        END_FOR;
    END_FOR;

    Converged := TRUE;
    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



