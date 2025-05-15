Certainly! Below is a structured control narrative for the entire beer brewing process, with a detailed focus on the lautering phase. This narrative includes setpoints and ranges, equipment and instrumentation, step-by-step process logic, and automated control actions.

### Beer Brewing Control Narrative

#### Overview
This control narrative outlines the standardized procedures and automation logic for each stage of the beer brewing process, ensuring precision and consistency in brewing operations. The document serves as a shared reference for engineering and operations teams to integrate control systems with process equipment.

---

### Section 1: Mash Tun Process

**Setpoints:**
- Mash rest temperature: 67°C ± 2°C
- Mash pH: 5.2 - 5.8

**Equipment & Instrumentation:**
- Mash tun with heating jacket
- Temperature sensor at bottom of mash tun
- pH meter at bottom of mash tun
- Stirring motor with speed controller

**Steps:**
1. Heat strike water to target temperature.
2. Mix strike water with crushed grains in mash tun.
3. Maintain temperature within setpoint using heating jacket.
4. Monitor pH and adjust with acid/base solutions if necessary.
5. Begin mashing by stirring periodically.
6. Hold mash temperature for specified time (e.g., 60 minutes).

**Automated Control Actions:**
- Temperature PID controller adjusts heating jacket power based on sensor feedback.
- pH controller adds acid/base solutions based on pH sensor readings.

---

### Section 2: Boiling Process

**Setpoints:**
- Boiling temperature: 98°C ± 1°C
- Boiling duration: 60 minutes

**Equipment & Instrumentation:**
- Boiler with heating element
- Temperature sensor at boiler outlet
- Timer for boiling duration

**Steps:**
1. Transfer mashed wort to boiler.
2. Bring wort to boiling point.
3. Maintain boiling temperature for specified duration.
4. Add hops at scheduled intervals.
5. Remove wort from boiler after boiling.

**Automated Control Actions:**
- Temperature PID controller maintains boiling temperature.
- Timer starts and stops boiling process.

---

### Section 3: Cooling Process

**Setpoints:**
- Target cooling temperature: 75°C ± 2°C
- Cooling rate: ≤ 2°C/min

**Equipment & Instrumentation:**
- Plate heat exchanger or chiller
- Temperature sensor before and after heat exchanger/chiller
- Flow transmitter on coolant line

**Steps:**
1. Transfer boiled wort to plate heat exchanger/chiller.
2. Cool wort to target temperature.
3. Ensure cooling rate does not exceed limits.

**Automated Control Actions:**
- Temperature PID controller adjusts coolant flow rate based on sensor feedback.
- Rate-of-change monitor ensures cooling rate compliance.

---

### Section 4: Lautering Process

**Setpoints:**
- Recirculation turbidity cut-off: < 200 NTU
- Wort flow rate: 1.8 L/min ± 0.2 L/min
- Sparge water temperature: 76°C ± 2°C
- Total wort volume: 200 L ± 5 L

**Equipment & Instrumentation:**
- Lauter tun with rake arms and false bottom
- Turbidity sensor at wort outlet
- Flow transmitter on wort and sparge lines
- Level transmitter in lauter tun
- Motorized valve to divert cloudy wort to waste
- Temperature sensor for sparge water

**Steps:**
1. Begin recirculation until turbidity < 200 NTU.
2. Start wort transfer to kettle.
3. Begin sparging at 76°C while maintaining lauter tun level.
4. Adjust rake arm position based on wort flow resistance.
5. Stop sparging when grain bed is dry and total volume reached.

**Automated Control Actions:**
- **Recirculation:**
  - Turbidity sensor monitors wort clarity.
  - Motorized valve diverts cloudy wort to waste.
  - Controller continues recirculation until turbidity < 200 NTU.
  
- **Wort Transfer:**
  - Flow transmitter monitors wort flow rate.
  - Valve opens to transfer wort to kettle.
  - Controller maintains flow rate within setpoint.
  
- **Sparging:**
  - Temperature sensor monitors sparge water temperature.
  - Valve opens to introduce sparge water.
  - Controller maintains sparge water temperature within setpoint.
  
- **Level Control:**
  - Level transmitter monitors lauter tun level.
  - Controller adjusts sparge water flow to maintain constant level.
  
- **Rake Arm Adjustment:**
  - Operator manually adjusts rake arm position based on observed flow resistance.
  
- **Completion Criteria:**
  - Level transmitter detects grain bed drying.
  - Volume sensor confirms total wort volume reached.
  - Controller stops sparging and closes valves.

---

### Section 5: Fermentation Process

**Setpoints:**
- Fermentation temperature: 18°C ± 2°C
- CO₂ production rate: ≥ 0.5 L/hour per liter of wort

**Equipment & Instrumentation:**
- Fermenter with cooling jacket
- Temperature sensor at fermenter bottom
- CO₂ sensor at fermenter headspace
- Airlock for CO₂ venting

**Steps:**
1. Transfer cooled wort to fermenter.
2. Pitch yeast into fermenter.
3. Maintain fermentation temperature.
4. Monitor CO₂ production rate.
5. Vent excess CO₂ through airlock.

**Automated Control Actions:**
- Temperature PID controller adjusts cooling jacket power based on sensor feedback.
- CO₂ sensor monitors production rate.

---

### Section 6: Packaging Process

**Setpoints:**
- Packaging temperature: 10°C ± 2°C
- Carbonation pressure: 2.5 bar ± 0.2 bar

**Equipment & Instrumentation:**
- Packaging machine
- Temperature sensor at packaging area
- Pressure sensor in keg/bottle
- Filling pump with flow control

**Steps:**
1. Transfer fermented beer to packaging machine.
2. Fill kegs/bottles to specified volume.
3. Apply carbonation pressure.
4. Seal kegs/bottles.
5. Store packaged beer.

**Automated Control Actions:**
- Temperature PID controller maintains packaging temperature.
- Pressure sensor monitors and controls carbonation pressure.
- Flow transmitter monitors filling rate.

---

### Conclusion

This control narrative provides a comprehensive guide for automating the beer brewing process, ensuring consistent quality and efficiency. Each section includes specific setpoints, instrumentation, and automated control actions, making it easier for engineering and operations teams to implement and maintain the control system.

By following this narrative, breweries can achieve precise and repeatable results, leading to higher-quality beer and reduced variability in production processes.
