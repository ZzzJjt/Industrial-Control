FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[1..8] OF REAL; // Real-valued time-domain input
END_VAR
VAR_OUTPUT
    Magnitude : ARRAY[1..8] OF REAL;   // Frequency-domain magnitude output
END_VAR
VAR
    RealPart  : ARRAY[1..8] OF REAL;
    ImagPart  : ARRAY[1..8] OF REAL;
    TempRe, TempIm : REAL;
    Angle, CosVal, SinVal : REAL;
    Stage, Index, Step, HalfStep, i, j, k : INT;
    N : INT := 8;
END_VAR

// Initialization: copy input to real part, zero imaginary
FOR i := 1 TO N DO
    RealPart[i] := InputSignal[i];
    ImagPart[i] := 0.0;
END_FOR

// Bit-reversal (only valid for N=8)
VAR
    BitRev : ARRAY[1..8] OF INT := [1,5,3,7,2,6,4,8]; // 1-based index
    TempReArr, TempImArr : ARRAY[1..8] OF REAL;
END_VAR
FOR i := 1 TO N DO
    TempReArr[i] := RealPart[BitRev[i]];
    TempImArr[i] := ImagPart[BitRev[i]];
END_FOR
FOR i := 1 TO N DO
    RealPart[i] := TempReArr[i];
    ImagPart[i] := TempImArr[i];
END_FOR

// FFT Cooley-Tukey
Stage := 1;
WHILE (Stage <= 3) DO // log2(8) = 3
    Step := 2 ** Stage;
    HalfStep := Step / 2;

    FOR k := 1 TO N BY Step DO
        FOR j := 0 TO HalfStep - 1 DO
            Angle := -2.0 * 3.14159265 * j / Step;
            CosVal := COS(Angle);
            SinVal := SIN(Angle);

            i := k + j;
            Index := i + HalfStep;

            TempRe :=  RealPart[Index] * CosVal - ImagPart[Index] * SinVal;
            TempIm :=  RealPart[Index] * SinVal + ImagPart[Index] * CosVal;

            RealPart[Index] := RealPart[i] - TempRe;
            ImagPart[Index] := ImagPart[i] - TempIm;

            RealPart[i] := RealPart[i] + TempRe;
            ImagPart[i] := ImagPart[i] + TempIm;
        END_FOR
    END_FOR
    Stage := Stage + 1;
END_WHILE

// Compute magnitude
FOR i := 1 TO N DO
    Magnitude[i] := SQRT(RealPart[i]*RealPart[i] + ImagPart[i]*ImagPart[i]);
END_FOR
