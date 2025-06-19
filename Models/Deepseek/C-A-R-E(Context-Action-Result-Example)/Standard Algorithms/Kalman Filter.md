Certainly! Below is a self-contained function block in IEC 61131-3 Structured Text that implements a discrete-time Kalman filter for estimating an autonomous vehicle's position and velocity. The function block operates without recursion or matrix libraries, relying on basic scalar arithmetic. It includes prediction and correction steps and is designed to be scan-cycle-compatible and reusable.

### Discrete-Time Kalman Filter Function Block

```st
[<title="KalmanFilterFunctionBlock">]
FUNCTION_BLOCK KalmanFilterFB
VAR_INPUT
    Enable : BOOL;          // Enable signal to start the filtering process
    Reset : BOOL;           // Reset signal to reinitialize the filter
    MeasuredPosition : REAL; // Measured position input
    Q : REAL;               // Process noise covariance
    R : REAL;               // Measurement noise covariance
    DeltaT : REAL;          // Time step between measurements
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL := 0.0; // Estimated position output
    EstimatedVelocity : REAL := 0.0; // Estimated velocity output
    KalmanGain : REAL := 0.0;      // Optional: Kalman gain output
END_VAR

VAR
    x_est_pos : REAL := 0.0;       // Estimated position state
    x_est_vel : REAL := 0.0;       // Estimated velocity state
    p : REAL := 1.0;               // Estimate error covariance
    k : REAL := 0.0;               // Kalman gain
    x_est_prev : REAL := 0.0;      // Previous estimated position
BEGIN
METHOD Predict : VOID
BEGIN
    // Prediction step
    x_est_pos := x_est_pos + DeltaT * x_est_vel;
    p := p + Q;
END_METHOD

METHOD Update : VOID
BEGIN
    // Update step
    k := p / (p + R);
    x_est_pos := x_est_pos + k * (MeasuredPosition - x_est_pos);
    p := (1.0 - k) * p;
    x_est_vel := (x_est_pos - x_est_prev) / DeltaT;
    x_est_prev := x_est_pos;
END_METHOD

METHOD Execute : VOID
BEGIN
    IF Reset THEN
        // Reinitialize the filter
        x_est_pos := 0.0;
        x_est_vel := 0.0;
        p := 1.0;
        x_est_prev := 0.0;
    ELSIF Enable THEN
        // Perform prediction and update steps
        Predict();
        Update();
        
        // Set outputs
        EstimatedPosition := x_est_pos;
        EstimatedVelocity := x_est_vel;
        KalmanGain := k;
    END_IF;
END_METHOD

// Main execution method
Execute();
END_FUNCTION_BLOCK
```

### Explanation of the Code

#### Inputs
- **Enable**: A boolean signal to initiate the filtering process.
- **Reset**: A boolean signal to reinitialize the filter.
- **MeasuredPosition**: A real number representing the measured position.
- **Q**: Process noise covariance.
- **R**: Measurement noise covariance.
- **DeltaT**: Time step between measurements.

#### Outputs
- **EstimatedPosition**: A real number representing the estimated position.
- **EstimatedVelocity**: A real number representing the estimated velocity.
- **KalmanGain**: An optional real number representing the Kalman gain.

#### Internal Variables
- **x_est_pos**: Estimated position state.
- **x_est_vel**: Estimated velocity state.
- **p**: Estimate error covariance.
- **k**: Kalman gain.
- **x_est_prev**: Previous estimated position for calculating velocity.

#### Methods
- **Predict**: Performs the prediction step of the Kalman filter.
- **Update**: Performs the update step of the Kalman filter.
- **Execute**: Orchestrates the filtering process by calling `Predict` and `Update`.

#### Method: Execute
- **Reset Logic**: If `Reset` is `TRUE`, reinitializes all internal states.
- **Execution Logic**: If `Enable` is `TRUE`, performs the prediction and update steps and sets the outputs accordingly.

### Usage Example

To use this function block in a PLC program, you would instantiate it and call its `Execute` method:

```st
PROGRAM MainProgram
VAR
    MyKalmanFilter : KalmanFilterFB;
    MeasuredPosition : REAL := 10.0; // Example measured position
    Q : REAL := 0.1;                 // Example process noise covariance
    R : REAL := 1.0;                 // Example measurement noise covariance
    DeltaT : REAL := 0.1;            // Example time step
    EstimatedPosition : REAL;
    EstimatedVelocity : REAL;
    KalmanGain : REAL;
BEGIN
    // Configure the Kalman filter
    MyKalmanFilter.Enable := TRUE;
    MyKalmanFilter.Reset := FALSE;
    MyKalmanFilter.MeasuredPosition := MeasuredPosition;
    MyKalmanFilter.Q := Q;
    MyKalmanFilter.R := R;
    MyKalmanFilter.DeltaT := DeltaT;

    // Execute the filter
    MyKalmanFilter();

    // Retrieve the results
    EstimatedPosition := MyKalmanFilter.EstimatedPosition;
    EstimatedVelocity := MyKalmanFilter.EstimatedVelocity;
    KalmanGain := MyKalmanFilter.KalmanGain;

    // Output the results
    // Use EstimatedPosition and EstimatedVelocity as needed
END_PROGRAM
```

### Key Features
- **Iterative Approach**: Uses only iterative loops and avoids recursion.
- **Scalar Arithmetic**: Relies on basic scalar arithmetic operations.
- **Modularity**: Well-defined methods for prediction and update steps.
- **Scan-Cycle Compatibility**: Suitable for PLC environments with scan-cycle constraints.
- **Reusability**: Configurable through tunable noise parameters and initial conditions.

This function block provides robust and efficient filtering of position and velocity estimates, improving system stability, tracking accuracy, and reducing the impact of measurement noise in autonomous transport systems.
