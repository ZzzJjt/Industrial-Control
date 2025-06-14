### Feedwater Control System I/O List

| Signal Name                        | Signal Tag Number | Analog/Digital | Engineering Unit | Range                    | Setpoint              | P&ID Reference     |
|------------------------------------|-------------------|------------------|--------------------|--------------------------|---------------------|------------------|
| Drum Level Transmitter             | AI-001            | Analog           | %                  | 0% to 100%               | 50%                 | PID-DrumLevel      |
| Feedwater Control Valve            | AO-001            | Analog           | % Opening          | 0% to 100%               | 50%                 | PID-FWControlValve |
| Feedwater Flow Transmitter         | AI-002            | Analog           | kg/s               | 0 to 1000 kg/s           | 500 kg/s              | PID-FWFlow         |
| Boiler Feed Pump Speed             | AI-003            | Analog           | RPM                | 0 to 3000 RPM            | 1500 RPM              | PID-BFPumpSpeed    |
| Boiler Feed Pump Motor Current     | AI-004            | Analog           | Amps               | 0 to 1000 Amps           | N/A                 | PID-BFPumpCurrent  |
| Boiler Feed Pump Status            | DI-001            | Digital          | Bool               | OFF/ON                   | N/A                 | PID-BFPumpStatus   |
| Deaerator Level Transmitter        | AI-005            | Analog           | %                  | 0% to 100%               | 40%                 | PID-DeaeratorLevel |
| Deaerator Temperature Transmitter  | AI-006            | Analog           | °C                 | 0 to 200 °C              | 100 °C              | PID-DeaeratorTemp  |
| Condenser Pressure Transmitter     | AI-007            | Analog           | BarA               | 0 to 1 BarA              | N/A                 | PID-CondenserPress |
| Feedwater Preheater Outlet Temp    | AI-008            | Analog           | °C                 | 0 to 250 °C              | N/A                 | PID-FWPrecHeatTemp |
| Steam Header Pressure Transmitter  | AI-009            | Analog           | BarA               | 0 to 20 BarA             | N/A                 | PID-SteamHeaderPress |
| Feedwater Heater Outlet Temp       | AI-010            | Analog           | °C                 | 0 to 300 °C              | N/A                 | PID-FWHeaterOutTemp  |
| Feedwater Heater Inlet Temp        | AI-011            | Analog           | °C                 | 0 to 300 °C              | N/A                 | PID-FWHeaterInTemp   |
| Feedwater Heater Outlet Press      | AI-012            | Analog           | BarA               | 0 to 20 BarA             | N/A                 | PID-FWHeaterOutPress |
| Feedwater Heater Inlet Press       | AI-013            | Analog           | BarA               | 0 to 20 BarA             | N/A                 | PID-FWHeaterInPress  |
| Feedwater Heater Bypass Valve      | AO-002            | Analog           | % Opening          | 0% to 100%               | 0%                  | PID-FWHBypassValve |
| Feedwater Heater Isolation Valve   | DO-001            | Digital          | Bool               | OFF/ON                   | OFF                 | PID-FWIsoValve     |
| Feedwater Pump Vibration Sensor    | AI-014            | Analog           | mm/s               | 0 to 10 mm/s             | N/A                 | PID-PumpVibration  |
| Feedwater Pump Bearing Temp        | AI-015            | Analog           | °C                 | 0 to 150 °C              | N/A                 | PID-PumpBearTemp   |
| Feedwater Pump Seal Water Flow     | AI-016            | Analog           | L/min              | 0 to 100 L/min           | N/A                 | PID-SealWaterFlow  |
| Emergency Stop Button              | DI-002            | Digital          | Bool               | OFF/ON                   | N/A                 | PID-EmergencyStop  |
| Start Button                       | DI-003            | Digital          | Bool               | OFF/ON                   | N/A                 | PID-StartButton    |
| Stop Button                        | DI-004            | Digital          | Bool               | OFF/ON                   | N/A                 | PID-StopButton     |
| Overload Relay                     | DI-005            | Digital          | Bool               | OFF/ON                   | N/A                 | PID-OverloadRelay  |
| Fault Indicator                    | DO-002            | Digital          | Bool               | OFF/ON                   | OFF                 | PID-FaultIndicator |

### Explanation of the Table

- **Signal Name**: Describes the purpose of each signal.
- **Signal Tag Number**: Unique identifier for each signal.
- **Analog/Digital**: Indicates whether the signal is analog or digital.
- **Engineering Unit**: Units of measurement for the signal.
- **Range**: Operational range of the signal.
- **Setpoint**: Target value for control signals (where applicable).
- **P&ID Reference**: Reference to the Process and Instrumentation Diagram for physical location and connection details.

This I/O list provides a complete and traceable reference that supports automation design, instrumentation layout, and integration with P&ID drawings, ensuring consistency and accuracy in the implementation of the feedwater control loop.
