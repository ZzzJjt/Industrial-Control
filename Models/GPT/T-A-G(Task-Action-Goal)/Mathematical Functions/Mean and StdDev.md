FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT;
END_VAR
VAR_OUTPUT
    Mean : REAL;
    StdDev : REAL;
END_VAR
VAR
    i : INT;
    Sum : REAL := 0.0;
    Temp : REAL;
    VarianceSum : REAL := 0.0;
END_VAR

(* Step 1: Calculate the Mean *)
Sum := 0.0;
FOR i := 1 TO 100 DO
    Sum := Sum + REAL(InputArray[i]);
END_FOR;
Mean := Sum / 100.0;

(* Step 2: Calculate the Standard Deviation *)
VarianceSum := 0.0;
FOR i := 1 TO 100 DO
    Temp := REAL(InputArray[i]) - Mean; (* Difference from mean *)
    VarianceSum := VarianceSum + Temp * Temp; (* Sum of squared differences *)
END_FOR;
StdDev := SQRT(VarianceSum / 99.0); (* Sample standard deviation *)
