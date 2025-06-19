
### Rolling Mill I/O List

| Signal Name                | Input/Output | Signal Tag Number | Analog/Digital | Engineering Unit | Range                  | Setpoint              | P&ID Reference |
|----------------------------|--------------|-------------------|------------------|--------------------|------------------------|---------------------|----------------|
| Roll Speed Feedback        | Input        | AI-001            | Analog           | RPM                | 0 to 1000 RPM          | N/A                 | PID-RollSpeed    |
| Hydraulic Pressure         | Input        | AI-002            | Analog           | Bar                | 0 to 50 Bar            | N/A                 | PID-Hydraulic    |
| Motor Current              | Input        | AI-003            | Analog           | Amps               | 0 to 1000 Amps         | N/A                 | PID-MotorCurrent   |
| Strip Tension Measurement  | Input        | AI-004            | Analog           | kN                 | 0 to 50 kN             | N/A                 | PID-Tension      |
| Oil Temperature            | Input        | AI-005            | Analog           | °C                 | 0 to 150 °C            | N/A                 | PID-OilTemp      |
| Bearing Temperature        | Input        | AI-006            | Analog           | °C                 | 0 to 120 °C            | N/A                 | PID-BearingTemp    |
| Cooling Valve Control      | Output       | AO-001            | Analog           | % Opening          | 0% to 100%             | 50%                 | PID-CoolingValve   |
| Lubrication Pump Speed     | Output       | AO-002            | Analog           | RPM                | 0 to 1000 RPM          | 500 RPM             | PID-LubePump       |
| Feed Rate Control          | Output       | AO-003            | Analog           | m/min              | 0 to 100 m/min         | 50 m/min            | PID-FeedRate       |
| Exit Speed Control         | Output       | AO-004            | Analog           | RPM                | 0 to 1000 RPM          | 500 RPM             | PID-ExitSpeed      |
| Strip Thickness Gauge      | Input        | AI-007            | Analog           | mm                 | 0 to 20 mm             | N/A                 | PID-Thickness      |
| Strip Width Gauge          | Input        | AI-008            | Analog           | mm                 | 0 to 500 mm            | N/A                 | PID-Width          |
| Strip Surface Roughness    | Input        | AI-009            | Analog           | μm                 | 0 to 100 μm            | N/A                 | PID-SurfaceRough   |
| Power Supply Voltage       | Input        | AI-010            | Analog           | VAC                | 0 to 600 VAC           | N/A                 | PID-PowerSupply    |
| Emergency Stop Button      | Input        | DI-001            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-EmergencyStop  |
| Start Button               | Input        | DI-002            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-StartButton    |
| Stop Button                | Input        | DI-003            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-StopButton     |
| Overload Relay             | Input        | DI-004            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-OverloadRelay  |
| Fault Indicator            | Output       | DO-001            | Digital          | Bool               | OFF/ON                 | OFF                 | PID-FaultIndicator |
| Coolant Flow Sensor        | Input        | AI-011            | Analog           | L/min              | 0 to 1000 L/min        | N/A                 | PID-CoolantFlow    |
| Air Compressor Status      | Input        | DI-005            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-AirCompressor  |
| Lubrication System Status  | Input        | DI-006            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-LubeSystem     |
| Belt Tensioner Position    | Input        | AI-012            | Analog           | mm                 | 0 to 100 mm            | N/A                 | PID-BeltTension    |
| Hydraulic Fluid Level      | Input        | AI-013            | Analog           | mm                 | 0 to 1000 mm           | N/A                 | PID-HydraulicLevel |
| Main Drive Motor Status    | Input        | DI-007            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-MainMotor      |
| Backup Generator Status    | Input        | DI-008            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-BackupGen      |
| Fire Detection Alarm       | Input        | DI-009            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-FireAlarm      |
| Maintenance Mode Switch    | Input        | DI-010            | Digital          | Bool               | OFF/ON                 | N/A                 | PID-MaintenanceMode|

### Explanation of the Table

- **Signal Name**: Describes the purpose of each signal.
- **Input/Output**: Specifies whether the signal is an input or output.
- **Signal Tag Number**: Unique identifier for each signal.
- **Analog/Digital**: Indicates whether the signal is analog or digital.
- **Engineering Unit**: Units of measurement for the signal.
- **Range**: Operational range of the signal.
- **Setpoint**: Target value for control signals (where applicable).
- **P&ID Reference**: Reference to the Process and Instrumentation Diagram for physical location and connection details.

This I/O list serves as a foundational reference for control system development, instrumentation wiring, and P&ID alignment, ensuring accurate communication between automation, mechanical, and electrical engineering teams.
