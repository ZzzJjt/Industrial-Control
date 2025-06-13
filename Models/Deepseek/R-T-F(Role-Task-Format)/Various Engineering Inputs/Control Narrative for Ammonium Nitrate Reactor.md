Certainly! Below is a detailed control narrative for the ammonium nitrate reactor, structured as requested. This document will serve as a formal guide for automation system configuration, safety documentation, and operator reference.

### 1. Process Overview

#### Purpose:
The primary purpose of the ammonium nitrate reactor is to neutralize nitric acid with ammonia to produce ammonium nitrate, a key component in fertilizers and explosives.

#### Reactor Type:
Continuous Stirred Tank Reactor (CSTR)

#### Normal Flow Conditions and Operational Limits:
- **Temperature**: 175°C ± 2°C
- **Pressure**: 4.8 bar ± 0.2 bar
- **Ammonia:Acid Ratio**: 1.01:1
- **pH Control**: Target = 6.2, acceptable range = 5.8–6.5

### 2. Key Setpoints and Ranges

- **Temperature (TIC-101)**:
  - Setpoint: 175°C ± 2°C

- **Pressure (PIC-102)**:
  - Setpoint: 4.8 bar ± 0.2 bar

- **Ammonia:Acid Ratio (FRC-103)**:
  - Setpoint: 1.01:1

- **pH Control (AIC-105)**:
  - Target: 6.2
  - Acceptable Range: 5.8–6.5

### 3. Instrumentation and Equipment

#### Control Loops:
- **Temperature Control Loop (TIC-101)**:
  - Controller: PID controller
  - Actuator: Heater/cooling jacket
  - Sensor: RTD temperature sensor

- **Pressure Control Loop (PIC-102)**:
  - Controller: PID controller
  - Actuator: Pressure relief valve
  - Sensor: Pressure transmitter

- **Flow Control Loop for Ammonia (FIC-103)**:
  - Controller: PID controller
  - Actuator: Control valve
  - Sensor: Flow transmitter

- **Flow Control Loop for Nitric Acid (FIC-104)**:
  - Controller: PID controller
  - Actuator: Control valve
  - Sensor: Flow transmitter

- **pH Control Loop (AIC-105)**:
  - Controller: PID controller
  - Actuator: pH adjustment doser
  - Sensor: pH analyzer

#### Safety Interlocks:
- **High-Pressure Switch**:
  - Trip Condition: Pressure > 5.2 bar
  - Action: Close feed valves

- **High-Temperature Alarm**:
  - Trip Condition: Temperature > 185°C
  - Action: Emergency shutdown

- **Emergency Shutoff Valve (ESD-106)**:
  - Trip Condition: High-temperature alarm or other critical faults
  - Action: Immediate isolation of feeds and depressurization

### 4. Sequence of Operation

#### Startup:
1. **Initialize Controllers**:
   - Set all controllers to manual mode.
   - Calibrate sensors and actuators.

2. **Ramp Up Feed Flows**:
   - Gradually open the ammonia and nitric acid feed valves.
   - Monitor flow rates using FIC-103 and FIC-104.

3. **Preheat Reactor**:
   - Activate the heater/cooling jacket to reach the setpoint temperature (175°C ± 2°C).
   - Monitor temperature using TIC-101.

4. **Activate Ratio and pH Control**:
   - Switch flow controllers to automatic mode (FRC-103).
   - Switch pH controller to automatic mode (AIC-105).

5. **Transition to Automatic Mode**:
   - Set all controllers to automatic mode.
   - Monitor process variables continuously.

#### Steady-State:
1. **Maintain PID Loops**:
   - Ensure all PID loops are functioning correctly.
   - Adjust setpoints if necessary based on process requirements.

2. **Monitor Deviation Alarms**:
   - Monitor deviation alarms for temperature, pressure, flow rates, and pH.
   - Take corrective actions if any alarms are triggered.

3. **Log Key Values**:
   - Log temperature, pressure, flow rates, and pH values regularly.
   - Use data for quality control and troubleshooting.

#### Shutdown:
1. **Isolate Feeds**:
   - Close all feed valves for ammonia and nitric acid.
   - Confirm no flow using FIC-103 and FIC-104.

2. **Depressurize**:
   - Open pressure relief valve to reduce pressure.
   - Monitor pressure using PIC-102.

3. **Stop Heating**:
   - Deactivate the heater/cooling jacket.
   - Allow the reactor to cool naturally.

4. **Drain Reactor Contents**:
   - Drain the reactor contents through the bottom outlet.
   - Clean the reactor thoroughly after draining.

### 5. Interlocks and Alarms

#### Interlocks:
- **High-Pressure Switch**:
  - Trip Condition: Pressure > 5.2 bar
  - Action: Close feed valves (FIC-103, FIC-104)

- **High-Temperature Alarm**:
  - Trip Condition: Temperature > 185°C
  - Action: Emergency shutdown via ESD-106

- **Emergency Shutoff Valve (ESD-106)**:
  - Trip Condition: High-temperature alarm or other critical faults
  - Action: Immediate isolation of feeds and depressurization

#### Alarms:
- **High-Temperature Alarm**:
  - Trip Condition: Temperature > 185°C
  - Action: Trigger emergency shutdown via ESD-106

- **Low-pH Alarm**:
  - Trip Condition: pH < 5.5
  - Action: Alarm + Reduce ammonia flow (FIC-103)

- **High-PH Alarm**:
  - Trip Condition: pH > 6.5
  - Action: Alarm + Increase ammonia flow (FIC-103)

This control narrative provides a comprehensive guide for configuring the automation system, ensuring safe and stable operation of the ammonium nitrate reactor. It includes detailed descriptions of the process, key parameters, instrumentation, sequences of operation, and interlock conditions.
