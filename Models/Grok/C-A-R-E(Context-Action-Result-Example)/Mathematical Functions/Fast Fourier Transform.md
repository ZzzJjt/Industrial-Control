(* IEC 61131-3 Structured Text: FFT_Block Function Block *)
(* Purpose: Computes Fast Fourier Transform of a time-domain signal *)

FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[1..64] OF REAL; (* Time-domain signal, N=64 *)
    N : INT;                            (* Signal length, power of 2, e.g., 64 *)
    ApplyWindow : BOOL;                 (* TRUE to apply Hamming window *)
    SampleRate : REAL;                  (* Sampling frequency, Hz, e.g., 1000 *)
END_VAR
VAR_OUTPUT
    Amplitude : ARRAY[1..32] OF REAL;   (* Amplitude spectrum, N/2 bins *)
    FrequencyBins : ARRAY[1..32] OF REAL; (* Frequencies, Hz, if SampleRate > 0 *)
    ErrorFlag : BOOL;                   (* TRUE if computation fails *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                     (* Number of log entries *)
END_VAR
VAR
    RealPart : ARRAY[1..64] OF REAL;    (* Real components, in-place *)
    ImagPart : ARRAY[1..64] OF REAL;    (* Imaginary components, in-place *)
    BitReverse : ARRAY[1..64] OF INT;   (* Bit-reversal indices *)
    TwiddleCos : ARRAY[1..32] OF REAL;  (* Precomputed cosines *)
    TwiddleSin : ARRAY[1..32] OF REAL;  (* Precomputed sines *)
    i, j, k, m, stage : INT;            (* Loop and stage indices *)
    nStages : INT;                      (* Number of FFT stages, log2(N) *)
    tempReal, tempImag : REAL;          (* Temporary values for butterfly *)
    angle, norm : REAL;                 (* Angle for twiddle, normalization *)
    Timestamp : STRING[20];             (* Simulated timestamp *)
    LogBufferFull : BOOL;               (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Step 1: Initialize outputs and validate inputs *)
ErrorFlag := FALSE;
FOR i := 1 TO 32 DO
    Amplitude[i] := 0.0;
    FrequencyBins[i] := 0.0;
END_FOR;

(* Validate N as power of 2 *)
IF N <> 64 OR N <= 0 THEN
    ErrorFlag := TRUE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-17 20:00:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid N: Must be 64');
    END_IF;
    RETURN;
END_IF;

(* Validate InputSignal for finite values *)
FOR i := 1 TO N DO
    IF NOT IS_VALID_REAL(InputSignal[i]) THEN
        ErrorFlag := TRUE;
        IF LogCount < 50 THEN
            LogCount := LogCount + 1;
            Timestamp := '2025-05-17 20:00:00';
            DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid InputSignal at index ', TO_STRING(i));
        END_IF;
        RETURN;
    END_IF;
END_FOR;

(* Step 2: Apply Hamming window if enabled *)
IF ApplyWindow THEN
    FOR i := 1 TO N DO
        (* Hamming window: w[i] = 0.54 - 0.46 * cos(2π(i-1)/(N-1)) *)
        angle := 2.0 * 3.141592653589793 * (i - 1) / (N - 1);
        RealPart[i] := InputSignal[i] * (0.54 - 0.46 * COS(angle));
        ImagPart[i] := 0.0;
    END_FOR;
ELSE
    FOR i := 1 TO N DO
        RealPart[i] := InputSignal[i];
        ImagPart[i] := 0.0;
    END_FOR;
END_IF;

(* Step 3: Precompute bit-reversal indices *)
FOR i := 1 TO N DO
    j := 0;
    k := i - 1;
    FOR m := 0 TO 5 DO (* log2(64) = 6 stages *)
        j := j * 2 + (k MOD 2);
        k := k / 2;
    END_FOR;
    BitReverse[i] := j + 1;
END_FOR;

(* Step 4: Reorder input using bit-reversal *)
FOR i := 1 TO N DO
    j := BitReverse[i];
    IF i < j THEN
        tempReal := RealPart[i];
        tempImag := ImagPart[i];
        RealPart[i] := RealPart[j];
        ImagPart[i] := ImagPart[j];
        RealPart[j] := tempReal;
        ImagPart[j] := tempImag;
    END_IF;
END_FOR;

(* Step 5: Precompute twiddle factors *)
FOR k := 0 TO N/2 - 1 DO
    angle := -2.0 * 3.141592653589793 * k / N;
    TwiddleCos[k + 1] := COS(angle);
    TwiddleSin[k + 1] := SIN(angle);
END_FOR;

(* Step 6: Perform FFT with radix-2 Cooley-Tukey *)
nStages := 6; (* log2(64) *)
FOR stage := 1 TO nStages DO
    (* Step 6.1: Compute block size and twiddle step *)
    m := 2 ** stage; (* Block size *)
    k := m / 2;      (* Twiddle factor step *)
    
    (* Step 6.2: Process each block *)
    FOR i := 1 TO N STEP m DO
        FOR j := 0 TO k - 1 DO
            (* Step 6.3: Butterfly operation *)
            tempReal := TwiddleCos[j * (N / m) + 1] * RealPart[i + j + k] - 
                        TwiddleSin[j * (N / m) + 1] * ImagPart[i + j + k];
            tempImag := TwiddleCos[j * (N / m) + 1] * ImagPart[i + j + k] + 
                        TwiddleSin[j * (N / m) + 1] * RealPart[i + j + k];
            
            (* Update upper and lower halves *)
            RealPart[i + j + k] := RealPart[i + j] - tempReal;
            ImagPart[i + j + k] := ImagPart[i + j] - tempImag;
            RealPart[i + j] := RealPart[i + j] + tempReal;
            ImagPart[i + j] := ImagPart[i + j] + tempImag;
        END_FOR;
    END_FOR;
END_FOR;

(* Step 7: Compute amplitude spectrum *)
norm := 2.0 / N; (* Normalization factor *)
FOR i := 1 TO N/2 DO
    (* Magnitude: sqrt(real^2 + imag^2) * norm *)
    Amplitude[i] := norm * SQRT(RealPart[i] * RealPart[i] + ImagPart[i] * ImagPart[i]);
    
    (* Compute frequency bins if SampleRate provided *)
    IF SampleRate > 0.0 THEN
        FrequencyBins[i] := (i - 1) * SampleRate / N;
    END_IF;
END_FOR;

(* Step 8: Log completion *)
IF LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-17 20:00:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' FFT Completed – NumBins: ', TO_STRING(N/2));
END_IF;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Computes FFT of a 64-point signal using radix-2 Cooley-Tukey.
   - Inputs:
     - InputSignal: ARRAY[1..64] OF REAL, time-domain signal.
     - N: INT, signal length (fixed at 64, power of 2).
     - ApplyWindow: BOOL, TRUE for Hamming window.
     - SampleRate: REAL, sampling frequency (Hz) for frequency bins.
   - Outputs:
     - Amplitude: ARRAY[1..32] OF REAL, magnitude spectrum (N/2 bins).
     - FrequencyBins: ARRAY[1..32] OF REAL, frequencies (Hz) if SampleRate > 0.
     - ErrorFlag: BOOL, TRUE if invalid N or input.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Algorithm:
     - Radix-2 Cooley-Tukey FFT, in-place, O(N log N) complexity.
     - Steps:
       1. Validate inputs, copy to RealPart, apply Hamming window if enabled.
       2. Precompute bit-reversal indices for reordering.
       3. Reorder input using bit-reversal.
       4. Precompute twiddle factors (cos, sin).
       5. Perform log2(N) stages of butterfly operations.
       6. Compute amplitude spectrum: sqrt(real^2 + imag^2) * 2/N.
       7. Compute frequency bins if SampleRate provided.
   - Optimization:
     - Preallocated arrays (RealPart, ImagPart) avoid dynamic allocation.
     - Fixed N=64 ensures bounded loops (6 stages, 32 butterflies/stage).
     - Precomputed twiddle factors reduce trigonometric calls.
     - In-place computation minimizes memory (~2*64 REALs).
   - Numerical Stability:
     - Validates finite inputs to prevent overflow/NaN.
     - Normalizes output by 2/N to scale amplitudes correctly.
     - Uses stable Cooley-Tukey algorithm, avoids small denominators.
   - Execution Time:
     - ~1000 FLOPs for N=64 (~1–2 ms on 1 MFLOP/s PLC), fits 10–100 ms cycles.
   - Usage:
     - Vibration monitoring: Processes 64-point signal, detects 25 Hz spike.
     - Example: Amplitude[13] = 0.5 at FrequencyBins[13] = 25 Hz (SampleRate=1000).
   - Platform Notes:
     - Assumes STRING[80], REAL (32-bit), INT; adjust for platform limits.
     - Timestamp is static; replace with system clock.
     - Compatible with PLCs (e.g., Siemens S7, Codesys) using Profibus DP or similar.
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
