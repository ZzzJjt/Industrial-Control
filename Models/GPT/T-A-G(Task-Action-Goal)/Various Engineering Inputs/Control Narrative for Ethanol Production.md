Section 3: Fermentation

⸻

3.1 Inoculation

Objective: Introduce yeast culture into the sterilized fermentation broth under controlled conditions.
	•	Setpoints: Broth temperature at inoculation = 32 °C ± 1 °C
	•	Control Method: Pre-inoculation verification logic ensures the fermenter is at correct temperature, pH is between 4.5–5.0, and dissolved oxygen is below 0.5 mg/L.
	•	Automation: When all setpoints are within acceptable ranges, and inoculation valve interlock is cleared, the inoculation pump is triggered. A countdown timer verifies complete addition.
	•	Interlocks:
	•	Inoculation prohibited if temperature > 35 °C
	•	Alarm triggered if dosing time exceeds 5 minutes

⸻

3.2 Temperature Control

Objective: Maintain optimal fermentation temperature to promote yeast activity while preventing thermal stress.
	•	Setpoint Band: 32–35 °C
	•	Control Method: PID loop using TIC-301, controlling chilled water flow to jacketed fermenter coil.
	•	Sensors and Equipment:
	•	RTD sensor for real-time temperature feedback
	•	Control valve on chilled water inlet
	•	Alarms:
	•	High-temp alarm at >36 °C
	•	Low-temp alarm at <31 °C
	•	Failsafe Logic:
	•	If cooling valve fails open or closed, triggers redundant backup chiller line with override signal.

⸻

3.3 pH Adjustment

Objective: Maintain slightly acidic conditions optimal for yeast metabolism.
	•	Setpoint Band: pH 4.5–5.0
	•	Control Method: PID loop via AIC-302 using pH probe feedback; controlled dosing of dilute H₂SO₄ or NaOH through metering pumps.
	•	Sensors and Actuators:
	•	Inline pH sensor with auto-calibration capability
	•	Acid/base dosing pumps
	•	Interlocks and Alarms:
	•	If pH < 4.2 or > 5.3, trigger deviation alarm
	•	Dosing interlocked with agitator status (only adjust if mixing is active)

⸻

3.4 Agitation

Objective: Ensure homogeneous mixture of nutrients and yeast; promote uniform mass and heat transfer.
	•	Setpoint: Agitator speed = 120–200 RPM depending on fermentation stage
	•	Control Method: VFD (variable frequency drive) controlled via LIC-303 with manual or auto mode.
	•	Automation Details:
	•	Agitation ramped up gradually after inoculation
	•	RPM adjusted based on dissolved oxygen level and CO₂ evolution rate
	•	Interlocks:
	•	Agitator disabled if fermenter level < 20%
	•	Alarm triggered on motor overcurrent, speed deviation >10%

⸻

3.5 Foam Control

Objective: Prevent overflow and ensure gas exchange is not hindered.
	•	Control Method: On/off logic driven by foam sensor signal.
	•	Setpoints:
	•	Foam height threshold: 80% of tank height
	•	Max defoamer dosing: 100 mL per cycle
	•	Automation:
	•	When foam probe is triggered, initiate defoamer dosing pump for 10 seconds
	•	Delay timer enforces minimum 2-minute gap between successive doses
	•	Safety Logic:
	•	If foam level > 90%, pause agitation and raise critical alarm
	•	Manual override available via HMI

 
