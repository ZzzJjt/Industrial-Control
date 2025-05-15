Certainly! Below is a complete implementation of the IEC 61131-3 Structured Text (ST) program to control a conveyor belt system with the specified requirements. The program includes logic for both manual and automatic modes, ensures safety by prioritizing stop signals from operator stations, and handles sensor inputs to maintain object tracking.

### Implementation



### Explanation of the Program

1. **Inputs**:
   - `StationStop1`, `StationStop2`, `StationStop3`: Boolean inputs representing stop signals from three operator stations.
   - `Sensor1`, `Sensor2`, `Sensor3`, `Sensor4`, `Sensor5`: Boolean inputs representing the presence of items detected by five sensors on the conveyor belt.
   - `AutoMode`: Boolean input indicating that the conveyor should operate in automatic mode.
   - `ManualMode`: Boolean input indicating that the conveyor should operate in manual mode.

2. **Outputs**:
   - `ConveyorRunning`: Boolean output controlling whether the conveyor belt is running.

3. **Internal Variables**:
   - `AllSensorsActive`: A boolean variable that stores the logical AND result of all sensor states, ensuring all sensors detect objects in automatic mode.

4. **Control Logic**:
   - **Safety Stops**: If any station sends a stop signal (`StationStop1`, `StationStop2`, or `StationStop3`), the conveyor is immediately stopped (`ConveyorRunning := FALSE`).
   - **Automatic Mode**:
     - Checks if all five sensors detect objects (`AllSensorsActive := Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5`).
     - If all sensors are active, the conveyor runs (`ConveyorRunning := TRUE`).
     - If any sensor fails to detect an object, the conveyor stops (`ConveyorRunning := FALSE`).
   - **Manual Mode**:
     - The conveyor runs regardless of the sensor states (`ConveyorRunning := TRUE`).
   - **Default Case**:
     - If neither `AutoMode` nor `ManualMode` is selected, the conveyor does not run (`ConveyorRunning := FALSE`).

5. **Comments**:
   - Detailed comments explain the purpose and functionality of each part of the control logic, ensuring clarity and maintainability.

### Practical Use Case

Given the following inputs:
```st
StationStop1 := FALSE;
StationStop2 := FALSE;
StationStop3 := FALSE;
Sensor1 := TRUE;
Sensor2 := TRUE;
Sensor3 := TRUE;
Sensor4 := TRUE;
Sensor5 := TRUE;
AutoMode := TRUE;
ManualMode := FALSE;
```

The `ConveyorControl` program will produce:
- `ConveyorRunning := TRUE` (all sensors active in automatic mode)

If a station sends a stop signal:
```st
StationStop1 := TRUE;
StationStop2 := FALSE;
StationStop3 := FALSE;
Sensor1 := TRUE;
Sensor2 := TRUE;
Sensor3 := TRUE;
Sensor4 := TRUE;
Sensor5 := TRUE;
AutoMode := TRUE;
ManualMode := FALSE;
```

The `ConveyorControl` program will produce:
- `ConveyorRunning := FALSE` (emergency stop takes priority)

In manual mode:
```st
StationStop1 := FALSE;
StationStop2 := FALSE;
StationStop3 := FALSE;
Sensor1 := TRUE;
Sensor2 := FALSE;
Sensor3 := TRUE;
Sensor4 := TRUE;
Sensor5 := TRUE;
AutoMode := FALSE;
ManualMode := TRUE;
```

The `ConveyorControl` program will produce:
- `ConveyorRunning := TRUE` (conveyor runs regardless of sensor states in manual mode)

This implementation ensures safe and precise control over the conveyor belt system, handling both manual and automatic modes, prioritizing safety, and maintaining a constant speed logic. It is structured for readability, scalability, and compliance with standards, making it suitable for deployment in real-world manufacturing or packaging lines.
