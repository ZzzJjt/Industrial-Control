FUNCTION_BLOCK FB_FFT_64
VAR_INPUT
    // Real-valued input signal array (fixed size)
    InputSignal : ARRAY[0..63] OF REAL;
END_VAR

VAR_OUTPUT
    // Output arrays for real and imaginary parts after FFT
    RealPart : ARRAY[0..32] OF REAL; // Only up to Nyquist frequency
    ImagPart : ARRAY[0..32] OF REAL;
    Magnitude : ARRAY[0..32] OF REAL; // Optional output for magnitude spectrum
END_VAR

VAR
    BitReversedIndices : ARRAY[0..63] OF INT; // For bit-reversal reordering
    TwiddleFactors : ARRAY[0..31] OF REAL; // Precomputed cosine values for twiddle factors
    i, j, k, m, n, stage, index : DINT;
    tempReal, tempImag : REAL;
    swapTemp : REAL;
    theta, cosVal, sinVal : REAL;
END_VAR

// Initialization method - call this once during setup
METHOD Initialize : BOOL
VAR
    i, j, bit, revIdx : DINT;
BEGIN
    // Compute twiddle factors (cosine values only, as sine is derived from it)
    FOR i := 0 TO 31 DO
        theta := -2 * PI * i / 64;
        TwiddleFactors[i] := COS(theta);
    END_FOR;

    // Compute bit-reversed indices for 6-bit numbers
    FOR i := 0 TO 63 DO
        j := i;
        revIdx := 0;
        FOR bit := 0 TO 5 DO // 6 bits for 64-point FFT
            IF ((j AND (1 << bit)) <> 0) THEN
                revIdx := revIdx OR (1 << (5 - bit));
            END_IF;
        END_FOR;
        BitReversedIndices[i] := revIdx;
    END_FOR;
    
    Initialize := TRUE;
END_METHOD

METHOD Execute : BOOL
VAR
    halfSize, stepSize, currentStage : DINT;
BEGIN
    // Bit-reversal reordering of input signal
    FOR i := 0 TO 63 DO
        IF i < BitReversedIndices[i] THEN
            swapTemp := InputSignal[i];
            InputSignal[i] := InputSignal[BitReversedIndices[i]];
            InputSignal[BitReversedIndices[i]] := swapTemp;
        END_IF;
    END_FOR;

    // FFT computation
    halfSize := 1;
    WHILE halfSize < 64 DO
        stepSize := halfSize * 2;
        FOR m := 0 TO halfSize - 1 DO
            FOR k := m TO 63 STEP stepSize DO
                index := k + halfSize;
                theta := -PI * m / halfSize;
                cosVal := COS(theta);
                sinVal := SIN(theta); // Derived from cosine for efficiency
                
                tempReal := cosVal * InputSignal[index] + sinVal * 0; // Assuming imaginary part starts as zero
                tempImag := cosVal * 0 - sinVal * InputSignal[k]; // Same here
                
                RealPart[k] := InputSignal[k] + tempReal;
                ImagPart[k] := tempImag;
                
                RealPart[index] := InputSignal[k] - tempReal;
                ImagPart[index] := -tempImag;
            END_FOR;
        END_FOR;
        halfSize := stepSize;
    END_WHILE;

    // Calculate magnitude for output
    FOR i := 0 TO 32 DO // Up to Nyquist frequency
        Magnitude[i] := SQRT(RealPart[i]*RealPart[i] + ImagPart[i]*ImagPart[i]);
    END_FOR;

    Execute := TRUE;
END_METHOD
