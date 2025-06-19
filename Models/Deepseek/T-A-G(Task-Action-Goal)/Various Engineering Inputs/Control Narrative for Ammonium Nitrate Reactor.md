### Control Narrative for Ammonium Nitrate Production Reactor

#### 1. Process Context
The process involves the continuous neutralization of nitric acid with ammonia in a stirred reactor to produce ammonium nitrate. The reactor must be operated under strict temperature, pressure, and pH control to ensure safety and product quality.

#### 2. Critical Parameters and Setpoints
- **Temperature**: 
  - Setpoint: 175°C ± 2°C
  - Range: 173°C to 177°C
- **Pressure**:
  - Setpoint: 4.8 bar ± 0.2 bar
  - Range: 4.6 bar to 5.0 bar
- **Ammonia-to-Acid Molar Ratio**:
  - Setpoint: 1.01:1
- **pH**:
  - Setpoint: 6.2 ± 0.3
  - Range: 5.9 to 6.5

#### 3. Instrumentation and Control Elements
- **TIC-101**: Reactor Temperature Controller
- **PIC-102**: Pressure Controller
- **FIC-103**: Ammonia Flow Controller
- **FIC-104**: Acid Flow Controller
- **AIC-105**: pH Controller

#### 4. Operating Sequence

##### Startup
1. **Valve Alignment**:
   - Open inlet valves for both ammonia and acid.
   - Close outlet valve for the reactor effluent.
   - Ensure all bypass valves are closed unless required for initial flushing.

2. **Flow Ramp-Up**:
   - Gradually increase the flow rates of ammonia and acid to avoid sudden changes.
   - Start FIC-103 (ammonia flow) at a low rate and gradually increase to the desired setpoint.
   - Start FIC-104 (acid flow) similarly.

3. **Temperature Control**:
   - Initiate heating of the reactor using TIC-101.
   - Monitor and adjust the temperature to reach and maintain the setpoint of 175°C ± 2°C.

4. **pH Stabilization**:
   - Begin adding base (if necessary) to stabilize the pH.
   - Use AIC-105 to monitor and control the pH to the setpoint of 6.2 ± 0.3.

##### Normal Operation
1. **Maintain Loop Control**:
   - Continuously monitor and control temperature using TIC-101.
   - Maintain pressure within the set range using PIC-102.
   - Adjust ammonia and acid flows to maintain the molar ratio of 1.01:1 using FIC-103 and FIC-104.
   - Control pH using AIC-105.

2. **Alarm Monitoring**:
   - Implement alarms for any deviations outside the control ranges.
   - Regularly check and log all process variables.

##### Shutdown
1. **Stop Feeds**:
   - Gradually reduce and stop the flow of ammonia and acid.
   - Ensure that all feed valves are fully closed.

2. **Isolate Reactor**:
   - Isolate the reactor by closing all inlet and outlet valves.
   - Ensure no material can enter or leave the reactor during isolation.

3. **Relieve Pressure**:
   - Slowly relieve the pressure in the reactor to atmospheric levels.
   - Monitor the pressure using PIC-102.

#### 5. Interlocks and Alarm Logic

##### Interlocks
- **High Pressure Trip**:
  - IF Pressure > 5.2 bar THEN trip the system.
  - Automatically shut down ammonia and acid feeds.
  - Open relief valve to vent excess pressure.

- **Emergency Shutdown**:
  - IF Temperature > 185°C THEN emergency shutdown.
  - Immediately stop all feeds and isolate the reactor.
  - Activate emergency cooling systems if available.

- **Low pH Adjustment**:
  - IF pH < 5.5 THEN reduce ammonia feed.
  - Decrease FIC-103 (ammonia flow) to lower the pH closer to the setpoint.

##### Alarms
- **Temperature High Alarm**:
  - IF Temperature > 177°C THEN raise high temperature alarm.
  - Notify operators to investigate and take corrective actions.

- **Temperature Low Alarm**:
  - IF Temperature < 173°C THEN raise low temperature alarm.
  - Notify operators to investigate and take corrective actions.

- **Pressure High Alarm**:
  - IF Pressure > 5.0 bar THEN raise high pressure alarm.
  - Notify operators to investigate and take corrective actions.

- **Pressure Low Alarm**:
  - IF Pressure < 4.6 bar THEN raise low pressure alarm.
  - Notify operators to investigate and take corrective actions.

- **pH High Alarm**:
  - IF pH > 6.5 THEN raise high pH alarm.
  - Notify operators to investigate and take corrective actions.

- **pH Low Alarm**:
  - IF pH < 5.9 THEN raise low pH alarm.
  - Notify operators to investigate and take corrective actions.

#### 6. Summary
This control narrative provides a comprehensive framework for automating the ammonium nitrate production reactor. It includes detailed setpoints, ranges for critical parameters, instrumentation and control elements, operating sequences, and interlock and alarm logic. By following this narrative, engineers and operators can ensure safe and consistent reactor operation, facilitating reliable PLC/DCS implementation and maintaining high-quality product output.

By adhering to these guidelines, the reactor will operate efficiently and safely, minimizing the risk of process upsets and ensuring consistent quality of ammonium nitrate production.
