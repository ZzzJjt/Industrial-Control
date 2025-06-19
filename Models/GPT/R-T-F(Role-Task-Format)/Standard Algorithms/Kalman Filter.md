FUNCTION_BLOCK FB_KalmanFilter
VAR_INPUT
    MeasuredPosition : REAL; // Noisy position input
    Q : REAL;                // Process noise covariance
    R : REAL;                // Measurement noise covariance
    DeltaT : REAL;           // Time interval between updates
    Reset : BOOL;            // Resets internal states
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL; // Output: filtered position
    EstimatedVelocity : REAL; // Output: filtered velocity
    KalmanGain : REAL;        // Optional: Kalman gain for position
END_VAR

VAR
    // State estimates
    x_pos : REAL := 0.0;       // Estimated position
    x_vel : REAL := 0.0;       // Estimated velocity

    // Covariance terms (2x2 matrix, scalar representation)
    p_00 : REAL := 1.0;        // Cov(position, position)
    p_01 : REAL := 0.0;        // Cov(position, velocity)
    p_10 : REAL := 0.0;        // Cov(velocity, position)
    p_11 : REAL := 1.0;        // Cov(velocity, velocity)

    // Intermediate variables
    y : REAL;                  // Measurement residual
    s : REAL;                  // Innovation covariance
    k_0 : REAL;                // Kalman gain for position
    k_1 : REAL;                // Kalman gain for velocity
    dt : REAL;                 // Cached DeltaT
    q : REAL;                  // Cached Q
    r : REAL;                  // Cached R
END_VAR

// === Reset logic ===
IF Reset THEN
    x_pos := 0.0;
    x_vel := 0.0;

    p_00 := 1.0;
    p_01 := 0.0;
    p_10 := 0.0;
    p_11 := 1.0;
END_IF

// Cache valid values for DeltaT, Q, R
dt := MAX(DeltaT, 0.0001); // Prevent divide-by-zero
q := MAX(Q, 0.0);
r := MAX(R, 0.0001);

// === PREDICTION STEP ===
// Predict next state using motion model:
// x_k|k-1 = A * x_k-1
x_pos := x_pos + dt * x_vel;

// Predict covariance:
// P = A * P * A^T + Q
// A = [1 dt; 0 1]
p_00 := p_00 + dt * (p_10 + p_01) + dt * dt * p_11 + q;
p_01 := p_01 + dt * p_11;
p_10 := p_10 + dt * p_11;
p_11 := p_11 + q;

// === UPDATE STEP ===
// Compute residual: y = z - H*x
y := MeasuredPosition - x_pos;

// Innovation covariance: s = H * P * H^T + R
// H = [1 0]
s := p_00 + r;

// Compute Kalman gain: K = P * H^T / s
k_0 := p_00 / s;
k_1 := p_10 / s;

// Update state estimate: x = x + K * y
x_pos := x_pos + k_0 * y;
x_vel := x_vel + k_1 * y;

// Update covariance: P = (I - K*H) * P
p_00 := p_00 - k_0 * p_00;
p_01 := p_01 - k_0 * p_01;
p_10 := p_10 - k_1 * p_00; // update using old p_00
p_11 := p_11 - k_1 * p_01;

// === OUTPUTS ===
EstimatedPosition := x_pos;
EstimatedVelocity := x_vel;
KalmanGain := k_0;
