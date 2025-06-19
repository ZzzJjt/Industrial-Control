+---------------------+
| Water Tank          |
|                     |
|  WT101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Nutrient Mixer      |
|                     |
|  NM101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Sterilizer          |
|                     |
|  ST101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Fermenter           |
|                     |
|  FR101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Separator           |
|                     |
|  SP101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Product Tank        |
|                     |
|  PT101              |
+---------------------+

Instruments and Control Loops:

Water Tank:
- WT101: Level Transmitter

Nutrient Mixer:
- NM101: Agitator Speed Sensor

Sterilizer:
- ST101: Temperature Transmitter
- TT201 → TIC201 → Steam Control Valve (TCV201)

Fermenter:
- FT301: Flow Transmitter
- LT401: Level Transmitter
- pH201: pH Transmitter
- AT501: Agitator Speed Sensor
- TT601: Temperature Transmitter

  Control Loops:
  - FT301 → FIC301 → Feed Control Valve (FCV301)
  - LT401 → LIC401 → Level Control Valve (LCV401)
  - pH201 → PIC201 → pH Adjustment Pump (PP201)
  - AT501 → AIC501 → Agitator Motor (AM501)
  - TT601 → TIC601 → Heating Jacket (HJ601)

Separator:
- SP101: Agitator Speed Sensor

Product Tank:
- PT101: Level Transmitter
- LT701: Level Transmitter

Control Philosophy:

1. **Temperature Control in Sterilizer**:
   - The temperature in the sterilizer is monitored by TT201.
   - A PID controller (TIC201) adjusts the steam control valve (TCV201) to maintain the setpoint temperature.

2. **Flow Control to Fermenter**:
   - The flow of nutrient mixture into the fermenter is monitored by FT301.
   - A PID controller (FIC301) adjusts the feed control valve (FCV301) to maintain the setpoint flow rate.

3. **Level Control in Fermenter**:
   - The level in the fermenter is monitored by LT401.
   - A PID controller (LIC401) adjusts the level control valve (LCV401) to maintain the setpoint level.

4. **pH Control in Fermenter**:
   - The pH in the fermenter is monitored by pH201.
   - A PID controller (PIC201) adjusts the pH adjustment pump (PP201) to maintain the setpoint pH.

5. **Agitation Control in Fermenter**:
   - The agitator speed in the fermenter is monitored by AT501.
   - A PID controller (AIC501) adjusts the agitator motor (AM501) to maintain the setpoint agitation speed.

6. **Temperature Control in Fermenter**:
   - The temperature in the fermenter is monitored by TT601.
   - A PID controller (TIC601) adjusts the heating jacket (HJ601) to maintain the setpoint temperature.

7. **Level Control in Product Tank**:
   - The level in the product tank is monitored by LT701.
   - A simple level transmitter (LT701) provides real-time level data for operator monitoring.

This PFD provides a clear and structured representation of the penicillin fermentation process, including major equipment, flow directions, key instrumentation, and control loops, supporting automation design and process control implementation.
