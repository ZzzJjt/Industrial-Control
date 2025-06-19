As a process control engineer, I’ll develop a formal control narrative for an ammonium nitrate reactor, designed for automation system configuration, safety documentation, and operator reference. The narrative will describe the process, specify key parameters, detail instrumentation and control equipment, outline operational sequences, and include interlocks and alarms to ensure safe and stable operation. The structure will follow the specified format, with precise setpoints, ranges, and control logic tailored for a continuous stirred tank reactor (CSTR).

### Control Narrative for Ammonium Nitrate Reactor

#### 1. Process Overview
- **Purpose**: The reactor neutralizes nitric acid (HNO₃) with ammonia (NH₃) to produce ammonium nitrate (NH₄NO₃), a key component in fertilizers and explosives. The exothermic reaction is:
  \[
  \text{NH}_3 + \text{HNO}_3 \rightarrow \text{NH}_4\text{NO}_3 + \text{Heat}
  \]
- **Reactor Type**: Continuous Stirred Tank Reactor (CSTR), ensuring uniform mixing and reaction conditions.
- **Process Description**: Ammonia gas and aqueous nitric acid are continuously fed into the reactor, where they react under controlled temperature, pressure, and pH conditions. The ammonium nitrate solution is withdrawn, cooled, and sent to downstream processing (e.g., evaporation, crystallization). The reactor operates continuously under steady-state conditions, with precise control to maintain product quality and safety.
- **Normal Flow Conditions**:
  - Nitric acid feed: 1000 kg/h ± 10% (63% HNO₃ solution).
  - Ammonia feed: Adjusted to maintain a molar ratio of 1.01:1 with nitric acid.
  - Product flow: ~1800 kg/h ammonium nitrate solution (80% concentration).
- **Operational Limits**:
  - Temperature: 170–180°C to prevent decomposition.
  - Pressure: 4.5–5.0 bar to ensure safe operation.
  - pH: 5.8–6.5 to avoid acidic or basic excursions.
  - Maximum feed rate: 1200 kg/h nitric acid to prevent runaway reaction.

#### 2. Key Setpoints and Ranges
The reactor’s control system maintains the following parameters within specified ranges to ensure product quality and safety:
- **Temperature (TIC-101)**:
  - Setpoint: 175°C.
  - Acceptable Range: 173–177°C (±2°C).
  - Purpose: Maintains optimal reaction rate while preventing thermal decomposition.
- **Pressure (PIC-102)**:
  - Setpoint: 4.8 bar.
  - Acceptable Range: 4.6–5.0 bar (±0.2 bar).
  - Purpose: Ensures safe containment and prevents vaporization.
- **Ammonia:Acid Ratio (FRC-103)**:
  - Setpoint: 1.01:1 molar ratio (slight ammonia excess).
  - Acceptable Range: 1.00–1.02:1.
  - Purpose: Ensures complete neutralization with minimal unreacted ammonia.
- **pH Control (AIC-105)**:
  - Setpoint: 6.2.
  - Acceptable Range: 5.8–6.5.
  - Purpose: Confirms neutralization and prevents corrosive conditions.
- **Product Flow (FIC-104)**:
  - Setpoint: 1800 kg/h.
  - Acceptable Range: 1700–1900 kg/h.
  - Purpose: Maintains steady-state operation and downstream processing capacity.

#### 3. Instrumentation and Equipment
The reactor control system relies on the following instrumentation and equipment to monitor and regulate the process:

- **Instrumentation**:
  - **TIC-101**: Temperature transmitter, reactor interior, 0–200°C range, controls cooling water valve (CV-101).
  - **PIC-102**: Pressure transmitter, reactor headspace, 0–10 bar range, controls vent valve (CV-102).
  - **FIC-103**: Ammonia flow transmitter, 0–200 kg/h, controls ammonia feed valve (CV-103).
  - **FIC-104**: Nitric acid flow transmitter, 0–1500 kg/h, controls nitric acid feed valve (CV-104).
  - **FIC-105**: Product flow transmitter, 0–2000 kg/h, controls product outlet valve (CV-105).
  - **AIC-105**: pH analyzer, reactor effluent, 0–14 pH range, adjusts ammonia feed via FRC-103.
  - **LT-106**: Level transmitter, reactor liquid level, 0–100%, monitors reactor inventory.
- **Control Equipment**:
  - **CV-101**: Cooling water valve, modulates flow to maintain TIC-101 setpoint.
  - **CV-102**: Vent valve, modulates to maintain PIC-102 setpoint.
  - **CV-103**: Ammonia feed valve, modulates for FRC-103 ratio control.
  - **CV-104**: Nitric acid feed valve, modulates for FIC-104 setpoint.
  - **CV-105**: Product outlet valve, modulates for FIC-105 setpoint.
  - **M-101**: Agitator motor, fixed speed (100 RPM) for uniform mixing.
  - **ESD-106**: Emergency shutoff valve, isolates all feeds on critical alarms.
- **Safety Interlocks**:
  - **PSH-107**: High-pressure switch, triggers at 5.2 bar, activates ESD-106.
  - **TSH-108**: High-temperature switch, triggers at 185°C, activates ESD-106.
  - **LSL-109**: Low-level switch, triggers at 20%, stops feed pumps to prevent cavitation.

#### 4. Sequence of Operation

##### Startup
The startup sequence initializes the reactor from a cold, empty state to steady-state operation, ensuring safe and controlled ramp-up.

1. **Pre-Start Checks**:
   - Verify all valves (CV-101 to CV-105, ESD-106) are closed.
   - Confirm M-101 (agitator) is operational.
   - Check LT-106 > 20% (minimum liquid level).
   - Reset all controllers (TIC-101, PIC-102, FRC-103, FIC-104, FIC-105, AIC-105) to manual mode.
   - **Logic**:
     ```st
     IF PreStartCheck THEN
         IF LT_106 > 20.0 AND M_101_Running AND NOT Fault THEN
             StartupActive := TRUE;
         ELSE
             Fault := TRUE;
         END_IF;
     END_IF;
     ```

2. **Preheat Reactor**:
   - Fill reactor with water to 50% level (LT-106 setpoint).
   - Start M-101 at 100 RPM.
   - Open CV-101 to circulate hot water, ramp TIC-101 to 150°C over 30 minutes.
   - **Logic**:
     ```st
     IF StartupActive AND Step = 1 THEN
         CV_104_Open := TRUE; (* Fill with water *)
         IF LT_106 >= 50.0 THEN
             CV_104_Open := FALSE;
             M_101_Speed := 100.0;
             TIC_101_Setpoint := 150.0;
             CV_101_Open := TRUE;
             PreheatTimer(IN := TRUE, PT := T#30m);
             IF PreheatTimer.Q THEN
                 Step := 2;
             END_IF;
         END_IF;
     END_IF;
     ```

3. **Initialize Feed Flows**:
   - Set FIC-104 to 500 kg/h nitric acid, open CV-104.
   - Set FRC-103 to 1.01:1 ratio, open CV-103 for ammonia.
   - Gradually increase FIC-104 to 1000 kg/h over 10 minutes.
   - **Logic**:
     ```st
     IF StartupActive AND Step = 2 THEN
         FIC_104_Setpoint := 500.0;
         FRC_103_Setpoint := 1.01;
         FeedRampTimer(IN := TRUE, PT := T#10m);
         FIC_104_Setpoint := FIC_104_Setpoint + (500.0 / 600.0) * FeedRampTimer.ET;
         IF FeedRampTimer.Q THEN
             FIC_104_Setpoint := 1000.0;
             Step := 3;
         END_IF;
     END_IF;
     ```

4. **Activate Control Loops**:
   - Switch TIC-101, PIC-102, FRC-103, FIC-104, FIC-105, and AIC-105 to auto mode.
   - Set TIC-101 to 175°C, PIC-102 to 4.8 bar, FIC-105 to 1800 kg/h, AIC-105 to 6.2.
   - Monitor LT-106 (40–60%) and adjust CV-105 as needed.
   - **Logic**:
     ```st
     IF StartupActive AND Step = 3 THEN
         TIC_101_Mode := AUTO;
         PIC_102_Mode := AUTO;
         FRC_103_Mode := AUTO;
         FIC_104_Mode := AUTO;
         FIC_105_Mode := AUTO;
         AIC_105_Mode := AUTO;
         TIC_101_Setpoint := 175.0;
         PIC_102_Setpoint := 4.8;
         FIC_105_Setpoint := 1800.0;
         AIC_105_Setpoint := 6.2;
         IF LT_106 < 40.0 THEN
             CV_105_Opening := CV_105_Opening - 0.01;
         ELSIF LT_106 > 60.0 THEN
             CV_105_Opening := CV_105_Opening + 0.01;
         END_IF;
         IF TIC_101_PV >= 173.0 AND PIC_102_PV >= 4.6 AND AIC_105_PV >= 5.8 THEN
             StartupActive := FALSE;
             SteadyStateActive := TRUE;
         END_IF;
     END_IF;
     ```

##### Steady-State Operation
The reactor operates continuously, with PID loops maintaining setpoints and alarms monitoring deviations.

- **Control Loops**:
  - TIC-101 modulates CV-101 to maintain 175°C.
  - PIC-102 modulates CV-102 to maintain 4.8 bar.
  - FRC-103 adjusts CV-103 to maintain 1.01:1 ammonia:acid ratio.
  - FIC-104 controls CV-104 at 1000 kg/h nitric acid.
  - FIC-105 controls CV-105 at 1800 kg/h product flow.
  - AIC-105 fine-tunes FRC-103 to maintain pH 6.2.
- **Monitoring**:
  - Log TIC-101, PIC-102, FRC-103, FIC-104, FIC-105, AIC-105, LT-106 every minute.
  - Alarm on deviations:
    - TIC-101 ± 5°C (High: 180°C, Low: 170°C).
    - PIC-102 ± 0.3 bar (High: 5.1 bar, Low: 4.5 bar).
    - AIC-105 ± 0.5 pH (High: 6.7, Low: 5.7).
- **Logic**:
  ```st
  IF SteadyStateActive THEN
      (* Maintain PID loops *)
      TIC_101_Control(CV_101);
      PIC_102_Control(CV_102);
      FRC_103_Control(CV_103);
      FIC_104_Control(CV_104);
      FIC_105_Control(CV_105);
      AIC_105_Control(FRC_103_Setpoint);
      
      (* Log data *)
      LogData(TIC_101_PV, PIC_102_PV, FRC_103_PV, FIC_104_PV, FIC_105_PV, AIC_105_PV, LT_106_PV);
      
      (* Deviation alarms *)
      IF TIC_101_PV > 180.0 OR TIC_101_PV < 170.0 THEN
          Alarm_Temperature := TRUE;
      END_IF;
      IF PIC_102_PV > 5.1 OR PIC_102_PV < 4.5 THEN
          Alarm_Pressure := TRUE;
      END_IF;
      IF AIC_105_PV > 6.7 OR AIC_105_PV < 5.7 THEN
          Alarm_pH := TRUE;
      END_IF;
  END_IF;
  ```

##### Shutdown
The shutdown sequence safely halts the reactor, isolating feeds and cooling the system.

1. **Isolate Feeds**:
   - Close CV-103 and CV-104 to stop ammonia and nitric acid feeds.
   - Set FRC-103, FIC-104 to manual mode, setpoints to 0.
   - **Logic**:
     ```st
     IF ShutdownActive AND Step = 1 THEN
         CV_103_Open := FALSE;
         CV_104_Open := FALSE;
         FRC_103_Mode := MANUAL;
         FIC_104_Mode := MANUAL;
         FRC_103_Setpoint := 0.0;
         FIC_104_Setpoint := 0.0;
         Step := 2;
     END_IF;
     ```

2. **Depressurize**:
   - Open CV-102 to vent reactor, reduce PIC-102 to 0.5 bar over 10 minutes.
   - **Logic**:
     ```st
     IF ShutdownActive AND Step = 2 THEN
         PIC_102_Setpoint := 0.5;
         CV_102_Open := TRUE;
         DepressurizeTimer(IN := TRUE, PT := T#10m);
         IF PIC_102_PV <= 0.5 AND DepressurizeTimer.Q THEN
             CV_102_Open := FALSE;
             Step := 3;
         END_IF;
     END_IF;
     ```

3. **Stop Heating and Cool**:
   - Close CV-101 to stop heating, circulate cold water to reduce TIC-101 to <50°C.
   - **Logic**:
     ```st
     IF ShutdownActive AND Step = 3 THEN
         CV_101_Open := FALSE;
         CoolingWaterPumpOn := TRUE;
         TIC_101_Setpoint := 50.0;
         IF TIC_101_PV <= 50.0 THEN
             CoolingWaterPumpOn := FALSE;
             Step := 4;
         END_IF;
     END_IF;
     ```

4. **Drain Reactor**:
   - Open CV-105 to drain contents, stop M-101 when LT-106 < 10%.
   - **Logic**:
     ```st
     IF ShutdownActive AND Step = 4 THEN
         CV_105_Open := TRUE;
         IF LT_106 < 10.0 THEN
             CV_105_Open := FALSE;
             M_101_Speed := 0.0;
             ShutdownActive := FALSE;
             ShutdownComplete := TRUE;
         END_IF;
     END_IF;
     ```

#### 5. Interlocks and Alarms

- **Interlocks**:
  - **High Temperature (TIC-101 > 185°C or TSH-108)**:
    - Activate ESD-106 to close CV-103, CV-104, CV-105.
    - Open CV-102 to vent pressure.
    - Stop M-101 and cooling water pump.
    - **Logic**:
      ```st
      IF TIC_101_PV > 185.0 OR TSH_108 THEN
          ESD_106_Activate := TRUE;
          CV_103_Open := FALSE;
          CV_104_Open := FALSE;
          CV_105_Open := FALSE;
          CV_102_Open := TRUE;
          M_101_Speed := 0.0;
          CoolingWaterPumpOn := FALSE;
          Fault := TRUE;
      END_IF;
      ```
  - **High Pressure (PIC-102 > 5.2 bar or PSH-107)**:
    - Close CV-103, CV-104 to stop feeds.
    - Open CV-102 to vent.
    - **Logic**:
      ```st
      IF PIC_102_PV > 5.2 OR PSH_107 THEN
          CV_103_Open := FALSE;
          CV_104_Open := FALSE;
          CV_102_Open := TRUE;
          Fault := TRUE;
      END_IF;
      ```
  - **Low Level (LSL-109 or LT-106 < 20%)**:
    - Stop feed pumps (CV-103, CV-104) to prevent cavitation.
    - **Logic**:
      ```st
      IF LT_106 < 20.0 OR LSL_109 THEN
          CV_103_Open := FALSE;
          CV_104_Open := FALSE;
          Fault := TRUE;
      END_IF;
      ```

- **Alarms**:
  - **Low pH (AIC-105 < 5.5)**:
    - Alarm and reduce ammonia flow by 10%.
    - **Logic**:
      ```st
      IF AIC_105_PV < 5.5 THEN
          Alarm_Low_pH := TRUE;
          FRC_103_Setpoint := FRC_103_Setpoint * 0.9;
      END_IF;
      ```
  - **High pH (AIC-105 > 6.7)**:
    - Alarm and reduce ammonia flow by 5%.
    - **Logic**:
      ```st
      IF AIC_105_PV > 6.7 THEN
          Alarm_High_pH := TRUE;
          FRC_103_Setpoint := FRC_103_Setpoint * 0.95;
      END_IF;
      ```
  - **Temperature Deviation (TIC-101 > 180°C or < 170°C)**:
    - Alarm to alert operators of potential process upset.
  - **Pressure Deviation (PIC-102 > 5.1 bar or < 4.5 bar)**:
    - Alarm to indicate pressure control issues.
  - **Flow Deviation (FIC-104 or FIC-105 ± 10%)**:
    - Alarm to detect feed or product flow anomalies.

#### Integration Notes
The control narrative is designed for implementation in a PLC-based distributed control system (e.g., Siemens PCS7, Rockwell PlantPAx). The sequence assumes a PLC with analog input/output capabilities, real-time clock, and PID control blocks. Each step executes within a single scan cycle (<50 ms), except for timed ramps (e.g., preheat, depressurize), which span multiple cycles using timers. Interlocks and alarms comply with safety standards (e.g., IEC 61511), ensuring protection against runaway reactions or equipment damage. The narrative supports operator interaction via HMI, displaying setpoints, process values, and alarms, with manual override capabilities for maintenance.

This control narrative provides a comprehensive, standards-compliant framework for automating the ammonium nitrate reactor, ensuring precise control, robust safety, and clear operational guidance for startup, steady-state, and shutdown phases.
