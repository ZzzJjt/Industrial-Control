**Pneumatic System Control Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text program (not a function block) to control a pneumatic system with a control loop frequency of 100 ms. The system should regulate the flow of air to actuators based on input signals, maintaining a flow rate of 50 standard liters per minute. The system must also ensure that the pressure remains within the specified range of 5.5 to 6 bar. Implement safety checks to handle any deviations in flow or pressure and ensure efficient and reliable operation under varying load conditions.

**R-T-F:**

🟥 R (Role) – Your Role

You are an industrial automation engineer responsible for writing a Structured Text (ST) control program in compliance with IEC 61131-3 standards. Your task is to develop a reliable control strategy for a pneumatic system operating under real-time constraints.

⸻

🟩 T (Task) – What You Need to Do

Develop a self-contained Structured Text program (not a function block) that meets the following requirements:
	1.	Control Loop Frequency:
	•	The control logic must execute on a 100 ms cycle.
	2.	Flow Control:
	•	Use the sensor input FlowInput (REAL) to maintain a flow rate of 50 standard liters per minute (SLPM).
	•	If the flow is below the setpoint, open the valve; otherwise, hold or close it.
	3.	Pressure Monitoring:
	•	Use PressureInput (REAL) to ensure pressure stays within 5.5 to 6.0 bar.
	•	If pressure is out of range, activate PressureReliefValve and raise a PressureError flag.
	4.	Safety and Fault Detection:
	•	If the absolute deviation between actual flow and setpoint exceeds 5.0 SLPM, set FlowError := TRUE.
	•	Ensure that both FlowError and PressureError can be used for system-level fault handling or alarms.

⸻

🟧 F (Format) – Expected Output

You are expected to provide a clean and modular IEC 61131-3 Structured Text program, such as:

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

// Flow control
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE;
ELSE
    FlowValveOutput := FALSE;
END_IF

// Pressure safety logic
IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE;
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF

// Flow deviation detection
IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF

This ensures:
	•	Stable regulation of airflow
	•	Pressure maintained within safe limits
	•	Real-time responsiveness in each 100 ms cycle
	•	Fault flags for safety handling or external diagnostics
