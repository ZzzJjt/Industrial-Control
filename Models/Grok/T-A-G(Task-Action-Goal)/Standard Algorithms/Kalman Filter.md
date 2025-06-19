(* IEC 61131-3 Structured Text Function Block for 2-State Kalman Filter *)
(* Estimates position and velocity of an autonomous vehicle from noisy position measurements *)
(* Uses scalar arithmetic, avoids matrix operations and recursion for scan-cycle safety *)
(* Conforms to industrial automation best practices *)

FUNCTION_BLOCK KalmanFilter
VAR_INPUT
    MeasuredPosition: REAL; (* Noisy position measurement (meters) *)
    Q: REAL;               (* Process noise covariance, e.g., 0.01 *)
    R: REAL;               (* Measurement noise covariance, e.g., 1.0 *)
    DeltaT: REAL;          (* Sample time in seconds, e.g., 0.1 *)
    Reset: BOOL;           (* TRUE to reinitialize state *)
END_VAR

VAR_OUTPUT
    EstPosition: REAL;     (* Estimated position (meters) *)
    EstVelocity: REAL;     (* Estimated velocity (m/s) *)
    ValidOutput: BOOL;     (* TRUE if outputs are valid *)
END_VAR

VAR
    (* State estimates *)
    x0: REAL;              (* Estimated position (m) *)
    x1: REAL;              (* Estimated velocity (m/s) *)
    
    (* Error covariances *)
    P00: REAL;             (* Position variance *)
    P01: REAL;             (* Position-velocity covariance *)
    P10: REAL;             (* Symmetric: P10 = P01 *)
    P11: REAL;             (* Velocity variance *)
    
    (* Kalman gains *)
    K0: REAL;              (* Gain for position *)
    K1: REAL;              (* Gain for velocity *)
    
    (* Temporary variables *)
    Innovation: REAL;      (* Measurement residual *)
    S: REAL;               (* Innovation covariance *)
    Temp: REAL;            (* Temporary for calculations *)
    
    (* Previous state *)
    PrevPosition: REAL;    (* Previous measured position for initialization *)
    Initialized: BOOL;     (* TRUE after first valid measurement *)
    
    (* Validation *)
    ValidInput: BOOL;      (* TRUE if inputs are valid *)
END_VAR

(* Step 1: Handle Reset *)
(* Reinitialize state and covariances when Reset is TRUE *)
IF Reset THEN
    x0 := 0.0;             (* Reset position *)
    x1 := 0.0;             (* Reset velocity *)
    P00 := 1.0;            (* Initial position variance *)
    P01 := 0.0;            (* Initial position-velocity covariance *)
    P10 := 0.0;            (* Symmetric *)
    P11 := 1.0;            (* Initial velocity variance *)
    EstPosition := 0.0;
    EstVelocity := 0.0;
    ValidOutput := FALSE;
    Initialized := FALSE;
    PrevPosition := 0.0;
    RETURN;
END_IF;

(* Step 2: Input Validation *)
(* Ensure Q, R, DeltaT are positive and MeasuredPosition is reasonable *)
ValidInput := (Q > 0.0) AND (R > 0.0) AND (DeltaT > 0.0) AND (ABS(MeasuredPosition) < 1.0E6);

IF NOT ValidInput THEN
    ValidOutput := FALSE;
    RETURN;
END_IF;

(* Step 3: Initialize on First Measurement *)
(* Set initial position to first valid measurement *)
IF NOT Initialized THEN
    x0 := MeasuredPosition;
    x1 := 0.0;             (* Assume zero initial velocity *)
    P00 := R;              (* Initial variance based on measurement noise *)
    P01 := 0.0;
    P10 := 0.0;
    P11 := 1.0;            (* Arbitrary velocity variance *)
    PrevPosition := MeasuredPosition;
    Initialized := TRUE;
    EstPosition := x0;
    EstVelocity := x1;
    ValidOutput := TRUE;
    RETURN;
END_IF;

(* Step 4: Prediction Step *)
(* Predict state and covariance based on constant velocity model *)
(* State prediction: x0 = x0 + DeltaT * x1, x1 = x1 *)
Temp := x0;                (* Store current position *)
x0 := x0 + DeltaT * x1;    (* Predict position *)
x1 := x1;                  (* Velocity unchanged *)

(* Covariance prediction *)
(* P' = F * P * F^T + Q, where F = [1, DeltaT; 0, 1] *)
(* Compute each element: P00, P01, P10, P11 *)
Temp := P00;               (* Store current P00 *)
P00 := P00 + 2.0 * DeltaT * P01 + DeltaT * DeltaT * P11;
P01 := P01 + DeltaT * P11;
P10 := P01;                (* Symmetric *)
P11 := P11 + Q;            (* Add process noise to velocity variance *)

(* Step 5: Update Step *)
(* Compute Kalman gain, update state and covariance *)
(* Measurement: z = x0 + v, v ~ N(0, R) *)
Innovation := MeasuredPosition - x0;  (* Measurement residual *)
S := P00 + R;                        (* Innovation covariance *)

(* Kalman gain: K = P * H^T / S, H = [1, 0] *)
K0 := P00 / S;                       (* Gain for position *)
K1 := P01 / S;                       (* Gain for velocity *)

(* State update: x = x + K * (z - Hx) *)
x0 := x0 + K0 * Innovation;          (* Update position *)
x1 := x1 + K1 * Innovation;          (* Update velocity *)

(* Covariance update: P = (I - K * H) * P *)
(* Compute each element *)
Temp := P00;                         (* Store current P00 *)
P00 := (1.0 - K0) * P00;
P01 := (1.0 - K0) * P01;
P10 := P01;                          (* Symmetric *)
P11 := P11 - K1 * P01;

(* Step 6: Output Results *)
(* Set estimated position and velocity *)
EstPosition := x0;
EstVelocity := x1;
ValidOutput := TRUE;

(* Step 7: Safety Check *)
(* Ensure covariances remain positive and outputs are reasonable *)
IF P00 <= 0.0 OR P11 <= 0.0 OR ABS(EstPosition) > 1.0E6 OR ABS(EstVelocity) > 1.0E3 THEN
    ValidOutput := FALSE;
    EstPosition := 0.0;
    EstVelocity := 0.0;
END_IF;

(* Step 8: Update Previous State *)
(* Store current measurement for next cycle *)
PrevPosition := MeasuredPosition;

END_FUNCTION_BLOCK
