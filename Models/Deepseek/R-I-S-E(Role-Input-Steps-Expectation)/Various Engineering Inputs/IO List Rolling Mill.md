Certainly! Below is a detailed I/O list for a rolling mill system in a steel production plant. This list includes 30 key control and monitoring signals, formatted as a structured table suitable for use in Excel, documentation, or PLC tag import.

### Rolling Mill I/O Specification

| Signal Name                | Input/Output | Signal Tagnumber | Analog/Digital | Engineering Unit | Range               | Setpoint          | P&ID Reference |
|----------------------------|--------------|------------------|----------------|--------------------|---------------------|-------------------|----------------|
| RollSpeed_Front            | Input        | RM1001           | Analog         | m/min              | 0 to 150            | N/A               | PID-RM-FRONT     |
| RollSpeed_Back             | Input        | RM1002           | Analog         | m/min              | 0 to 150            | N/A               | PID-RM-BACK      |
| MotorTorque_Front          | Input        | RM1003           | Analog         | kW                 | 0 to 500            | N/A               | PID-MTR-FRONT    |
| MotorTorque_Back           | Input        | RM1004           | Analog         | kW                 | 0 to 500            | N/A               | PID-MTR-BACK     |
| CoolingSystem_Actuation    | Output       | RM1005           | Digital        | %                  | 0 to 100            | N/A               | PID-COOLING      |
| GapFeedback                | Input        | RM1006           | Analog         | mm                 | 0 to 10             | N/A               | PID-GAP          |
| TensionControl             | Input        | RM1007           | Analog         | kN                 | 0 to 50             | N/A               | PID-TENSION      |
| OilPressure_Hydraulic      | Input        | RM1008           | Analog         | bar                | 0 to 20             | N/A               | PID-OILPRESSURE  |
| OilTemperature_Hydraulic   | Input        | RM1009           | Analog         | °C                 | -20 to 100          | N/A               | PID-OILTEMP      |
| WaterFlow_Cooling          | Input        | RM1010           | Analog         | L/min              | 0 to 100            | N/A               | PID-WATERFLOW    |
| WaterTemp_Cooling          | Input        | RM1011           | Analog         | °C                 | 0 to 100            | N/A               | PID-WATERTEMP    |
| MotorCurrent_Front         | Input        | RM1012           | Analog         | A                  | 0 to 1000           | N/A               | PID-CURRENT-FRONT|
| MotorCurrent_Back          | Input        | RM1013           | Analog         | A                  | 0 to 1000           | N/A               | PID-CURRENT-BACK |
| BearingTemp_Front          | Input        | RM1014           | Analog         | °C                 | 0 to 150            | N/A               | PID-BEARING-FRNT |
| BearingTemp_Back           | Input        | RM1015           | Analog         | °C                 | 0 to 150            | N/A               | PID-BEARING-BACK |
| GearboxOilLevel            | Input        | RM1016           | Analog         | mm                 | 0 to 500            | N/A               | PID-GEARBOXLEVEL |
| GearboxOilTemp             | Input        | RM1017           | Analog         | °C                 | -20 to 100          | N/A               | PID-GEARBOXTMP   |
| LubricationPressure        | Input        | RM1018           | Analog         | bar                | 0 to 20             | N/A               | PID-LUBRICATION  |
| LubricationFlow            | Input        | RM1019           | Analog         | L/min              | 0 to 100            | N/A               | PID-LUBRICATIONF |
| HydraulicValve_Position    | Input        | RM1020           | Analog         | %                  | 0 to 100            | N/A               | PID-HYDRAULICV   |
| BeltSpeed_Main             | Input        | RM1021           | Analog         | m/min              | 0 to 150            | N/A               | PID-BELTSPEED    |
| BeltTension_Main           | Input        | RM1022           | Analog         | kN                 | 0 to 50             | N/A               | PID-BELTTENSION  |
| MaterialThickness_Input    | Input        | RM1023           | Analog         | mm                 | 0 to 50             | N/A               | PID-THICKNESS-IN   |
| MaterialWidth_Input        | Input        | RM1024           | Analog         | mm                 | 0 to 500            | N/A               | PID-WIDTH-IN     |
| MaterialLength_Output      | Output       | RM1025           | Analog         | m                  | 0 to 1000           | N/A               | PID-LENGTH-OUT   |
| MaterialQuality_Check      | Input        | RM1026           | Digital        | Bool               | 0/1               | N/A               | PID-QUALITY      |
| EmergencyStop_Button       | Input        | RM1027           | Digital        | Bool               | 0/1               | N/A               | PID-EMERGENCY    |
| PowerSupply_Status         | Input        | RM1028           | Digital        | Bool               | 0/1               | N/A               | PID-PWRSTATUS    |
| Maintenance_Mode_Switch    | Input        | RM1029           | Digital        | Bool               | 0/1               | N/A               | PID-MAINTMODE    |
| SafetyInterlock_Roll       | Input        | RM1030           | Digital        | Bool               | 0/1               | N/A               | PID-SAFETYROLL   |

### Explanation of Columns

- **Signal Name**: Descriptive name of the signal.
- **Input/Output**: Indicates whether the signal is an input (sensor) or output (actuator).
- **Signal Tagnumber**: Unique identifier for each signal.
- **Analog/Digital**: Specifies the type of signal (analog or digital).
- **Engineering Unit**: The unit of measurement for the signal.
- **Range**: The expected range of the signal values.
- **Setpoint**: The target value for control signals (if applicable).
- **P&ID Reference**: Reference to the Process and Instrumentation Diagram (P&ID) for traceability.

This I/O specification provides all necessary metadata for PLC configuration, field wiring, and HMI integration, ensuring clarity and consistency throughout the project.
