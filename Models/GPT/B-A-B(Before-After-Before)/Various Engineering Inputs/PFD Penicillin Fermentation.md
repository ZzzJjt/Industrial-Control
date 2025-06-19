───────────────────── Penicillin Fermentation – Text-Based PFD ─────────────────────

[Water Tank] (WT01)
   |
   |--> [Nutrient Mixing Tank] (MT01)
   |        ├── TT101: Temperature Transmitter
   |        └── AGL102: Agitator Motor
   |              → Agitation via local VFD, no closed-loop control
   |
   |--> [Sterilizer] (ST01)
   |        ├── TT102: Sterilization Temp Transmitter
   |        └── TIC102: Temp PID Controller
   |              → Controls heating valve HV102 for sterilization at 121 °C
   |
   |--> [Seed Fermenter] (F01)
   |        ├── TT201: Fermenter Temperature Transmitter
   |        ├── TIC201: Temp Controller → TC201: Jacket Control Valve
   |        ├── pH201: pH Probe
   |        ├── PIC202: Pressure Indicator Controller
   |        └── AGL201: Agitation Motor
   |
   |--> [Main Fermenter] (F02)
   |        ├── TT301: Temperature Transmitter
   |        ├── TIC301: Temp Controller → TC301: Jacket Cooling Valve
   |        ├── pH302: pH Probe → pH Controller → Acid/Base Dosing Valves
   |        ├── LT303: Level Transmitter
   |        └── AGL302: Variable-Speed Agitation
   |
   |--> [Filtration Unit] (FU01)
   |        ├── DP401: Pressure Differential across filter
   |        └── FIC402: Flow Control Valve for filtrate
   |
   |--> [Solvent Extraction System] (SE01)
   |        ├── Mixer → Settler
   |        └── TIC501: Temp Control for Solvent Loop
   |
   |--> [Crystallizer] (CR01)
   |        ├── TT601: Crystallizer Temp Sensor
   |        └── TIC601: Cooling Jacket Control
   |
   |--> [Centrifuge] (CF01)
   |        └── VFD Control via local panel
   |
   |--> [Product Storage Tank] (TK02)
            └── LT701: Product Level Transmitter

────────────────────────────────────────────────────────────────────────────────────

🛠 Control Philosophy Summary:

• Temperature Control:
   - All fermenters (F01, F02) use PID loops:
     TT → TIC → Jacket Valve (TC)
   - Sterilizer controlled at 121 °C via TIC102

• pH Control:
   - Online pH probes (pH201, pH302)
   - Closed-loop acid/base dosing valves activated when deviation exceeds ±0.2 pH

• Agitation:
   - All agitators VFD-controlled
   - Manual speed setting except main fermenter (F02), which uses logic-based speed ramping

• Level Control:
   - LT303, LT701 used for level monitoring and alarm generation

• Flow/Filtration:
   - FIC402 maintains target filtrate flow during filtration cycle

• Final Transfer:
   - Automated interlock from crystallizer outlet to product tank based on level and purity thresholds

────────────────────────────────────────────────────────────────────────────────────
