# Textual Piping and Instrumentation Diagram (P&ID) for Distillation Column System

## System Overview
- **Unit**: Distillation Column (C-101)
- **Purpose**: Separates liquid mixture into components based on boiling points.
- **Key Equipment**:
  - C-101: Distillation Column
  - E-101: Reboiler (provides heat for vaporization)
  - E-102: Condenser (condenses overhead vapor)
- **Process Flow**:
  - Feed enters C-101 via FV-101 (Feed Valve).
  - E-101 heats liquid at column base, generating vapor.
  - Vapor rises, condenses in E-102, and exits as distillate.
  - Bottom product exits C-101 base.

## Instrumentation
- **Transmitters**:
  - PT-101: Pressure Transmitter (measures column pressure, range: 0–150 psi)
  - TT-101: Temperature Transmitter (measures reboiler temperature, range: 0–200°C)
  - LT-101: Level Transmitter (measures bottom liquid level, range: 0–100%)
- **Valves**:
  - FV-101: Feed Valve (controls feed flow, pneumatic, fail-closed)
  - PRV-101: Pressure Relief Valve (relieves excess pressure, spring-loaded, set at 125 psi)
- **Actuators**:
  - E-101-H: Reboiler Heater Control (on/off control for heating system)

## Control Loops
- **Temperature Control Loop**:
  - **Sensor**: TT-101 (measures reboiler temperature)
  - **Controller**: TIC-101 (PID controller, setpoint: 150°C)
  - **Actuator**: E-101-H (modulates reboiler heating)
  - **Function**: Maintains reboiler temperature at setpoint by adjusting heater power.
- **Level Control Loop**:
  - **Sensor**: LT-101 (measures bottom liquid level)
  - **Controller**: LIC-101 (PID controller, setpoint: 50%)
  - **Actuator**: LV-101 (bottom product valve, not detailed in interlocks)
  - **Function**: Maintains liquid level by adjusting bottom product outflow.
- **Pressure Control Loop**:
  - **Sensor**: PT-101 (measures column pressure)
  - **Controller**: PIC-101 (PID controller, setpoint: 100 psi)
  - **Actuator**: CV-101 (vapor outlet valve, not detailed in interlocks)
  - **Function**: Maintains column pressure by adjusting vapor outflow.

## Interlock Actions
- **High Pressure (PT-101 > 120 psi)**:
  - Open PRV-101 to relieve pressure.
  - Close FV-101 to stop feed input.
  - Stop E-101-H to halt heating.
  - Issue alarm (ALM-101).
- **Low Pressure (PT-101 < 50 psi)**:
  - Close FV-101 to stop feed input.
  - Issue alarm (ALM-102).
- **High Temperature (TT-101 > 180°C)**:
  - Stop E-101-H to halt heating.
  - Issue alarm (ALM-103).
- **Low Liquid Level (LT-101 < 10%)**:
  - Close FV-101 to stop feed input.
  - Stop E-101-H to halt heating.
  - Issue alarm (ALM-104).
- **High Liquid Level (LT-101 > 90%)**:
  - Close FV-101 to stop feed input.
  - Issue alarm (ALM-105).

## Notes
- **Standards**: Instrumentation tagnames follow ISA-5.1 (e.g., PT-101 for pressure transmitter).
- **Safety Limits**:
  - Pressure: Safe range 50–120 psi; relief valve set at 125 psi.
  - Temperature: Safe range 40–180°C.
  - Level: Safe range 10–90%.
- **Interlock Implementation**: Executed via PLC using IEC 61131-3 Structured Text.
- **Assumptions**:
  - Single distillation column with standard instrumentation.
  - PLC communicates with devices via Profibus DP or similar.
  - Alarms are logged and displayed on HMI.
- **Usage**:
  - High Pressure: Opens PRV-101, closes FV-101, stops E-101-H, issues ALM-101.
  - High Temperature: Stops E-101-H, issues ALM-103.
