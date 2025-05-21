(* Function Block: Fast Fourier Transform (FFT) for 64-point input *)
FUNCTION_BLOCK FB_FFT_Block
VAR_INPUT
    Execute : BOOL;                       (* Trigger FFT computation *)
    InputSignal : ARRAY[1..64] OF REAL;   (* Input time-domain signal, 64 samples *)
END_VAR

VAR_OUTPUT
    Magnitude : ARRAY[1..32] OF REAL;     (* Magnitude spectrum, 1 to N/2 bins *)
    Done : BOOL;                          (* Computation completed *)
    Error : BOOL;                         (* Error flag *)
    ErrorID : DWORD;                      (* Error code *)
END_VAR

VAR
    (* Constants *)
    N : UINT := 64;                       (* FFT size, power of 2 *)
    LOG2_N : UINT := 6;                   (* log2(N) = 6 for N=64 *)
    PI : REAL := 3.141592653589793;       (* Pi constant *)
    
    (* Working arrays *)
    RealPart : ARRAY[1..64] OF REAL;      (* Real part of complex signal *)
    ImagPart : ARRAY[1..64] OF REAL;      (* Imaginary part of complex signal *)
    TempReal : ARRAY[1..64] OF REAL;      (* Temporary real part for bit-reversal *)
    TempImag : ARRAY[1..64] OF REAL;      (* Temporary imaginary part for bit-reversal *)
    
    (* Twiddle factors (precomputed) *)
    CosTable : ARRAY[0..31] OF REAL;      (* Cosine values for twiddle factors *)
    SinTable : ARRAY[0..31] OF REAL;      (* Sine values for twiddle factors *)
    
    (* Internal variables *)
    i, j, k, m, n1, n2 : UINT;           (* Loop and index variables *)
    Stage : UINT;                         (* FFT stage counter *)
    Step : UINT;                          (* Step size for butterfly *)
    Angle : REAL;                         (* Angle for twiddle factors *)
    TempR, TempI : REAL;                  (* Temporary real/imaginary values *)
    State : UINT;                         (* State machine: 0=Idle, 1=Init, 2=BitReverse, 3=Butterfly, 4=ComputeMag *)
    Initialized : BOOL;                    (* Flag for twiddle table initialization *)
END_VAR

(* Initialize outputs *)
Done := FALSE;
Error := FALSE;
ErrorID := 0;
FOR i := 1 TO 32 DO
    Magnitude[i] := 0.0;
END_FOR;

(* Main logic *)
IF Execute THEN
    CASE State OF
        0: (* Idle *)
            (* Validate input *)
            FOR i := 1 TO N DO
                IF ABS(InputSignal[i]) > 1.0E10 THEN
                    Error := TRUE;
                    ErrorID := 16#80010000; (* Invalid input values *)
                    RETURN;
                END_IF;
            END_FOR;
            
            (* Initialize twiddle factors once *)
            IF NOT Initialized THEN
                FOR i := 0 TO N/2 - 1 DO
                    Angle := -2.0 * PI * REAL_OF_UINT(i) / REAL_OF_UINT(N);
                    CosTable[i] := COS(Angle);
                    SinTable[i] := SIN(Angle);
                END_FOR;
                Initialized := TRUE;
            END_IF;
            
            State := 1; (* Move to initialization *)

        1: (* Initialize *)
            (* Copy input to real part, set imaginary to zero *)
            FOR i := 1 TO N DO
                RealPart[i] := InputSignal[i];
                ImagPart[i] := 0.0;
                TempReal[i] := 0.0;
                TempImag[i] := 0.0;
            END_FOR;
            State := 2; (* Move to bit-reversal *)

        2: (* Bit-reversal reordering *)
            (* Reorder input array using bit-reversed indices *)
            FOR i := 1 TO N DO
                j := 0;
                FOR k := 0 TO LOG2_N - 1 DO
                    j := j OR ((i - 1) AND (1 << k) >> k) << (LOG2_N - 1 - k);
                END_FOR;
                j := j + 1; (* Adjust for 1-based indexing *)
                TempReal[j] := RealPart[i];
                TempImag[j] := ImagPart[i];
            END_FOR;
            
            (* Copy reordered data back *)
            FOR i := 1 TO N DO
                RealPart[i] := TempReal[i];
                ImagPart[i] := TempImag[i];
            END_FOR;
            
            State := 3; (* Move to butterfly computation *)
            Stage := 1;

        3: (* Butterfly computation *)
            IF Stage <= LOG2_N THEN
                (* Compute butterflies for current stage *)
                Step := UINT_SHL(1, Stage); (* Step = 2^Stage *)
                FOR k := 1 TO N/Step DO
                    FOR j := 0 TO Step/2 - 1 DO
                        n1 := (k - 1) * Step + j + 1; (* Index of first point *)
                        n2 := n1 + Step/2;             (* Index of second point *)
                        
                        (* Twiddle factor index *)
                        m := j * UINT_SHL(1, LOG2_N - Stage);
                        
                        (* Butterfly computation *)
                        TempR := CosTable[m] * RealPart[n2] - SinTable[m] * ImagPart[n2];
                        TempI := CosTable[m] * ImagPart[n2] + SinTable[m] * RealPart[n2];
                        
                        (* Update values *)
                        RealPart[n2] := RealPart[n1] - TempR;
                        ImagPart[n2] := ImagPart[n1] - TempI;
                        RealPart[n1] := RealPart[n1] + TempR;
                        ImagPart[n1] := ImagPart[n1] + TempI;
                    END_FOR;
                END_FOR;
                
                Stage := Stage + 1;
                IF Stage > LOG2_N THEN
                    State := 4; (* Move to magnitude computation *)
                END_IF;
            END_IF;

        4: (* Compute magnitude spectrum *)
            (* Calculate magnitude for first N/2 bins *)
            FOR i := 1 TO N/2 DO
                Magnitude[i] := SQRT(RealPart[i] * RealPart[i] + ImagPart[i] * ImagPart[i]) / REAL_OF_UINT(N);
                IF Magnitude[i] > 1.0E10 THEN
                    Error := TRUE;
                    ErrorID := 16#80020000; (* Numerical overflow *)
                    State := 0;
                    RETURN;
                END_IF;
            END_FOR;
            
            Done := TRUE;
            State := 0;
    END_CASE;
ELSE
    (* Reset on Execute falling edge *)
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    FOR i := 1 TO 32 DO
        Magnitude[i] := 0.0;
    END_FOR;
    State := 0;
END_IF;

END_FUNCTION_BLOCK
