FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[0..63] OF REAL; // Fixed length input signal, e.g., 64 points
END_VAR

VAR_OUTPUT
    OutputMagnitude : ARRAY[0..32] OF REAL; // Magnitude output, half due to symmetry
    OutputPhase : ARRAY[0..32] OF REAL; // Phase output
END_VAR

VAR
    TempReal : ARRAY[0..63] OF REAL;
    TempImag : ARRAY[0..63] OF REAL := 0; // Initialize imaginary parts to zero
    BitReversedIndex : ARRAY[0..63] OF INT;
    i, j, k : INT;
    n : INT := SIZEOF(InputSignal); // Size of input array
    m : INT;
    stageCount : INT;
    butterflySize : INT;
    halfButterflySize : INT;
    twiddleReal : REAL;
    twiddleImag : REAL;
    tempRealVal : REAL;
    tempImagVal : REAL;
    angle : REAL;
END_VAR

// Precompute bit-reversed indices
FOR i := 0 TO n - 1 DO
    BitReversedIndex[i] := ReverseBits(i, LOG2(n));
END_FOR

// Copy input signal to temporary arrays and reorder by bit-reversed index
FOR i := 0 TO n - 1 DO
    TempReal[BitReversedIndex[i]] := InputSignal[i];
END_FOR

// Perform FFT using Cooley-Tukey algorithm
m := LOG2(n);
stageCount := m;
FOR s := 1 TO stageCount DO
    butterflySize := POW(2, s);
    halfButterflySize := butterflySize / 2;
    FOR k := 0 TO halfButterflySize - 1 DO
        angle = 2 * PI * k / butterflySize;
        twiddleReal = COS(angle);
        twiddleImag = -SIN(angle);
        FOR j := 0 TO n - 1 BY butterflySize DO
            i := j + k;
            tempRealVal := TempReal[i] + (TempReal[i + halfButterflySize] * twiddleReal - TempImag[i + halfButterflySize] * twiddleImag);
            tempImagVal := TempImag[i] + (TempImag[i + halfButterflySize] * twiddleReal + TempReal[i + halfButterflySize] * twiddleImag);
            TempReal[i] := tempRealVal;
            TempImag[i] := tempImagVal;
            TempReal[i + halfButterflySize] := TempReal[i] - 2 * (TempReal[i + halfButterflySize] * twiddleReal - TempImag[i + halfButterflySize] * twiddleImag);
            TempImag[i + halfButterflySize] := TempImag[i] - 2 * (TempImag[i + halfButterflySize] * twiddleReal + TempReal[i + halfButterflySize] * twiddleImag);
        END_FOR
    END_FOR
END_FOR

// Calculate magnitude and phase
FOR i := 0 TO n/2 DO
    OutputMagnitude[i] := SQRT(TempReal[i]*TempReal[i] + TempImag[i]*TempImag[i]);
    OutputPhase[i] := ATAN2(TempImag[i], TempReal[i]);
END_FOR

END_FUNCTION_BLOCK

// Helper function to compute bit-reversed index
FUNCTION ReverseBits(IN val : INT; IN numBits : INT) : INT
VAR
    reversedVal : INT := 0;
    i : INT;
END_VAR

FOR i := 0 TO numBits - 1 DO
    IF BIT_TEST(val, i) THEN
        BIT_SET(reversedVal, numBits - 1 - i);
    ELSE
        BIT_CLR(reversedVal, numBits - 1 - i);
    END_IF
END_FOR

ReverseBits := reversedVal;

END_FUNCTION
