FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[1..64] OF REAL; (* Input time-domain signal, 64 samples *)
END_VAR
VAR_OUTPUT
    MagnitudeSpectrum : ARRAY[1..64] OF REAL; (* Output magnitude spectrum *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR
VAR
    (* Working arrays for real and imaginary components *)
    RealPart : ARRAY[1..64] OF REAL;
    ImagPart : ARRAY[1..64] OF REAL;
    
    (* Precomputed twiddle factors (cos and sin for N=64) *)
    TwiddleCos : ARRAY[0..31] OF REAL; (* Cosine values for twiddle factors *)
    TwiddleSin : ARRAY[0..31] OF REAL; (* Sine values for twiddle factors *)
    
    (* Loop variables *)
    i, j, k, m, stage, step, idx1, idx2 : INT;
    tempReal, tempImag, angle : REAL;
    
    (* Constants *)
    N : INT := 64; (* FFT size *)
    Log2N : INT := 6; (* log2(64) = 6 stages *)
    PI : REAL := 3.141592653589793;
END_VAR

(* Initialize outputs *)
FOR i := 1 TO 64 DO
    MagnitudeSpectrum[i] := 0.0;
END_FOR;

(* Validate input *)
FOR i := 1 TO 64 DO
    IF NOT IS_VALID_REAL(InputSignal[i]) THEN
        ErrorCode := 1; (* Invalid input *)
        RETURN;
    END_IF;
    RealPart[i] := InputSignal[i]; (* Copy input to working array *)
    ImagPart[i] := 0.0; (* Initialize imaginary part to zero *)
END_FOR;

(* Precompute twiddle factors *)
FOR k := 0 TO 31 DO
    angle := -2.0 * PI * INT_TO_REAL(k) / INT_TO_REAL(N);
    TwiddleCos[k] := COS(angle);
    TwiddleSin[k] := SIN(angle);
END_FOR;

(* Bit-reversal reordering *)
FOR i := 1 TO N-1 DO
    j := 0;
    m := i;
    FOR k := 0 TO Log2N-1 DO
        j := j * 2 + (m MOD 2);
        m := m / 2;
    END_FOR;
    IF i < j THEN
        (* Swap RealPart[i] and RealPart[j], same for ImagPart *)
        tempReal := RealPart[i];
        tempImag := ImagPart[i];
        RealPart[i] := RealPart[j];
        ImagPart[i] := ImagPart[j];
        RealPart[j] := tempReal;
        ImagPart[j] := tempImag;
    END_IF;
END_FOR;

(* Cooley-Tukey radix-2 FFT *)
FOR stage := 1 TO Log2N DO
    step := 2 ** stage; (* Distance between butterfly pairs *)
    FOR k := 1 TO N BY step DO
        FOR j := 0 TO (step / 2) - 1 DO
            idx1 := k + j; (* Index of first element in butterfly *)
            idx2 := idx1 + (step / 2); (* Index of second element *)
            
            (* Butterfly computation *)
            tempReal := RealPart[idx2] * TwiddleCos[j * (N / step)] - ImagPart[idx2] * TwiddleSin[j * (N / step)];
            tempImag := RealPart[idx2] * TwiddleSin[j * (N / step)] + ImagPart[idx2] * TwiddleCos[j * (N / step)];
            
            (* Update values *)
            RealPart[idx2] := RealPart[idx1] - tempReal;
            ImagPart[idx2] := ImagPart[idx1] - tempImag;
            RealPart[idx1] := RealPart[idx1] + tempReal;
            ImagPart[idx1] := ImagPart[idx1] + tempImag;
        END_FOR;
    END_FOR;
END_FOR;

(* Compute magnitude spectrum *)
FOR i := 1 TO N DO
    MagnitudeSpectrum[i] := SQRT(RealPart[i] * RealPart[i] + ImagPart[i] * ImagPart[i]) / INT_TO_REAL(N);
END_FOR;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
