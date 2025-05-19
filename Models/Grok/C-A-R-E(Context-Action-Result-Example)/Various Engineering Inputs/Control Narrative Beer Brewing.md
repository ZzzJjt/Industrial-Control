I'll develop a structured control narrative for the entire beer brewing process, with a detailed Section 4 focused on the lautering phase, including setpoints, equipment, instrumentation, step-by-step logic, and automated control actions. The narrative will be practical, actionable, and designed to standardize operations for consistent beer quality.

### Control Narrative for Beer Brewing Process

#### Overview
The beer brewing process consists of several stages: milling, mashing, lautering, boiling, fermentation, conditioning, and packaging. Each stage requires precise control of temperature, flow, pressure, and other parameters to ensure product quality and process efficiency. This control narrative outlines the automation logic, setpoints, and instrumentation for all stages, with a detailed focus on lautering, to provide a shared reference for engineering and operations teams.

#### Section 1: Milling
- **Objective**: Grind malt to achieve consistent particle size for optimal mash extraction.
- **Setpoints**:
  - Roller gap: 0.8–1.2 mm
  - Feed rate: 500 kg/h
- **Equipment & Instrumentation**:
  - Roller mill with adjustable gap
  - Mass flow sensor for feed rate
  - Motor VFD for mill speed control
- **Steps**:
  1. Set roller gap based on malt type (via HMI input).
  2. Start mill and adjust VFD to maintain feed rate.
  3. Monitor particle size distribution (manual sampling or optional inline analyzer).
- **Automation**:
  - PID control of VFD speed based on flow sensor feedback.
  - Alarm if feed rate deviates by ±10%.

#### Section 2: Mashing
- **Objective**: Convert starches in malt to fermentable sugars through enzymatic activity.
- **Setpoints**:
  - Mash temperature: 65–68°C (recipe-dependent)
  - Mash duration: 60–90 min
  - pH: 5.2–5.6
- **Equipment & Instrumentation**:
  - Mash tun with steam jacket
  - Temperature sensors (PT100)
  - pH probe
  - Agitator with VFD
- **Steps**:
  1. Heat water to strike temperature (e.g., 72°C).
  2. Add malt and water at specified ratio (e.g., 3:1 L/kg).
  3. Maintain mash temperature via steam valve control.
  4. Agitate to ensure homogeneity.
  5. Complete mash rest and prepare for lautering.
- **Automation**:
  - PID control of steam valve for temperature stability.
  - pH adjustment via automated dosing pump if out of range.
  - Timer for mash duration with operator override.

#### Section 3: Lautering (Detailed in Section 4 Below)

#### Section 4: Lautering
- **Objective**: Separate clear wort from spent grain while maximizing sugar extraction.
- **Setpoints**:
  - Mash rest temperature: 67 ± 1°C
  - Lautering flow rate: 1.8 ± 0.2 L/min
  - Turbidity cut-off: < 200 NTU (Nephelometric Turbidity Units)
  - Sparge water temperature: 76 ± 1°C
  - Lauter tun level: 30–50% of capacity
  - Wort collection volume: Recipe-dependent (e.g., 1000 L)
- **Equipment & Instrumentation**:
  - Lauter tun with false bottom and motorized rake arms
  - Turbidity sensor (inline) at wort outlet
  - Flow transmitter (electromagnetic) on wort and sparge water lines
  - Level transmitter (ultrasonic) in lauter tun
  - Temperature sensors (PT100) on lauter tun and sparge water
  - Motorized valve for wort diversion (to kettle or waste)
  - VFD-controlled pump for wort transfer
  - Sparge water heater with steam valve
  - Rake arm motor with position feedback
- **Steps**:
  1. **Vorlauf (Recirculation)**:
     - Maintain mash temperature at 67°C using steam jacket.
     - Start wort recirculation at 1.0 L/min through false bottom.
     - Monitor turbidity sensor; divert wort to waste if turbidity ≥ 200 NTU.
     - Continue until turbidity < 200 NTU for 2 consecutive minutes.
  2. **Wort Transfer**:
     - Open motorized valve to wort kettle.
     - Increase flow rate to 1.8 L/min using VFD pump.
     - Monitor flow transmitter to maintain setpoint.
  3. **Sparging**:
     - Start sparge water flow at 1.8 L/min, heated to 76°C.
     - Adjust sparge flow to maintain lauter tun level at 30–50% (via level transmitter).
     - Monitor wort flow resistance (pressure drop across grain bed); adjust rake arm position if resistance exceeds 0.5 bar.
  4. **Grain Bed Management**:
     - Rotate rake arms at 2 RPM to prevent grain bed compaction.
     - Raise rake arms incrementally (e.g., 5 cm every 10 min) as grain bed settles.
     - Stop rake arms if flow resistance normalizes.
  5. **Completion**:
     - Stop sparging when grain bed is dry (level < 10% or flow resistance > 1.0 bar).
     - Verify total wort volume collected (via kettle level sensor).
     - Close wort valve and prepare for grain removal.
- **Automation**:
  - **Turbidity Control**: PID loop adjusts wort recirculation valve based on turbidity sensor feedback. If turbidity ≥ 200 NTU, divert wort to waste; if < 200 NTU, route to kettle.
  - **Flow Control**: PID loop on VFD pump maintains wort flow at 1.8 L/min, using flow transmitter feedback.
  - **Level Control**: PID loop adjusts sparge water valve to maintain lauter tun level at 30–50%.
  - **Temperature Control**: PID loop on steam valve maintains sparge water at 76°C and mash at 67°C.
  - **Rake Arm Adjustment**: Automated sequence raises rake arms based on pressure drop (flow resistance) or timer, with operator override via HMI.
  - **Interlocks**:
    - Stop wort transfer if turbidity ≥ 200 NTU after 10 min of recirculation (alarm triggered).
    - Halt sparging if lauter tun level < 10% or > 70% to prevent overflow or dry run.
    - Disable rake arm rotation if motor fault detected.
  - **Operator Inputs**:
    - Start/stop lautering via HMI.
    - Override rake arm position or sparge flow rate.
    - Confirm wort volume and grain bed dry state.
- **Monitoring**:
  - Log turbidity, flow rate, level, and temperature every 30 seconds.
  - Display real-time data on HMI, including alarms for out-of-range conditions.

#### Section 5: Boiling
- **Objective**: Sterilize wort, extract hop flavors, and concentrate sugars.
- **Setpoints**:
  - Boil temperature: 100°C
  - Boil duration: 60–90 min
  - Hop addition timing: Recipe-dependent (e.g., 0, 30, 60 min)
- **Equipment & Instrumentation**:
  - Brew kettle with steam jacket
  - Temperature and pressure sensors
  - Automated hop dosing system
- **Steps**:
  1. Heat wort to 100°C.
  2. Maintain boil with steam valve control.
  3. Add hops at specified times.
  4. Cool wort post-boil using heat exchanger.
- **Automation**:
  - PID control of steam valve for boil temperature.
  - Timer-based hop dosing with operator confirmation.

#### Section 6: Fermentation
- **Objective**: Convert sugars to alcohol and CO2 using yeast.
- **Setpoints**:
  - Fermentation temperature: 18–22°C (ale), 8–12°C (lager)
  - Fermentation duration: 7–14 days
  - CO2 pressure: 1–2 bar
- **Equipment & Instrumentation**:
  - Fermentation tank with cooling jacket
  - Temperature sensors
  - Pressure transmitter
  - Yeast dosing system
- **Steps**:
  1. Cool wort to fermentation temperature.
  2. Pitch yeast at specified rate.
  3. Maintain temperature and pressure.
  4. Monitor specific gravity to determine completion.
- **Automation**:
  - PID control of cooling jacket for temperature.
  - Pressure relief valve control for CO2.

#### Section 7: Conditioning and Packaging
- **Objective**: Mature beer and package for distribution.
- **Setpoints**:
  - Conditioning temperature: 0–4°C
  - Carbonation level: 2.4–2.8 volumes CO2
- **Equipment & Instrumentation**:
  - Conditioning tanks
  - Bottling/kegging line
  - CO2 injection system
- **Steps**:
  1. Transfer beer to conditioning tanks.
  2. Maintain temperature and carbonation.
  3. Package beer into bottles or kegs.
- **Automation**:
  - Automated transfer and packaging sequences.
  - CO2 injection control based on sensor feedback.

### Expected Outcome

This control narrative provides a comprehensive, standardized reference for the beer brewing process, with a detailed focus on the lautering phase. It ensures:
- **Consistency**: Precise setpoints and automated controls produce uniform wort quality.
- **Clarity**: Detailed equipment, instrumentation, and logic descriptions align engineering and operations teams.
- **Efficiency**: Automation minimizes manual intervention and optimizes lautering flow and sparging.
- **Integration**: Modular design supports seamless integration with PLCs, HMIs, and process equipment.

The lautering section, in particular, offers actionable logic for turbidity control, flow management, and grain bed handling, making it a practical tool for breweries aiming to enhance automation and product quality.
