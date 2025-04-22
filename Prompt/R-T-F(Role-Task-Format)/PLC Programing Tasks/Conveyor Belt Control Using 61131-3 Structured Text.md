**Conveyor Belt Control Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text program (not a function block) to control a conveyor belt system with three stations, where each station allows a user to stop the conveyor. The system should automatically start and stop based on input from five sensors that detect the presence of objects on the conveyor. The conveyor belt speed must be maintained at 2 meters per second. The program should manage both manual and automatic control modes while ensuring safe and efficient operation.

Implement logic that prioritizes safety by ensuring the conveyor stops if any station triggers a stop command or if an object is not detected by the sensors.

**R-T-F:**

🟥 R (Role) – Your Role

You are a PLC programmer or control engineer responsible for implementing a conveyor belt control program using IEC 61131-3 Structured Text. Your program must ensure both automatic object-based control and manual station-based override, prioritizing safety and consistent operation across all modes.

⸻

🟩 T (Task) – What You Need to Do

Write a self-contained Structured Text program (not a function block) to control a conveyor belt system with the following requirements:
	•	Inputs:
	•	StationStop1, StationStop2, StationStop3 : BOOL — manual stop buttons
	•	Sensor1, Sensor2, Sensor3, Sensor4, Sensor5 : BOOL — object detection sensors
	•	AutoMode : BOOL — when TRUE, system runs based on sensors
	•	ManualMode : BOOL — when TRUE, system runs manually (unless stopped)
	•	Outputs:
	•	ConveyorRunning : BOOL — TRUE if the conveyor is operating
	•	Conveyor speed is fixed at 2 m/s (logic only, no motor control required)
	•	Control Logic:
	1.	If any StationStopX input is TRUE, immediately stop the conveyor
	2.	If AutoMode = TRUE, run the conveyor only if all 5 sensors are TRUE
	3.	If ManualMode = TRUE, run the conveyor unless a station stop is active
	4.	Default state: Conveyor is stopped

⸻

🟧 F (Format) – Expected Output

Deliver a clear Structured Text implementation, such as:
VAR
    StationStop1, StationStop2, StationStop3 : BOOL;
    Sensor1, Sensor2, Sensor3, Sensor4, Sensor5 : BOOL;
    AutoMode, ManualMode : BOOL;
    ConveyorRunning : BOOL;
    ConveyorSpeed : REAL := 2.0; // meters per second, logic only
END_VAR

IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE;
ELSIF AutoMode THEN
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE;
    END_IF;
ELSIF ManualMode THEN
    ConveyorRunning := TRUE;
ELSE
    ConveyorRunning := FALSE; // Fallback if no mode selected
END_IF;
This program must be:
	•	Self-contained and readable
	•	Safe by design: manual stops override everything
	•	Easily expandable (e.g., add start/reset logic or HMIs)
	•	IEC 61131-3 compliant, suitable for industrial conveyor systems
