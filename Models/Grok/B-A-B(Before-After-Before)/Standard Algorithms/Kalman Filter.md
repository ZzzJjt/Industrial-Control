(* IEC 61131-3 Structured Text: Kalman Filter Function Block *)
(* Purpose: Estimates position and velocity of a vehicle using a 2-state Kalman filter *)

FUNCTION_BLOCK KalmanFilter
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run filter *)
    Reset : BOOL;                   (* TRUE to reset estimates and covariance *)
    MeasuredPosition : REAL;         (* Measured position in meters *)
    Q : REAL;                       (* Process noise covariance, e.g., 0.01 *)
    R : REAL;                       (* Measurement noise covariance, e.g., 1.0 *)
    DeltaT : REAL;                  (* Sample time in seconds, e.g., 0.1 *)
END_VAR
VAR_OUTPUT
    EstimatedPosition : REAL;        (* Filtered position estimate in meters *)
    EstimatedVelocity : REAL;        (* Filtered velocity estimate in m/s *)
    KalmanGain : REAL;              (* Kalman gain for position update *)
    Error : BOOL;                   (* TRUE if input validation fails *)
END_VAR
VAR
    (* State Estimates *)
    X_hat : REAL := 0.0;            (* Estimated position *)
    V_hat : REAL := 0.0;            (* Estimated velocity *)
    
    (* Error Covariance Matrix P *)
    P_11 : REAL := 1.0;             (* Variance of position *)
    P_12 : REAL := 0.0;             (* Covariance of position and velocity *)
    P_21 : REAL := 0.0;             (* Covariance of velocity and position *)
    P_22 : REAL := 1.0;             (* Variance of velocity *)
    
    (* Temporary Variables for Prediction *)
    X_pred : REAL;                  (* Predicted position *)
    V_pred : REAL;                  (* Predicted velocity *)
    P_11_pred : REAL;               (* Predicted P_11 *)
    P_12_pred : REAL;               (* Predicted P_12 *)
    P_21_pred : REAL;               (* Predicted P_21 *)
    P_22_pred : REAL;               (* Predicted P_22 *)
    
    (* Kalman Gain *)
    K_pos : REAL;                   (* Gain for position *)
    K_vel : REAL;                   (* Gain for velocity *)
    
    (* Initialization Flag *)
    Initialized : BOOL := FALSE;    (* TRUE after first run or reset *)
END_VAR

(* Main Logic *)
IF Reset THEN
    (* Reset estimates and covariance *)
    Initialized := FALSE;
END_IF;

IF NOT Enable THEN
    (* Disable filter, maintain outputs *)
    EstimatedPosition := X_hat;
    EstimatedVelocity := V_hat;
    KalmanGain := K_pos;
    Error := FALSE;
ELSE
    (* Input Validation *)
    IF Q <= 0.0 OR R <= 0.0 OR DeltaT <= 0.0 THEN
        (* Invalid parameters *)
        Error := TRUE;
        EstimatedPosition := 0.0;
        EstimatedVelocity := 0.0;
        KalmanGain := 0.0;
    ELSE
        Error := FALSE;
        
        (* Initialize on first run or after reset *)
        IF NOT Initialized THEN
            X_hat := MeasuredPosition;  (* Start with measurement *)
            V_hat := 0.0;               (* Assume zero initial velocity *)
            P_11 := 1.0;                (* Initial position uncertainty *)
            P_12 := 0.0;
            P_21 := 0.0;
            P_22 := 1.0;                (* Initial velocity uncertainty *)
            Initialized := TRUE;
        END_IF;
        
        (* Step 1: Predict *)
        (* State prediction: x(k+1|k) = F * x(k|k) *)
        X_pred := X_hat + DeltaT * V_hat;  (* Position = prev position + dt * velocity *)
        V_pred := V_hat;                   (* Velocity unchanged *)
        
        (* Covariance prediction: P(k+1|k) = F * P(k|k) * F^T + Q *)
        P_11_pred := P_11 + DeltaT * (P_21 + P_12 + DeltaT * P_22) + Q;
        P_12_pred := P_12 + DeltaT * P_22;
        P_21_pred := P_21 + DeltaT * P_22;
        P_22_pred := P_22 + Q;
        
        (* Step 2: Update *)
        (* Innovation: z(k) - H * x(k+1|k) *)
        (* H = [1, 0], so measurement is position *)
        Innovation := MeasuredPosition - X_pred;
        
        (* Innovation covariance: S = H * P(k+1|k) * H^T + R *)
        S := P_11_pred + R;
        
        (* Kalman Gain: K = P(k+1|k) * H^T * S^-1 *)
        K_pos := P_11_pred / S;
        K_vel := P_21_pred / S;
        
        (* Update state: x(k|k) = x(k+1|k) + K * innovation *)
        X_hat := X_pred + K_pos * Innovation;
        V_hat := V_pred + K_vel * Innovation;
        
        (* Update covariance: P(k|k) = (I - K * H) * P(k+1|k) *)
        P_11 := (1.0 - K_pos) * P_11_pred;
        P_12 := (1.0 - K_pos) * P_12_pred;
        P_21 := P_21_pred - K_vel * P_11_pred;
        P_22 := P_22_pred - K_vel * P_12_pred;
        
        (* Update Outputs *)
        EstimatedPosition := X_hat;
        EstimatedVelocity := V_hat;
        KalmanGain := K_pos;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Implements a 2-state Kalman filter for vehicle position and velocity estimation.
   - Model:
     - States: Position (x), Velocity (v).
     - Dynamics: x(k+1) = x(k) + dt * v(k), v(k+1) = v(k) + noise.
     - Measurement: Position only, z(k) = x(k) + noise.
   - Inputs:
     - Enable: Runs filter when TRUE.
     - Reset: Resets estimates and covariance when TRUE.
     - MeasuredPosition: REAL, measured position from sensor (m).
     - Q: REAL, process noise covariance (e.g., 0.01).
     - R: REAL, measurement noise covariance (e.g., 1.0).
     - DeltaT: REAL, sample time (s, e.g., 0.1).
   - Outputs:
     - EstimatedPosition: Filtered position estimate (m).
     - EstimatedVelocity: Filtered velocity estimate (m/s).
     - KalmanGain: Gain for position update (diagnostic).
     - Error: TRUE if Q, R, or DeltaT are invalid (<= 0).
   - Logic:
     - Predict: Advances state (x, v) and covariance using constant velocity model.
     - Update: Computes Kalman gain, corrects state with measurement, updates covariance.
   - Safety:
     - Validates Q, R, DeltaT to prevent division by zero or invalid operations.
     - Initializes with measured position and zero velocity for stability.
     - Resets outputs on disable or error for safe state.
   - Usage:
     - Suitable for navigation, control, or safety in autonomous vehicles.
     - Reusable in scan-cycle-driven PLCs (e.g., CODESYS, TwinCAT).
   - Maintenance:
     - Scalar math avoids matrix libraries, ensuring PLC compatibility.
     - Comments and modular design aid debugging.
     - KalmanGain and Error support diagnostics via HMI/logging.
   - Platform Notes:
     - Uses REAL for calculations; LREAL can be used for higher precision.
     - Assumes single-cycle execution; multi-cycle logic can be added for complex systems.
*)
END_FUNCTION_BLOCK
