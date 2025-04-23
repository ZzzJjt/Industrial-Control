**Kalman Filter:**
Implement a self-contained function block in IEC 61131-3 structured text to create a Kalman filter for estimating the position and velocity of an autonomous transport vehicle. Include configurable parameters for noise covariance and initial conditions.

**C-A-R-E:**

🟥 C (Context) – Background Situation

In autonomous transport systems, accurate estimation of position and velocity is critical for navigation, obstacle avoidance, and smooth control. However, real-world sensor data is often noisy and unreliable. Implementing a Kalman filter allows for real-time fusion of noisy measurements and predictive models, yielding more accurate state estimates. While Kalman filters are common in embedded systems, their implementation on PLC platforms using IEC 61131-3 Structured Text requires a simplified, scan-cycle-safe structure due to the lack of native matrix operations.

⸻

🟩 A (Action) – Task to Perform

Develop a self-contained function block in Structured Text that implements a discrete-time Kalman filter to estimate an autonomous vehicle’s position and velocity. The function block must:
	•	Accept inputs including a measured position, process noise covariance (Q), measurement noise covariance (R), and initial state conditions
	•	Perform both prediction and correction (update) steps of the Kalman filter using a 2-state scalar form
	•	Operate without recursion or matrix libraries, relying on basic scalar arithmetic
	•	Be modular, scan-cycle-compatible, and reusable across different automation projects

⸻

🟨 R (Result) – Expected Outcome

The result should be a robust, efficient function block that provides filtered estimates of position and velocity. It should improve system stability, tracking accuracy, and reduce the impact of measurement noise. The block will be easy to configure through tunable noise parameters and suitable for use in both simulation and real-time industrial applications.

⸻

🟦 E (Example) – Concrete Illustration

Sample internal logic:

// Prediction
x_est_pos := x_est_pos + DeltaT * x_est_vel;
p := p + Q;

// Update
k := p / (p + R);
x_est_pos := x_est_pos + k * (MeasuredPosition - x_est_pos);
x_est_vel := (x_est_pos - x_est_prev) / DeltaT;
p := (1 - k) * p;

Inputs:
	•	MeasuredPosition : REAL
	•	Q, R : REAL
	•	DeltaT : REAL
	•	Reset : BOOL

Outputs:
	•	EstimatedPosition : REAL
	•	EstimatedVelocity : REAL
	•	KalmanGain : REAL (optional)
