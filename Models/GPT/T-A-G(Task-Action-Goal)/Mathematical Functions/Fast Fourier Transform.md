FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputArray : ARRAY[1..64] OF REAL;
    Execute : BOOL;
END_VAR

VAR_OUTPUT
    MagnitudeSpectrum : ARRAY[1..32] OF REAL; // Only positive frequencies
    Done : BOOL;
    Error : BOOL;
END_VAR

VAR
    RealPart : ARRAY[1..64] OF REAL;
    ImagPart : ARRAY[1..64] OF REAL;
    BitReversedIndex : ARRAY[1..64] OF INT;
    PI : REAL := 3.14159265;
    N : INT := 64;
    i, j, k, m, n1, n2, a : INT;
    tReal, tImag, uReal, uImag, angle : REAL;
    Stage : INT;
    TempReal, TempImag : REAL;
    Executed : BOOL;
END_VAR

// Bit-reversal ordering
FOR i := 0 TO N - 1 DO
    j := 0;
    FOR k := 0 TO 5 DO // log2(64) = 6, k from 0 to 5
        IF ((i / (2 ** k)) MOD 2) = 1 THEN
            j := j + (2 ** (5 - k));
        END_IF;
    END_FOR;
    BitReversedIndex[i+1] := j + 1;
    RealPart[j+1] := InputArray[i+1];
    ImagPart[j+1] := 0.0;
END_FOR;

// FFT computation using Cooley-Tukey radix-2 DIT
n2 := 1;
FOR Stage := 1 TO 6 DO // log2(64) = 6
    n1 := n2;
    n2 := n2 * 2;
    FOR j := 0 TO n1 - 1 DO
        angle := -2.0 * PI * j / n2;
        c := COS(angle);
        s := SIN(angle);
        FOR i := j TO N - 1 BY n2 DO
            a := i + n1;
            tReal := c * RealPart[a+1] - s * ImagPart[a+1];
            tImag := s * RealPart[a+1] + c * ImagPart[a+1];
            uReal := RealPart[i+1];
            uImag := ImagPart[i+1];
            RealPart[i+1] := uReal + tReal;
            ImagPart[i+1] := uImag + tImag;
            RealPart[a+1] := uReal - tReal;
            ImagPart[a+1] := uImag - tImag;
        END_FOR;
    END_FOR;
END_FOR;

// Compute magnitude spectrum
FOR i := 1 TO 32 DO // Only positive frequencies
    MagnitudeSpectrum[i] := SQRT(RealPart[i] * RealPart[i] + ImagPart[i] * ImagPart[i]);
END_FOR;

Done := TRUE;
Error := FALSE;
END_FUNCTION_BLOCK
