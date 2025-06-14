### Automated Beer Brewing Process Control Narrative

#### 1. Overview
The automated beer brewing process involves several key stages: milling, mashing, lautering, boiling, fermentation, and conditioning. Each stage has specific setpoints and control ranges to ensure consistent quality and operational reliability.

#### 2. Milling Phase
- **Setpoints**:
  - Grind Size: Medium (target grit size of 0.8 mm)
- **Control Ranges**:
  - Grind Size: 0.75 mm to 0.85 mm

#### 3. Mashing Phase
- **Setpoints**:
  - Mash Temperature: 65°C ± 1°C
  - Mash Time: 60 minutes
- **Control Ranges**:
  - Mash Temperature: 64°C to 66°C
  - Mash pH: 5.2 to 5.6

#### 4. Lautering Phase
##### Required Equipment
- Lauter Tun
- Rake Motor
- Sparge System
- Wort Pump

##### Required Instrumentation
- Turbidity Meter
- Flow Transmitter
- Level Sensor
- Temperature Probe

##### Step-by-Step Procedure
1. **Start Recirculation of Wort**
   - Activate the wort pump to circulate the wort in the lauter tun.
   - Ensure the flow rate is within the range of 20 to 30 liters per minute.

2. **Monitor Turbidity**
   - Continuously monitor the turbidity using the turbidity meter.
   - Wait until the turbidity drops below 200 NTU.

3. **Begin Transferring Wort to the Kettle**
   - Once the turbidity is below 200 NTU, activate the wort pump to transfer the wort to the kettle.
   - Maintain a flow rate between 20 to 30 liters per minute.

4. **Add Sparge Water at 76°C**
   - Begin adding sparge water to the lauter tun through the sparge system.
   - Maintain the sparge water temperature at 76°C ± 1°C.
   - Keep the lauter tun level between 60% and 80%.

5. **Stop Process Based on Turbidity or Volume Criteria**
   - Continue monitoring turbidity and volume.
   - Stop the process when either the turbidity reaches 50 NTU or the desired volume of wort has been transferred.

##### Interlock Logic
- **Turbidity Monitoring**
  - IF Turbidity > 200 NTU THEN open waste valve.
  - IF Turbidity < 200 NTU THEN close waste valve.

- **Level Monitoring**
  - IF Level < 60% THEN stop sparging.
  - IF Level > 80% THEN reduce sparge water flow rate.

- **Temperature Monitoring**
  - IF Sparge Water Temperature > 77°C THEN reduce sparge water temperature.
  - IF Sparge Water Temperature < 75°C THEN increase sparge water temperature.

#### 5. Boiling Phase
- **Setpoints**:
  - Boil Time: 60 minutes
  - Boil Temperature: 98°C ± 1°C
- **Control Ranges**:
  - Boil Temperature: 97°C to 99°C

#### 6. Fermentation Phase
- **Setpoints**:
  - Fermentation Temperature: 18°C ± 1°C
- **Control Ranges**:
  - Fermentation Temperature: 17°C to 19°C

#### 7. Conditioning Phase
- **Setpoints**:
  - Conditioning Temperature: 4°C ± 1°C
- **Control Ranges**:
  - Conditioning Temperature: 3°C to 5°C

### Summary
This control narrative provides a structured approach to automating the beer brewing process, focusing particularly on the lautering phase. It includes specific setpoints, control ranges, required equipment and instrumentation, step-by-step procedures, and interlock logic to ensure consistency, quality, and operational reliability throughout the brewing process.

By following this narrative, control engineers, brewers, and programmers can implement an automated system that maintains precise control over each stage, leading to high-quality beer production.
