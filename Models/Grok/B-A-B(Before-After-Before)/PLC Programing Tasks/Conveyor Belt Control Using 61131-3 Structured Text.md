(* Conveyor Belt Control Logic in IEC 61131-3 Structured Text *)
(* Purpose: Manage conveyor with dual modes (Auto/Manual), safety stops, and sensor-based operation *)

PROGRAM ConveyorControl
VAR
    (* Inputs *)
    Sensor1 : BOOL;               (* Object detection sensor 1 *)
    Sensor2 : BOOL;               (* Object detection sensor 2 *)
    Sensor3 : BOOL;               (* Object detection sensor 3 *)
    Sensor4 : BOOL;               (* Object detection sensor 4 *)
    Sensor5 : BOOL;               (* Object detection sensor 5 *)
    StationStop1 : BOOL;          (* Manual stop command from station 1 *)
    StationStop2 : BOOL;          (* Manual stop command from station 2 *)
    StationStop3 : BOOL;          (* Manual stop command from station 3 *)
    ManualMode : BOOL;            (* TRUE for manual mode *)
    AutoMode : BOOL;              (* TRUE for automatic mode *)

    (* Outputs *)
    ConveyorMotor : BOOL;         (* Controls conveyor motor (TRUE = running) *)
    ConveyorSpeed : REAL := 2.0;  (* Fixed conveyor speed in m/s *)

    (* Internal Variables *)
    ConveyorRunning : BOOL;       (* Internal state of conveyor operation *)
    AllSensorsActive : BOOL;      (* TRUE when all sensors detect objects *)
END_VAR

(* Main Control Logic *)
(* 1. Safety Check: Manual stop commands take priority *)
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE;
ELSE
    (* 2. Mode Selection and Logic *)
    (* Ensure only one mode is active; AutoMode takes precedence if both are TRUE *)
    IF AutoMode AND NOT ManualMode THEN
        (* Automatic Mode: Conveyor runs only if all sensors detect objects *)
        AllSensorsActive := Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5;
        IF AllSensorsActive THEN
            ConveyorRunning := TRUE;
        ELSE
            ConveyorRunning := FALSE;
        END_IF;
    ELSIF ManualMode AND NOT AutoMode THEN
        (* Manual Mode: Conveyor runs unless stopped by stations *)
        ConveyorRunning := TRUE;
    ELSE
        (* No mode selected or invalid state: Stop conveyor *)
        ConveyorRunning := FALSE;
    END_IF;
END_IF;

(* 3. Output Assignment *)
ConveyorMotor := ConveyorRunning;  (* Map internal state to motor output *)

(* Notes:
   - Safety: StationStop1, StationStop2, StationStop3 override all operations, ensuring immediate stop
   - Mode Exclusivity: Logic assumes AutoMode and ManualMode are mutually exclusive in normal operation
   - AutoMode: Requires all sensors (Sensor1 to Sensor5) to be TRUE for ConveyorRunning
   - ManualMode: Allows conveyor to run unless stopped by a station
   - ConveyorSpeed: Fixed at 2.0 m/s, applied via motor control (e.g., VFD parameter)
   - Scalability:
     - Add sensors by extending AllSensorsActive with new sensor inputs
     - Add stations by extending OR condition in safety check
   - Physical Integration:
     - Sensors: Photoelectric or proximity sensors for object detection
     - StationStop: Push-buttons or HMI inputs at each station
     - ConveyorMotor: Relay or VFD control for motor
     - Mode Inputs: Toggle switches or HMI for ManualMode/AutoMode
   - For maintenance: Add HMI to display ConveyorRunning, mode, and sensor/station status
*)
END_PROGRAM
