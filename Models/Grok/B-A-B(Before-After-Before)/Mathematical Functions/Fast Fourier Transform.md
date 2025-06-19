(* Function Block: FFT_Block
   Purpose: Computes the Fast Fourier Transform (FFT) of a 64-point REAL input signal.
   Features:
   - Accepts InputSignal: ARRAY[1..64] OF REAL
   - Applies Hamming window to reduce spectral leakage
   - Uses radix-2 Cooley-Tukey algorithm for efficiency
   - Returns Amplitudes: ARRAY[1..32] OF REAL (magnitudes up to Nyquist frequency)
   - Optimized for PLCs: fixed 64-point length, precomputed twiddle factors, no recursion
   - Detailed comments for each step: windowing, bit-reversal, butterfly, magnitude
   Notes:
   - Assumes power-of-two input length (N=64) for radix-2 algorithm
   - Outputs amplitudes for frequencies 0 to Nyquist (N/2 points)
   - Designed for low-frequency industrial signals (e.g., vibration monitoring)
*)

FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[1..64] OF REAL;     (* Input signal, 64 samples *)
END_VAR
VAR_OUTPUT
    Amplitudes : ARRAY[1..32] OF REAL;      (* Output magnitudes, 0 to Nyquist *)
    ErrorFlag : BOOL;                       (* TRUE if computation fails *)
END_VAR
VAR
    (* Constants *)
    N : UINT := 64;                         (* Fixed FFT length *)
    LOG2_N : UINT := 6;                     (* log2(64) for stages *)
    PI : REAL := 3.141592653589793;         (* Pi constant *)
    
    (* Internal arrays *)
    RealPart : ARRAY[1..64] OF REAL;        (* Real part of complex signal *)
    ImagPart : ARRAY[1..64] OF REAL;        (* Imaginary part of complex signal *)
    Window : ARRAY[1..64] OF REAL;          (* Hamming window coefficients *)
    TwiddleReal : ARRAY[1..32] OF REAL;     (* Precomputed twiddle factors, real *)
    TwiddleImag : ARRAY[1..32] OF REAL;     (* Precomputed twiddle factors, imaginary *)
    
    (* Working variables *)
    i, j, k, m, stage, step, idx1, idx2 : UINT;
    TempReal, TempImag, Angle, WReal, WImag : REAL;
    BitReverseIdx : ARRAY[1..64] OF UINT;   (* Bit-reversal index mapping *)
    ValidComputation : BOOL;                (* Tracks computation validity *)
END_VAR

(* Initialize outputs and variables *)
FOR i := 1 TO 32 DO
    Amplitudes[i] := 0.0;
END_FOR
ErrorFlag := FALSE;
ValidComputation := TRUE;

(* Precompute Hamming window *)
FOR i := 1 TO N DO
    Window[i] := 0.54 - 0.46 * COS(2.0 * PI * REAL(i - 1) / REAL(N - 1));
END_FOR

(* Precompute twiddle factors for N/2 points *)
FOR i := 1 TO N/2 DO
    Angle := -2.0 * PI * REAL(i - 1) / REAL(N);
    TwiddleReal[i] := COS(Angle);
    TwiddleImag[i] := SIN(Angle);
END_FOR

(* Precompute bit-reversal indices *)
FOR i := 1 TO N DO
    j := 0;
    FOR k := 0 TO LOG2_N - 1 DO
        IF (i - 1) AND (1 SHL k) <> 0 THEN
            j := j OR (1 SHL (LOG2_N - 1 - k));
        END_IF
    END_FOR
    BitReverseIdx[i] := j + 1;
END_FOR

(* Apply Hamming window and initialize complex arrays *)
FOR i := 1 TO N DO
    IF ABS(InputSignal[i]) > 1.0E6 THEN
        ErrorFlag := TRUE;
        ValidComputation := FALSE;
        EXIT;
    END_IF
    RealPart[BitReverseIdx[i]] := InputSignal[i] * Window[i];
    ImagPart[BitReverseIdx[i]] := 0.0;
END_FOR

(* Perform radix-2 FFT if valid *)
IF ValidComputation THEN
    (* Iterate over stages *)
    FOR stage := 1 TO LOG2_N DO
        step := 1 SHL stage;    (* Step size for current stage *)
        FOR j := 1 TO N/2 BY step/2 DO
            (* Access precomputed twiddle factor *)
            WReal := TwiddleReal[j];
            WImag := TwiddleImag[j];
            
            (* Butterfly computations *)
            FOR k := j TO N STEP step DO
                idx1 := k;
                idx2 := k + step/2;
                
                (* Butterfly: X[idx1] = X[idx1] + W * X[idx2] *)
                TempReal := WReal * RealPart[idx2] - WImag * ImagPart[idx2];
                TempImag := WReal * ImagPart[idx2] + WImag * RealPart[idx2];
                
                (* Update X[idx2] = X[idx1] - W * X[idx2] *)
                RealPart[idx2] := RealPart[idx1] - TempReal;
                ImagPart[idx2] := ImagPart[idx1] - TempImag;
                
                (* Update X[idx1] = X[idx1] + W * X[idx2] *)
                RealPart[idx1] := RealPart[idx1] + TempReal;
                ImagPart[idx1] := ImagPart[idx1] + TempImag;
            END_FOR
        END_FOR
    END_FOR
    
    (* Compute magnitudes (amplitudes) up to Nyquist frequency *)
    FOR i := 1 TO N/2 DO
        Amplitudes[i] := SQRT(RealPart[i] * RealPart[i] + ImagPart[i] * ImagPart[i]) / REAL(N/2);
        IF ABS(Amplitudes[i]) > 1.0E6 THEN
            ErrorFlag := TRUE;
            ValidComputation := FALSE;
            EXIT;
        END_IF
    END_FOR
END_IF

(* Set ErrorFlag if computation failed *)
IF NOT ValidComputation THEN
    ErrorFlag := TRUE;
    FOR i := 1 TO 32 DO
        Amplitudes[i] := 0.0;
    END_FOR
END_IF

END_FUNCTION_BLOCK
