FUNCTION_BLOCK FFT64
VAR_INPUT
    Execute      : BOOL;
    InputSignal  : ARRAY[1..64] OF REAL; // Real-valued time-domain input
END_VAR
VAR_OUTPUT
    Done         : BOOL;
    RealPart     : ARRAY[1..64] OF REAL; // Real part of frequency-domain output
    ImagPart     : ARRAY[1..64] OF REAL; // Imaginary part
END_VAR
VAR
    i, j, k, n, m, stage, index, halfSize, step : INT;
    angle, twiddleRe, twiddleIm, tempRe, tempIm : REAL;
    CosLUT : ARRAY[0..31] OF REAL := [
        1.0, 0.995185, 0.980785, 0.956940, 0.923880, 0.881921, 0.831470, 0.773010,
        0.707107, 0.634393, 0.555570, 0.471397, 0.382683, 0.290285, 0.195090, 0.098017,
        0.000000, -0.098017, -0.195090, -0.290285, -0.382683, -0.471397, -0.555570, -0.634393,
        -0.707107, -0.773010, -0.831470, -0.881921, -0.923880, -0.956940, -0.980785, -0.995185];
    SinLUT : ARRAY[0..31] OF REAL := [
        0.000000, 0.098017, 0.195090, 0.290285, 0.382683, 0.471397, 0.555570, 0.634393,
        0.707107, 0.773010, 0.831470, 0.881921, 0.923880, 0.956940, 0.980785, 0.995185,
        1.000000, 0.995185, 0.980785, 0.956940, 0.923880, 0.881921, 0.831470, 0.773010,
        0.707107, 0.634393, 0.555570, 0.471397, 0.382683, 0.290285, 0.195090, 0.098017];
    BitReversedIndex : ARRAY[1..64] OF INT := [
        1,33,17,49,9,41,25,57,5,37,21,53,13,45,29,61,
        3,35,19,51,11,43,27,59,7,39,23,55,15,47,31,63,
        2,34,18,50,10,42,26,58,6,38,22,54,14,46,30,62,
        4,36,20,52,12,44,28,60,8,40,24,56,16,48,32,64];
END_VAR

// Initialize FFT arrays from InputSignal
IF Execute AND NOT Done THEN
    FOR i := 1 TO 64 DO
        RealPart[i] := InputSignal[BitReversedIndex[i]];
        ImagPart[i] := 0.0;
    END_FOR;

    // Cooley-Tukey FFT: log2(64) = 6 stages
    m := 6;
    FOR stage := 1 TO m DO
        n := 2 ** stage;
        halfSize := n / 2;
        step := 64 / n;

        FOR k := 0 TO step - 1 DO
            FOR j := 0 TO halfSize - 1 DO
                index := j * 64 / n;
                twiddleRe := CosLUT[index MOD 32];
                twiddleIm := -SinLUT[index MOD 32];

                i := k * n + j + 1;
                n := halfSize;

                // Butterfly operation
                tempRe := RealPart[i + n] * twiddleRe - ImagPart[i + n] * twiddleIm;
                tempIm := RealPart[i + n] * twiddleIm + ImagPart[i + n] * twiddleRe;

                RealPart[i + n] := RealPart[i] - tempRe;
                ImagPart[i + n] := ImagPart[i] - tempIm;
                RealPart[i] := RealPart[i] + tempRe;
                ImagPart[i] := ImagPart[i] + tempIm;
            END_FOR;
        END_FOR;
    END_FOR;

    Done := TRUE;
END_IF;

