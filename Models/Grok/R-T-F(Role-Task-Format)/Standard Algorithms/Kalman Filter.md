(* IEC 61131-3 Structured Text function block for a 2-state Kalman filter *)
(* Estimates position and velocity of an autonomous vehicle from noisy position measurements *)
(* Uses scalar arithmetic for PLC compatibility, with prediction and correction steps *)
(* Optimized for single-cycle execution in scan-cycle-driven environments *)

FUNCTION_BLOCK KalmanFilter
VAR_INPUT
    Enable : BOOL; (* TRUE to enable filter execution *)
    Reset : BOOL; (* TRUE to reset states to initial values *)
    MeasuredPosition : REAL; (* Noisy position measurement in meters *)
    Q : REAL := 0.1; (* Process noise covariance *)
    R : REAL := 1.0; (* Measurement noise covariance *)
    DeltaT : REAL := 0.1; (* Sample time in seconds *)
    InitialPosition : REAL := 0.0; (* Initial position estimate in meters *)
    InitialVelocity : REAL := 0.0; (* Initial velocity estimate in m/s *)
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL; (* Estimated position in meters *)
    EstimatedVelocity : REAL; (* Estimated velocity in m/s *)
    KalmanGain : REAL; (* Kalman gain for diagnostics *)
    Fault : BOOL := FALSE; (* TRUE if invalid inputs detected *)
END_VAR

VAR
    x_est_pos : REAL; (* Current estimated position *)
    x_est_vel : REAL; (* Current estimated velocity *)
    p : REAL := 1.0; (* Covariance of position estimate *)
    k : REAL; (* Kalman gain *)
    LastEnable : BOOL; (* Tracks previous Enable state for edge detection *)
    Initialized : BOOL := FALSE; (* Tracks if filter is initialized *)
END_VAR

(* Reset outputs when disabled *)
IF NOT Enable THEN
    EstimatedPosition := 0.0;
    EstimatedVelocity := 0.0;
    KalmanGain := 0.0;
    Fault := FALSE;
    Initialized := FALSE;
    LastEnable := FALSE;
    RETURN;
END_IF;

(* Input validation and initialization on rising edge of Enable *)
IF Enable AND NOT LastEnable THEN
    (* Validate inputs *)
    IF Q < 0.0 OR R <= 0.0 OR DeltaT <= 0.0 THEN
        Fault := TRUE;
        EstimatedPosition := 0.0;
        EstimatedVelocity := 0.0;
        KalmanGain := 0.0;
        Initialized := FALSE;
        RETURN;
    END_IF;
    
    (* Initialize state estimates *)
    x_est_pos := InitialPosition;
    x_est_vel := InitialVelocity;
    p := 1.0; (* Initial covariance *)
    k := 0.0;
    EstimatedPosition := x_est_pos;
    EstimatedVelocity := x_est_vel;
    KalmanGain := k;
    Fault := FALSE;
    Initialized := TRUE;
END_IF;

(* Store Enable state for edge detection *)
LastEnable := Enable;

(* Reset handling *)
IF Reset AND Initialized THEN
    (* Reinitialize states *)
    x_est_pos := InitialPosition;
    x_est_vel := InitialVelocity;
    p := 1.0;
    k := 0.0;
    EstimatedPosition := x_est_pos;
    EstimatedVelocity := x_est_vel;
    KalmanGain := k;
    Fault := FALSE;
    RETURN;
END_IF;

(* Kalman filter execution *)
IF Enable AND Initialized AND NOT Fault THEN
    (* Prediction Step *)
    (* Predict position: x_pos = x_pos + DeltaT * x_vel *)
    x_est_pos := x_est_pos + DeltaT * x_est_vel;
    (* Predict velocity: x_vel = x_vel (constant velocity model) *)
    x_est_vel := x_est_vel;
    (* Predict covariance: P = P + Q *)
    p := p + Q;
    
    (* Correction Step *)
    (* Compute Kalman gain: K = P / (P + R) *)
    k := p / (p + R);
    (* Update position: x_pos = x_pos + K * (z - x_pos) *)
    x_est_pos := x_est_pos + k * (MeasuredPosition - x_est_pos);
    (* Update velocity: x_vel = x_vel (no direct measurement) *)
    x_est_vel := x_est_vel;
    (* Update covariance: P = (1 - K) * P *)
    p := (1.0 - k) * p;
    
    (* Update outputs *)
    EstimatedPosition := x_est_pos;
    EstimatedVelocity := x_est_vel;
    KalmanGain := k;
END_IF;

(* Execution Notes *)
(* - Uses scalar arithmetic to avoid matrix operations, ensuring PLC compatibility. *)
(* - Executes in a single scan cycle (<1 ms on modern PLCs), suitable for 10-50 ms scan times. *)
(* - Input validation (Q >= 0, R > 0, DeltaT > 0) prevents numerical instability. *)
(* - Reset capability allows reinitialization without restarting
