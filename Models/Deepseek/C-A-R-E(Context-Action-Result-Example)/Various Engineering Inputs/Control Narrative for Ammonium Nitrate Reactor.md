Certainly! Below is a detailed control narrative for an ammonium nitrate reactor, focusing on control setpoints, instrumentation, operational sequences, and control loops with interlock logic.

### Ammonium Nitrate Reactor Control Narrative

#### Overview
This control narrative outlines the standardized procedures and automation logic for operating an ammonium nitrate reactor. The document serves as a shared reference for engineers, operators, and programmers to ensure safe, consistent, and efficient operation of the reactor, supporting PLC/DCS programming, commissioning, training, and troubleshooting.

---

### Control Parameters and Setpoints

**Reactor Temperature (TIC-101):**
- Setpoint: 175°C ± 2°C
- Alarm Limits:
  - High Alarm: 180°C
  - High-High Alarm (Interlock): 185°C

**Reactor Pressure (PIC-102):**
- Setpoint: 4.8 bar ± 0.2 bar
- Alarm Limits:
  - High Alarm: 5.0 bar
  - High-High Alarm (Interlock): 5.2 bar

**Ammonia to Nitric Acid Flow Ratio (FRC-103):**
- Setpoint: 1.01:1
- Alarm Limits:
  - Low Alarm: 0.99:1
  - High Alarm: 1.03:1

**pH Range (AIC-104):**
- Setpoint: 6.2 ± 0.3
- Alarm Limits:
  - Low Alarm: 5.9
  - Low-Low Alarm (Interlock): 5.5

---

### Instrumentation and Equipment

**Flow Transmitters:**
- FT-101: Ammonia flow transmitter
- FT-102: Nitric acid flow transmitter

**Pressure Transmitter:**
- PT-101: Reactor pressure transmitter

**Temperature Transmitter:**
- TT-101: Reactor temperature transmitter

**pH Sensor:**
- pH-101: pH sensor in reactor outlet

**Valves:**
- VV-101: Ammonia feed valve
- VV-102: Nitric acid feed valve
- SV-101: Steam valve (for heating)
- SV-102: Emergency shutoff valve (reactor inlet)
- SV-103: Emergency shutoff valve (reactor outlet)

**Interlocks:**
- Interlock for high temperature (>185°C)
- Interlock for overpressure (>5.2 bar)
- Interlock for low pH (<5.5)

---

### Operational Sequence Overview

#### Startup Procedure

1. **Initialize System:**
   - Confirm all valves are closed.
   - Verify tank levels are within acceptable ranges.
   - Power up all instrumentation and control systems.

2. **Begin Controlled Feed:**
   - Open ammonia feed valve (VV-101) slowly to establish initial flow.
   - Open nitric acid feed valve (VV-102) slowly to maintain the specified flow ratio (1.01:1).

3. **Ramp Up Reactor Temperature:**
   - Open steam valve (SV-101) gradually to heat the reactor.
   - Monitor reactor temperature using TIC-101 and adjust steam flow as needed.

4. **Adjust Flow Ratios:**
   - Continuously monitor ammonia and nitric acid flows using FT-101 and FT-102.
   - Adjust feed valves (VV-101 and VV-102) to maintain the correct flow ratio and pH.

5. **Monitor Steady-State Operation:**
   - Once steady-state conditions are achieved, monitor all parameters continuously.
   - Activate alarms and interlocks to respond to deviations from setpoints.

#### Steady-State Operation

1. **Maintain Setpoints:**
   - Use PID controllers to maintain reactor temperature, pressure, and flow ratios.
   - Adjust pH by monitoring AIC-104 and making necessary adjustments to feed rates or adding neutralizing agents if required.

2. **Continuous Monitoring:**
   - Continuously monitor reactor temperature (TT-101).
   - Continuously monitor reactor pressure (PT-101).
   - Continuously monitor ammonia and nitric acid flows (FT-101 and FT-102).
   - Continuously monitor pH (AIC-104).

3. **Alarm and Interlock Response:**
   - Respond to high temperature, overpressure, and low pH alarms promptly.
   - Trigger emergency shutdown if high-high temperature, overpressure, or low-low pH interlocks are activated.

#### Shutdown Procedure

1. **Close Feeds:**
   - Gradually close ammonia feed valve (VV-101).
   - Gradually close nitric acid feed valve (VV-102).

2. **Vent Pressure Safely:**
   - Close steam valve (SV-101).
   - Slowly vent pressure from the reactor while monitoring PT-101.

3. **Cool Down:**
   - Allow the reactor to cool naturally or use cooling water if necessary.
   - Monitor reactor temperature (TT-101) until it reaches ambient temperature.

4. **Final Checks:**
   - Confirm all valves are closed.
   - Verify no residual pressure in the reactor (PT-101).
   - Ensure all instruments are powered down safely.

---

### Control Loops and Interlock Logic

#### Control Loops

1. **Temperature Control Loop (PID Controller):**
   - Input: Reactor temperature (TT-101)
   - Output: Steam valve (SV-101)
   - Setpoint: 175°C ± 2°C
   - Action: Adjust steam flow to maintain temperature within setpoint.

2. **Pressure Control Loop (PID Controller):**
   - Input: Reactor pressure (PT-101)
   - Output: Not applicable (manual intervention required for venting)
   - Setpoint: 4.8 bar ± 0.2 bar
   - Action: Monitor pressure and trigger manual venting if necessary.

3. **Flow Control Loop (PID Controllers):**
   - Input: Ammonia flow (FT-101) and Nitric acid flow (FT-102)
   - Outputs: Ammonia feed valve (VV-101) and Nitric acid feed valve (VV-102)
   - Setpoint: Ammonia to Nitric Acid flow ratio of 1.01:1
   - Action: Adjust feed valves to maintain the correct flow ratio.

4. **pH Control Loop (PID Controller):**
   - Input: pH (AIC-104)
   - Outputs: Ammonia feed valve (VV-101) and Nitric acid feed valve (VV-102)
   - Setpoint: 6.2 ± 0.3
   - Action: Adjust feed valves to maintain pH within setpoint.

#### Interlock Logic

1. **High Temperature Interlock:**
   - Condition: Reactor temperature > 185°C
   - Action: Close all feed valves (VV-101, VV-102) and steam valve (SV-101). Trigger emergency shutoff valves (SV-102, SV-103).

2. **Overpressure Interlock:**
   - Condition: Reactor pressure > 5.2 bar
   - Action: Close all feed valves (VV-101, VV-102) and steam valve (SV-101). Trigger emergency shutoff valves (SV-102, SV-103).

3. **Low pH Interlock:**
   - Condition: pH < 5.5
   - Action: Close all feed valves (VV-101, VV-102) and steam valve (SV-101). Trigger emergency shutoff valves (SV-102, SV-103).

---

### Conclusion

This control narrative provides a comprehensive guide for automating the ammonium nitrate reactor process, ensuring safe and consistent operation. Each section includes specific setpoints, instrumentation, and automated control actions, making it easier for engineering, operations, and programming teams to implement and maintain the control system.

By following this narrative, manufacturers can achieve precise and repeatable results, leading to higher yields, improved safety, and regulatory compliance.
