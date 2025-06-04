FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[0..63] OF REAL; (* N = 64 input samples *)
    Start : BOOL;                       (* Start trigger *)
END_VAR
VAR_OUTPUT
    Amplitude : ARRAY[0..31] OF REAL;   (* Output amplitude spectrum *)
    Done : BOOL;                        (* TRUE when FFT is complete *)
END_VAR
VAR
    ReX, ReY : ARRAY[0..63] OF REAL;    (* Real part storage *)
    ImX, ImY : ARRAY[0..63] OF REAL;    (* Imaginary part storage *)
    PI : REAL := 3.1415926;
    i, j, k, n, m : INT;
    stage, stages : INT;
    step, halfStep : INT;
    angle, cosVal, sinVal : REAL;
    tempRe, tempIm : REAL;
    bit, reversed : INT;
    temp : REAL;
    running : BOOL := FALSE;
END_VAR

(* === Start FFT Computation === *)
IF Start AND NOT running THEN
    running := TRUE;
    Done := FALSE;

    (* Step 1: Initialize real and imag arrays with input *)
    FOR i := 0 TO 63 DO
        ReX[i] := InputSignal[i];
        ImX[i] := 0.0;
    END_FOR

    (* Step 2: Bit-reversal reordering *)
    FOR i := 0 TO 63 DO
        reversed := 0;
        temp := i;
        FOR bit := 0 TO 5 DO (* log2(64) = 6 *)
            reversed := (reversed * 2) + (temp MOD 2);
            temp := temp / 2;
        END_FOR
        ReY[i] := ReX[reversed];
        ImY[i] := ImX[reversed];
    END_FOR

    (* Step 3: FFT Butterfly computation *)
    stages := 6; (* log2(64) = 6 *)
    FOR stage := 1 TO stages DO
        step := 2 ** stage;
        halfStep := step / 2;

        FOR k := 0 TO 63 BY step DO
            FOR j := 0 TO halfStep - 1 DO
                angle := -2.0 * PI * j / step;
                cosVal := COS(angle);
                sinVal := SIN(angle);

                i := k + j;
                m := i + halfStep;

                tempRe := cosVal * ReY[m] - sinVal * ImY[m];
                tempIm := cosVal * ImY[m] + sinVal * ReY[m];

                ReX[i] := ReY[i] + tempRe;
                ImX[i] := ImY[i] + tempIm;
                ReX[m] := ReY[i] - tempRe;
                ImX[m] := ImY[i] - tempIm;
            END_FOR
        END_FOR

        (* Swap pointers for next stage *)
        FOR i := 0 TO 63 DO
            ReY[i] := ReX[i];
            ImY[i] := ImX[i];
        END_FOR
    END_FOR

    (* Step 4: Compute amplitude spectrum (magnitude) *)
    FOR i := 0 TO 31 DO
        Amplitude[i] := SQRT(ReX[i] * ReX[i] + ImX[i] * ImX[i]) / 64.0;
    END_FOR

    Done := TRUE;
    running := FALSE;
END_IF
