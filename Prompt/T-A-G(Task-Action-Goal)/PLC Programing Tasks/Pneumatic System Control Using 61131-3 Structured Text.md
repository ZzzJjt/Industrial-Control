**Pneumatic System Control Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text program (not a function block) to control a pneumatic system with a control loop frequency of 100 ms. The system should regulate the flow of air to actuators based on input signals, maintaining a flow rate of 50 standard liters per minute. The system must also ensure that the pressure remains within the specified range of 5.5 to 6 bar. Implement safety checks to handle any deviations in flow or pressure and ensure efficient and reliable operation under varying load conditions.

**T-A-G:**

🟥 T (Task) – What You Need to Do

Develop a self-contained IEC 61131-3 Structured Text (ST) program to control a pneumatic system. The system must maintain a stable flow rate and safe pressure range while executing on a 100 ms control cycle. You must also implement safety checks for fault conditions.

⸻

🟩 A (Action) – How to Do It
	1.	Flow Control Logic:
	•	Use FlowInput (REAL) to monitor the current airflow rate.
	•	Maintain a flow rate of 50 SLPM by controlling FlowValveOutput (BOOL).
	•	Open the valve if FlowInput is below the setpoint; otherwise, hold or close.
	2.	Pressure Monitoring and Protection:
	•	Use PressureInput (REAL) to ensure the pressure stays between 5.5 and 6.0 bar.
	•	If pressure is out of bounds, activate PressureReliefValve (BOOL) and raise a PressureError flag.
	3.	Deviation & Fault Detection:
	•	If the absolute difference between FlowInput and FlowSetpoint is greater than 5.0, set FlowError := TRUE.
	4.	Timing Requirement:
	•	Ensure the logic runs every 100 ms, complying with the system’s real-time requirements.

⸻

🟦 G (Goal) – What You Want to Achieve

Your program should:
	•	Maintain airflow stability at 50 SLPM
	•	Keep system pressure safely between 5.5 and 6.0 bar
	•	Detect and report anomalies in flow or pressure
	•	Run reliably in a real-time industrial control loop, ready for integration with alarms, HMI displays, or safety logic

⸻

✅ Example Code Snippet:

VAR
    FlowInput : REAL;
    PressureInput : REAL;
    FlowSetpoint : REAL := 50.0;
    MinPressure : REAL := 5.5;
    MaxPressure : REAL := 6.0;

    FlowValveOutput : BOOL := FALSE;
    PressureReliefValve : BOOL := FALSE;
    FlowError : BOOL := FALSE;
    PressureError : BOOL := FALSE;
END_VAR

// Flow control logic
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE;
ELSE
    FlowValveOutput := FALSE;
END_IF

// Pressure check logic
IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE;
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF

// Flow deviation check
IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF
