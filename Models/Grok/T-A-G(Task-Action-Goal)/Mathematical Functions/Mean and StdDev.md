(* Function Block: Mean and Standard Deviation Calculator for 100 Integer Values *)
FUNCTION_BLOCK FB_MeanStdDevCalculator
VAR_INPUT
    Execute : BOOL;                       (* Trigger computation *)
    InputArray : ARRAY[1..100] OF INT;    (* Input array of 100 integer values *)
END_VAR

VAR_OUTPUT
    Mean : REAL;                          (* Computed mean of the input array *)
    StdDev : REAL;                        (* Computed sample standard deviation *)
    Done : BOOL;                          (* Computation completed *)
    Error : BOOL;                         (* Error flag *)
    ErrorID : DWORD;                      (* Error code *)
END_VAR

VAR
    i : UINT;                             (* Loop index *)
    Sum : REAL;                           (* Accumulator for sum of values *)
    SumSquaredDiff : REAL;                (* Accumulator for sum of squared differences *)
    Diff : REAL;                          (* Temporary difference from mean *)
    N : REAL := 100.0;                    (* Array size as REAL for division *)
    N_minus_1 : REAL := 99.0;             (* N-1 for sample standard deviation *)
END_VAR

(* Initialize outputs *)
Mean := 0.0;
StdDev := 0.0;
Done := FALSE;
Error := FALSE;
ErrorID := 0;

(* Main logic *)
IF Execute THEN
    (* Validate inputs to prevent overflow *)
    FOR i := 1 TO 100 DO
        IF ABS(InputArray[i]) > 32767 THEN (* INT range: -32768 to 32767 *)
            Error := TRUE;
            ErrorID := 16#80010000;       (* Input value out of range *)
            Done := TRUE;
            RETURN;
        END_IF;
    END_FOR;

    (* Step 1: Compute mean *)
    Sum := 0.0;                           (* Initialize sum accumulator *)
    FOR i := 1 TO 100 DO
        (* Convert INT to REAL to avoid overflow and ensure precision *)
        Sum := Sum + REAL_OF_INT(InputArray[i]);
    END_FOR;
    
    (* Calculate mean: Sum / N *)
    Mean := Sum / N;                      (* Divide by 100.0 for average *)
    
    (* Check for numerical issues in mean *)
    IF ABS(Mean) > 1.0E10 THEN
        Error := TRUE;
        ErrorID := 16#80020000;           (* Numerical overflow in mean *)
        Done := TRUE;
        RETURN;
    END_IF;

    (* Step 2: Compute sample standard deviation *)
    SumSquaredDiff := 0.0;                (* Initialize accumulator for squared differences *)
    FOR i := 1 TO 100 DO
        (* Compute difference from mean *)
        Diff := REAL_OF_INT(InputArray[i]) - Mean;
        (* Accumulate squared difference *)
        SumSquaredDiff := SumSquaredDiff + Diff * Diff;
    END_FOR;
    
    (* Calculate variance: SumSquaredDiff / (N-1) *)
    (* Use N-1 (99.0) for sample standard deviation to correct for bias *)
    StdDev := SQRT(SumSquaredDiff / N_minus_1);
    
    (* Check for numerical issues in standard deviation *)
    IF ABS(StdDev) > 1.0E10 OR StdDev < 0.0 THEN
        Error := TRUE;
        ErrorID := 16#80020001;           (* Numerical issue in standard deviation *)
        StdDev := 0.0;                    (* Fallback to zero *)
    END_IF;

    Done := TRUE;                         (* Mark computation as complete *)
ELSE
    (* Reset outputs when Execute is FALSE *)
    Mean := 0.0;
    StdDev := 0.0;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_FUNCTION_BLOCK
