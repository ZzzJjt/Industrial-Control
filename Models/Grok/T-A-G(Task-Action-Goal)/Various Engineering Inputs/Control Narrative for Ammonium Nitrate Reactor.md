I'll develop a detailed control narrative for a continuous ammonium nitrate production reactor, focusing on the neutralization of nitric acid with ammonia in a stirred reactor. The narrative will include concrete setpoints, equipment specifications, instrumentation, operating logic, interlocks, and alarm logic, ensuring automation-readiness for PLC/DCS implementation. The goal is to provide a clear, structured document that ensures safe operation, consistent product quality, and reliable control.

### Control Narrative for Ammonium Nitrate Reactor

#### Process Context
The ammonium nitrate production process involves the continuous neutralization of nitric acid (HNO₃) with ammonia (NH₃) in a stirred, jacketed reactor to produce ammonium nitrate (NH₄NO₃) solution. The reaction is exothermic and occurs at elevated temperature and pressure to maintain the solution in a liquid state:

\[ \text{NH}_3 + \text{HNO}_3 \rightarrow \text{NH}_4\text{NO}_3 \]

The reactor operates continuously, with feeds of aqueous nitric acid (60% w/w) and gaseous ammonia, producing a 90% w/w ammonium nitrate solution that is pumped to a downstream evaporator. The process is controlled to maintain precise temperature, pressure, pH, and molar ratio to ensure safety, product quality, and operational stability. The reactor has a working volume of 5000 L and is designed for a production rate of 2000 kg/h of ammonium nitrate solution.

#### Process Stages
The process is divided into three main stages: Startup, Normal Operation, and Shutdown, each with specific setpoints, control actions, and safety measures.

1. **Startup**
   - **Objective**: Safely initialize the reactor, align valves, ramp up feeds, and stabilize temperature and pH.
   - **Duration**: Approximately 60 minutes.

2. **Normal Operation**
   - **Objective**: Maintain steady-state production with precise control of temperature, pressure, molar ratio, and pH, while monitoring for alarms.

3. **Shutdown**
   - **Objective**: Safely stop feeds, isolate the reactor, and relieve pressure for maintenance or emergency conditions.

#### Critical Setpoints and Control Ranges
- **Temperature**: 175°C ± 2°C (ensures liquid phase, prevents decomposition).
- **Pressure**: 4.8 bar ± 0.2 bar (maintains solution stability, prevents vaporization).
- **Ammonia-to-Acid Molar Ratio**: 1.01:1 ± 0.02 (slight ammonia excess for complete neutralization).
- **pH**: 6.2 ± 0.3 (indicates proper neutralization, slightly basic due to ammonia excess).
- **Nitric Acid Feed Flow**: 800 L/h ± 50 L/h (60% w/w, ~8.6 mol/L, delivers ~6880 mol/h HNO₃).
- **Ammonia Feed Flow**: 118 kg/h ± 10 kg/h (gaseous, ~6940 mol/h NH₃, for 1.01:1 molar ratio).
- **Reactor Level**: 70% ± 5% (3500 L ± 250 L, ensures sufficient mixing and residence time).
- **Agitator Speed**: 100 RPM ± 10 RPM (ensures uniform mixing).
- **Product Flow**: 2222 L/h ± 100 L/h (90% w/w NH₄NO₃ solution, ~2000 kg/h).

#### Equipment
- **Reactor (R-101)**: 5000 L stainless steel, jacketed, stirred reactor with cooling water and steam systems.
- **Agitator**: Variable-speed motor, 0–150 RPM.
- **Nitric Acid Feed System**: Centrifugal pump (P-101), control valve (FV-103).
- **Ammonia Feed System**: Gas compressor (C-101), control valve (FV-104).
- **Product Pump (P-102)**: Transfers ammonium nitrate solution to evaporator.
- **Cooling Water System**: Supplies cooling water to jacket for temperature control.
- **Steam System**: Supplies steam to jacket for heating during startup.
- **Vent System**: Pressure relief valve (PV-102) to flare or scrubber.

#### Instrumentation and Control Elements
- **TIC-101 (Temperature Controller)**: Reactor temperature probe, controls cooling water and steam valves.
- **PIC-102 (Pressure Controller)**: Reactor pressure transmitter, controls vent valve (PV-102).
- **FIC-103 (Nitric Acid Flow Controller)**: Flow transmitter on nitric acid feed line, controls valve FV-103.
- **FIC-104 (Ammonia Flow Controller)**: Flow transmitter on ammonia feed line, controls valve FV-104.
- **AIC-105 (pH Controller)**: pH probe in reactor, adjusts ammonia flow via FIC-104 setpoint.
- **LIC-106 (Level Controller)**: Level transmitter, controls product pump (P-102) speed.
- **SIC-107 (Agitator Speed Controller)**: Speed sensor, controls agitator motor.
- **Safety Instrumentation**:
  - **TI-108**: Independent high-temperature switch (activates at 185°C).
  - **PI-109**: Independent high-pressure switch (activates at 5.2 bar).

#### Operating Sequence

**1. Startup**
   - **Objective**: Initialize reactor to operating conditions safely.
   - **Duration**: ~60 minutes.
   - **Procedure**:
     1. **Valve Alignment** (0–5 min):
        - Open cooling water valve (CV-101) to 50% for initial cooling.
        - Close vent valve (PV-102), product valve (FV-106), and feed valves (FV-103, FV-104).
        - Confirm agitator stopped (SIC-107 = 0 RPM).
     2. **Pre-Fill Reactor** (5–15 min):
        - Open nitric acid valve (FV-103) to fill reactor to 50% level (2500 L ± 100 L) at 800 L/h.
        - Start agitator (SIC-107) at 100 RPM when level > 20%.
        - Monitor LIC-106; stop FV-103 when level reaches 50%.
     3. **Temperature Ramp-Up** (15–30 min):
        - Open steam valve (SV-101) to heat reactor to 175°C at 5°C/min via TIC-101.
        - Transition to cooling water (CV-101) when temperature > 170°C to maintain 175°C ± 2°C.
     4. **Feed Ramp-Up** (30–45 min):
        - Start nitric acid feed (FIC-103) at 800 L/h ± 50 L/h.
        - Start ammonia feed (FIC-104) at 118 kg/h ± 10 kg/h, maintaining 1.01:1 molar ratio.
        - Adjust ammonia flow via AIC-105 to achieve pH 6.2 ± 0.3.
        - Monitor LIC-106; start product pump (P-102) at 2222 L/h when level > 70%.
     5. **Stabilization** (45–60 min):
        - Confirm temperature (TIC-101 = 175°C ± 2°C), pressure (PIC-102 = 4.8 bar ± 0.2 bar), pH (AIC-105 = 6.2 ± 0.3), and level (LIC-106 = 70% ± 5%).
        - Transition to Normal Operation if all parameters are stable for 5 min.
   - **Control Actions**:
     - TIC-101: PID control of steam (SV-101) and cooling water (CV-101) valves.
     - FIC-103/104: PID control of feed valves FV-103/FV-104.
     - AIC-105: Cascade control, adjusting FIC-104 setpoint for pH.
     - LIC-106: PID control of P-102 speed.
   - **Monitoring**:
     - Verify molar ratio (calculated from FIC-103/104 flows) = 1.01 ± 0.02.
     - Check agitator speed (SIC-107) = 100 RPM ± 10 RPM.

**2. Normal Operation**
   - **Objective**: Maintain steady-state production with tight control of process parameters.
   - **Duration**: Continuous until shutdown.
   - **Procedure**:
     1. **Temperature Control**:
        - TIC-101 maintains 175°C ± 2°C via cooling water valve (CV-101).
        - Adjust cooling water flow (10–50 L/min) to counter exothermic reaction heat.
     2. **Pressure Control**:
        - PIC-102 maintains 4.8 bar ± 0.2 bar via vent valve (PV-102).
        - Open PV-102 to flare/scrubber if pressure > 4.9 bar.
     3. **Feed Control**:
        - FIC-103 maintains nitric acid flow at 800 L/h ± 50 L/h.
        - FIC-104 maintains ammonia flow at 118 kg/h ± 10 kg/h.
        - AIC-105 adjusts FIC-104 setpoint to maintain pH 6.2 ± 0.3.
     4. **Level Control**:
        - LIC-106 maintains reactor level at 70% ± 5% by adjusting product pump (P-102) speed for 2222 L/h output.
     5. **Agitator Control**:
        - SIC-107 maintains agitator speed at 100 RPM ± 10 RPM.
     6. **Alarm Monitoring**:
        - Monitor TIC-101, PIC-102, AIC-105, LIC-106 for deviations.
        - Log molar ratio every 5 min to ensure 1.01:1 ± 0.02.
   - **Control Actions**:
     - TIC-101: PID control with anti-windup for CV-101.
     - PIC-102: Proportional control for PV-102 to minimize venting.
     - FIC-103/104: PID control with feedforward from molar ratio calculation.
     - AIC-105: Cascade PID, tuning ammonia flow.
     - LIC-106: PID control of P-102 speed, ensuring stable level.
   - **Monitoring**:
     - Sample pH every 1 min, average over 3 samples to reduce noise.
     - Verify product flow via flow transmitter on P-102 output.
     - Log temperature, pressure, pH, and level every 5 min for quality records.

**3. Shutdown**
   - **Objective**: Safely stop feeds, isolate reactor, and relieve pressure.
   - **Duration**: ~30 minutes.
   - **Procedure**:
     1. **Stop Feeds** (0–5 min):
        - Close nitric acid valve (FV-103) and ammonia valve (FV-104).
        - Stop product pump (P-102).
     2. **Cool Reactor** (5–20 min):
        - Open cooling water valve (CV-101) to 100% to reduce temperature to < 100°C at 5°C/min.
        - Maintain agitator at 100 RPM to ensure mixing during cooling.
     3. **Relieve Pressure** (20–25 min):
        - Open vent valve (PV-102) to reduce pressure to 1.0 bar ± 0.1 bar.
        - Route vented gases to scrubber.
     4. **Isolate Reactor** (25–30 min):
        - Stop agitator (SIC-107 = 0 RPM).
        - Close all valves (CV-101, PV-102, FV-106).
        - Confirm reactor level < 10% via LIC-106.
   - **Control Actions**:
     - TIC-101: Control CV-101 for cooling.
     - PIC-102: Control PV-102 for pressure relief.
     - LIC-106: Monitor level to confirm draining.
   - **Monitoring**:
     - Verify temperature < 100°C and pressure < 1.1 bar before isolation.
     - Confirm zero flow on FIC-103/104 and P-102.

#### Interlocks and Alarm Logic

**Interlocks**:
- **High Pressure Trip**:
  - IF PIC-102 > 5.2 bar OR PI-109 activates THEN:
    - Close FV-103, FV-104 (stop feeds).
    - Open PV-102 to maximum to relieve pressure.
    - Stop P-102 and agitator (SIC-107).
    - Signal alarm: "High Pressure Trip".
- **High Temperature Emergency Shutdown**:
  - IF TIC-101 > 185°C OR TI-108 activates THEN:
    - Initiate full emergency shutdown:
      - Close FV-103, FV-104, FV-106.
      - Stop P-102, agitator, and compressor (C-101).
      - Open CV-101 to 100% and PV-102 to maximum.
    - Signal alarm: "Emergency Shutdown – High Temperature".
- **Low pH Adjustment**:
  - IF AIC-105 < 5.5 for > 1 min THEN:
    - Reduce FIC-104 setpoint by 10% (ammonia flow to ~106 kg/h).
    - Signal alarm: "Low pH – Ammonia Feed Reduced".
    - IF pH < 5.0 for > 2 min THEN close FV-104 and signal "Critical Low pH".
- **Level Interlock**:
  - IF LIC-106 < 50% THEN:
    - Reduce FIC-103/104 flows to 50% (400 L/h nitric acid, 59 kg/h ammonia).
    - Signal alarm: "Low Reactor Level".
  - IF LIC-106 > 80% THEN:
    - Increase P-102 speed to 2500 L/h.
    - Signal alarm: "High Reactor Level".
- **Flow Interlock**:
  - IF FIC-103 < 600 L/h OR FIC-104 < 90 kg/h for > 1 min THEN:
    - Stop P-101, C-101, and P-102.
    - Signal alarm: "Low Feed Flow – Pump/Compressor Fault".
- **Safety Interlock**:
  - IF Emergency Stop is pressed THEN:
    - Stop all equipment (P-101, C-101, P-102, agitator).
    - Close all valves (FV-103, FV-104, FV-106, PV-102, CV-101, SV-101).
    - Signal alarm: "Emergency Stop Activated".

**Alarms**:
- **Temperature Deviation**: TIC-101 > 177°C or < 173°C for > 2 min → "Temperature Out of Range".
- **Pressure Deviation**: PIC-102 > 5.0 bar or < 4.6 bar for > 1 min → "Pressure Out of Range".
- **pH Deviation**: AIC-105 > 6.5 or < 5.9 for > 1 min → "pH Out of Range".
- **Level Deviation**: LIC-106 > 80% or < 50% for > 1 min → "Level Out of Range".
- **Molar Ratio Deviation**: Ratio < 0.99 or > 1.03 for > 2 min → "Molar Ratio Out of Range".
- **Sensor Fault**: Any sensor (TIC-101, PIC-102, AIC-105, LIC-106, FIC-103/104) reports invalid data → "Sensor Fault".

#### Additional Notes

- **Safety**:
  - High temperature (>185°C) and pressure (>5.2 bar) interlocks prevent decomposition or explosion risks, critical for ammonium nitrate’s sensitivity.
  - pH control (6.2 ± 0.3) ensures complete neutralization, avoiding corrosive acidic conditions or excess ammonia.
  - Emergency shutdown logic isolates the reactor rapidly, venting to a scrubber to handle ammonia emissions.

- **Consistency and Quality**:
  - Precise molar ratio (1.01:1) ensures stoichiometric reaction, maximizing ammonium nitrate yield (~99% conversion).
  - Temperature (175°C) and pressure (4.8 bar) maintain the solution in liquid form, preventing crystallization in the reactor.
  - pH stabilization (6.2) confirms proper neutralization, critical for downstream processing.

- **Operational Reliability**:
  - Automated setpoints and PID control (e.g., TIC-101, AIC-105) reduce operator intervention, ensuring repeatability.
  - Interlocks and alarms protect equipment (e.g., pumps, agitator) and process integrity (e.g., pH < 5.5).
  - Cascade control (AIC-105 adjusting FIC-104) stabilizes pH despite feed variations.

- **Automation Readiness**:
  - Concrete setpoints (e.g., 175°C, 4.8 bar, 800 L/h) and interlocks are directly implementable in PLC/DCS systems using IEC 61131-3 (e.g., Structured Text).
  - Instrumentation (TIC-101, PIC-102, etc.) supports real-time feedback for precise control.
  - Operating sequence aligns with sequential function chart (SFC) or state machine implementation, with clear transitions (e.g., stabilization to normal operation).

This control narrative provides a clear, structured, and automation-friendly framework for the ammonium nitrate reactor, ensuring safe operation, consistent quality, and reliable PLC/DCS implementation. Let me know if you need a Structured Text function block for any control loop (e.g., TIC-101, AIC-105) or further details on instrumentation or safety systems!
