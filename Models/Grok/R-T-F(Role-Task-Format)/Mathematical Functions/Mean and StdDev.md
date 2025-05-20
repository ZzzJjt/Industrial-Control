FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT; (* Input array of 100 integer values *)
END_VAR
VAR_OUTPUT
    Mean : REAL; (* Arithmetic mean of the input values *)
    StdDev : REAL; (* Sample standard deviation *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR
VAR
    i : INT; (* Loop index *)
    Sum : REAL; (* Accumulator for sum of values *)
    SumSquaredDiff : REAL; (* Accumulator for sum of squared differences *)
    Diff : REAL; (* Difference between value and mean *)
    N : REAL := 100.0; (* Array size as REAL for division *)
    N_minus_1 : REAL := 99.0; (* N-1 for sample standard deviation *)
END_VAR

(* Initialize outputs *)
Mean := 0.0;
StdDev := 0.0;
ErrorCode := 0;
Sum := 0.0;

(* Compute mean: sum all values and divide by N *)
FOR i := 1 TO 100 DO
    (* Convert INT to REAL to prevent overflow and ensure precision *)
    Sum := Sum + INT_TO_REAL(InputArray[i]);
END_FOR;

(* Check for valid sum to avoid division issues *)
IF NOT IS_VALID_REAL(Sum) THEN
    ErrorCode := 1; (* Invalid input or overflow *)
    RETURN;
END_IF;

(* Calculate mean: Sum / N *)
Mean := Sum / N;

(* Compute sample standard deviation *)
SumSquaredDiff := 0.0;
FOR i := 1 TO 100 DO
    (* Calculate difference from mean *)
    Diff := INT_TO_REAL(InputArray[i]) - Mean;
    (* Accumulate squared differences *)
    SumSquaredDiff := SumSquaredDiff + (Diff * Diff);
END_FOR;

(* Check for valid sum of squared differences *)
IF NOT IS_VALID_REAL(SumSquaredDiff) OR SumSquaredDiff < 0.0 THEN
    ErrorCode := 1; (* Invalid result *)
    RETURN;
END_IF;

(* Calculate standard deviation: sqrt(sum of squared differences / (N-1)) *)
(* Use N-1 for sample standard deviation to account for degrees of freedom *)
StdDev := SQRT(SumSquaredDiff / N_minus_1);

(* Validate final standard deviation *)
IF NOT IS_VALID_REAL(StdDev) THEN
    ErrorCode := 1; (* Invalid result *)
    StdDev := 0.0;
END_IF;

(* Helper function to check if a REAL value is valid *)
METHOD IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_METHOD

END_FUNCTION_BLOCK
