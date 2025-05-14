(* Function Block: MeanStdDevCalculator
   Purpose: Computes the mean and standard deviation of a 100-element integer array.
   Features:
   - Input: InputArray (ARRAY[1..100] OF INT)
   - Outputs: Mean (REAL, arithmetic average), StdDev (REAL, sample standard deviation)
   - Logic:
     1. Sum all values and divide by 100 to compute Mean
     2. Sum squared differences from Mean, divide by (N-1), take square root for StdDev
   - Uses REAL for calculations to prevent integer overflow
   - Includes detailed comments for each step
   - Optimized for PLCs: deterministic, minimal memory, robust error handling
   Notes:
   - Assumes 32-bit REAL arithmetic (IEEE 754 single-precision)
   - Uses sample standard deviation (divide by N-1=99) for statistical accuracy
   - Suitable for sensor noise filtering, statistical quality control, cycle time monitoring
*)

FUNCTION_BLOCK MeanStdDevCalculator
VAR_INPUT
    InputArray : ARRAY[1..100] OF INT;      (* Input array of 100 integers *)
END_VAR
VAR_OUTPUT
    Mean : REAL;                            (* Arithmetic mean of the array *)
    StdDev : REAL;                          (* Sample standard deviation *)
    ErrorFlag : BOOL;                       (* TRUE if computation fails *)
END_VAR
VAR
    i : UINT;                               (* Loop index *)
    Sum : REAL;                             (* Sum of array values *)
    SumSquaredDiff : REAL;                  (* Sum of squared differences *)
    Temp : REAL;                            (* Temporary difference from mean *)
    N : REAL := 100.0;                      (* Array size as REAL *)
    NMinus1 : REAL := 99.0;                 (* N-1 for sample standard deviation *)
END_VAR

(* Initialize outputs *)
Mean := 0.0;
StdDev := 0.0;
ErrorFlag := FALSE;

(* Step 1: Compute the mean *)
Sum := 0.0;
FOR i := 1 TO 100 DO
    (* Convert INT to REAL and accumulate sum *)
    Sum := Sum + REAL(InputArray[i]);
    
    (* Check for potential overflow in sum *)
    IF ABS(Sum) > 1.0E6 THEN
        ErrorFlag := TRUE;
        EXIT;
    END_IF
END_FOR

IF NOT ErrorFlag THEN
    (* Calculate mean: Sum / N *)
    Mean := Sum / N;
    
    (* Step 2: Compute sum of squared differences *)
    SumSquaredDiff := 0.0;
    FOR i := 1 TO 100 DO
        (* Compute difference from mean *)
        Temp := REAL(InputArray[i]) - Mean;
        
        (* Accumulate squared difference *)
        SumSquaredDiff := SumSquaredDiff + (Temp * Temp);
        
        (* Check for overflow in sum of squares *)
        IF SumSquaredDiff > 1.0E6 THEN
            ErrorFlag := TRUE;
            EXIT;
        END_IF
    END_FOR
END_IF

(* Step 3: Compute standard deviation *)
IF NOT ErrorFlag THEN
    (* Sample standard deviation: SQRT(SumSquaredDiff / (N-1)) *)
    StdDev := SQRT(SumSquaredDiff / NMinus1);
    
    (* Check for numerical issues in result *)
    IF ABS(StdDev) > 1.0E6 OR StdDev <> StdDev THEN (* NaN check *)
        ErrorFlag := TRUE;
        StdDev := 0.0;
        Mean := 0.0;
    END_IF
ELSE
    (* Set outputs to safe values on error *)
    Mean := 0.0;
    StdDev := 0.0;
END_IF

END_FUNCTION_BLOCK
