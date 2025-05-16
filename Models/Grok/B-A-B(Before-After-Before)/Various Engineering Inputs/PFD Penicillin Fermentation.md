```
Text-Based Process Flow Diagram (PFD) for Penicillin Fermentation Process
------------------------------------------------------------------------
Description: Represents the penicillin fermentation process in a plain-text
format, detailing major unit operations, material flow, key instrumentation,
and control philosophy. The process converts raw materials (water, nutrients,
yeast) into penicillin through media preparation, sterilization, fermentation,
filtration, and recovery, with automated control for optimal yield and safety.

Notation:
- Equipment: WT01 (Water Tank), FM01 (Fermenter), etc.
- Instruments: TT (Temperature Transmitter), pH (pH Sensor), FIC (Flow Indicating Controller).
- Flow: --> indicates material flow direction.
- Control: [tag:controller-->actuator] denotes control loop (e.g., TT201:TIC201-->TC201).
- Piping: Labeled (e.g., W1 for Water Line 1, M1 for Media Line 1).
- Format: Indented blocks by process stage, with equipment, instruments, flow, and control.

Text-Based PFD:
===============

1. Media Preparation
-------------------
  - Water Tank (WT01): Stores purified water for media.
    - LT101 (Level Transmitter, 0–100%): Monitors water level.
    - LIC101 (Level Controller): Setpoint 80%, LIC101-->LV101 (makeup valve).
    - TT102 (Temperature Transmitter, 0–50°C): Monitors water temperature.
  - Piping: W1 (Water Line 1) --> [LT101:LV101] --> WT01
  - Control: Maintains water level at 80% via PID-controlled makeup valve.

  - Nutrient Tank (NT01): Stores glucose, nitrogen, and salts.
    - LT103 (Level Transmitter, 0–100%): Monitors nutrient level.
    - WT104 (Weight Transmitter, 0–1000 kg): Monitors nutrient mass.
    - FT105 (Flow Transmitter, 0–500 L/h): Monitors nutrient feed flow.
    - FIC105 (Flow Controller): Setpoint 200 L/h, FIC105-->FV105 (nutrient valve).
  - Piping: N1 (Nutrient Line 1) --> [FT105:FV105] --> NT01
  - Control: Meters nutrient feed at 200 L/h to ensure consistent media composition.

  - Mixer (MX01): Blends water and nutrients to form media.
    - FT106 (Flow Transmitter, 0–1000 L/h): Monitors water inflow.
    - FIC106 (Flow Controller): Setpoint 800 L/h, FIC106-->FV106 (water valve).
    - ST107 (Speed Transmitter, 0–200 RPM): Monitors mixer speed.
    - SIC107 (Speed Controller): Setpoint 100 RPM, SIC107-->SV107 (VFD).
  - Piping: WT01 --> [FIC106:FV106] --> MX01
  - Piping: NT01 --> [FIC105:FV105] --> MX01
  - Piping: MX01 --> M1 (Media Line 1)
  - Control: Blends water (800 L/h) and nutrients (200 L/h) at 100 RPM for homogeneous media.

2. Sterilization
----------------
  - Sterilizer (ST01): Heats media to 121°C for 30 min to eliminate contaminants.
    - TT108 (Temperature Transmitter, 0–150°C): Monitors media temperature.
    - TIC108 (Temperature Controller): Setpoint 121°C, TIC108-->TC108 (steam valve).
    - PT109 (Pressure Transmitter, 0–5 bar): Monitors sterilizer pressure.
    - FT110 (Flow Transmitter, 0–500 L/h): Monitors media inflow.
    - FIC110 (Flow Controller): Setpoint 300 L/h, FIC110-->FV110 (media valve).
  - Piping: M1 --> [FIC110:FV110] --> ST01
  - Piping: ST01 --> M2 (Sterilized Media Line)
  - Control: Maintains 121°C and 2 bar for 30 min via PID-controlled steam valve; interlock stops FV110 if TAH-108 (>125°C).

3. Fermentation
---------------
  - Fermenter (FM01): Converts sugars to penicillin via yeast fermentation.
    - TT201 (Temperature Transmitter, 0–50°C): Monitors fermenter temperature.
    - TIC201 (Temperature Controller): Setpoint 34°C, TIC201-->TC201 (cooling valve).
    - pH202 (pH Sensor, 0–14): Monitors fermenter pH.
    - AIC202 (pH Controller): Setpoint 5.0, AIC202-->AV202 (acid/base valve).
    - ST203 (Speed Transmitter, 0–200 RPM): Monitors agitator speed.
    - SIC203 (Speed Controller): Setpoint 90 RPM, SIC203-->SV203 (VFD).
    - LT204 (Level Transmitter, 0–100%): Monitors fermenter level.
    - LIC204 (Level Controller): Setpoint 75%, LIC204-->LV204 (media valve).
    - FT205 (Flow Transmitter, 0–500 L/h): Monitors media inflow.
    - FIC205 (Flow Controller): Setpoint 250 L/h, FIC205-->FV205 (media valve).
    - AT206 (Dissolved Oxygen Sensor, 0–100%): Monitors DO level.
    - AIC206 (DO Controller): Setpoint 30%, AIC206-->AV206 (air sparge valve).
    - FT207 (CO2 Flow Transmitter, 0–500 L/min): Monitors off-gas CO2.
  - Piping: M2 --> [FIC205:FV205] --> FM01
  - Piping: Air --> [AIC206:AV206] --> FM01
  - Piping: FM01 --> P1 (Product Line 1)
  - Control Philosophy:
    - Temperature: Maintained at 34°C ± 1°C via PID loop TT201:TIC201-->TC201, cooling water adjusted to counter exothermic reaction heat.
    - pH: Regulated at 5.0 ± 0.2 via PID loop pH202:AIC202-->AV202, dosing sulfuric acid or sodium hydroxide to prevent yeast inhibition.
    - Agitation: Set at 90 RPM ± 10 RPM via SIC203-->SV203, ensuring uniform mixing and oxygen transfer without shear stress.
    - Level: Controlled at 75% via LIC204-->LV204, adjusting media inflow to maintain optimal volume.
    - Dissolved Oxygen: Kept at 30% via AIC206-->AV206, sparging air to support yeast metabolism.
    - Interlocks: Halt FV205 if TAH-201 (>38°C), PALL-202 (<4.5), or LAH-204 (>90%); alarm on FAH-207 (CO2 >50 L/min after 72 hours, stalled fermentation).

4. Filtration
-------------
  - Separator (SP01): Removes biomass and solids from fermentation broth.
    - FT208 (Flow Transmitter, 0–1000 L/h): Monitors broth inflow.
    - FIC208 (Flow Controller): Setpoint 500 L/h, FIC208-->FV208 (broth valve).
    - PT209 (Pressure Transmitter, 0–10 bar): Monitors separator pressure.
    - LT210 (Level Transmitter, 0–100%): Monitors filtrate level.
  - Piping: P1 --> [FIC208:FV208] --> SP01
  - Piping: SP01 --> F1 (Filtrate Line 1)
  - Piping: SP01 --> W1 (Waste Line, biomass)
  - Control: Maintains 500 L/h broth flow via FIC208-->FV208; interlock stops FV208 if PAH-209 (>8 bar).

5. Downstream Recovery
---------------------
  - Product Tank (PT01): Stores filtered penicillin solution for further processing.
    - LT211 (Level Transmitter, 0–100%): Monitors tank level.
    - LIC211 (Level Controller): Setpoint 80%, LIC211-->LV211 (filtrate valve).
    - TT212 (Temperature Transmitter, 0–50°C): Monitors solution temperature.
    - AT213 (Penicillin Analyzer, 0–100 g/L): Monitors penicillin concentration.
  - Piping: F1 --> [LIC211:LV211] --> PT01
  - Control: Maintains tank level at 80% via LIC211-->LV211; monitors AT213 for quality (target 50–80 g/L); interlock stops LV211 if TAH-212 (>40°C).

Notes:
- Process: Converts sugars to penicillin via fermentation, with media preparation, sterilization, filtration, and recovery stages.
- Equipment: WT01 (Water Tank), NT01 (Nutrient Tank), MX01 (Mixer), ST01 (Sterilizer), FM01 (Fermenter), SP01 (Separator), PT01 (Product Tank).
- Instrumentation: Follows ISA-5.1 (e.g., TT for Temperature Transmitter, FIC for Flow Controller).
- Flow: --> indicates material flow (e.g., W1 --> WT01).
- Control Loops: Denoted as transmitter:controller-->actuator (e.g., TT201:TIC201-->TC201).
- Piping: Labeled (e.g., M1, F1, W1) for clarity.
- Ranges/Setpoints: Typical for penicillin fermentation, adjustable per plant.
  - TT201: 0–50°C, setpoint 34°C.
  - pH202: 0–14, setpoint 5.0.
  - ST203: 0–200 RPM, setpoint 90 RPM.
  - LT204: 0–100%, setpoint 75%.
  - AT213: 0–100 g/L, target 50–80 g/L.
- Safety: Interlocks prevent unsafe conditions (e.g., halt feeds on high temperature, low pH, or high level).
- Usage: Guides PLC configuration, HMI setup, and control narrative development.
- Format: Plain-text, readable in code editors, suitable for documentation/collaboration.
- Assumptions:
  - Batch fermentation process with Saccharomyces cerevisiae.
  - Fermenter size ~2000 L, typical for medium-scale production.
  - PLC supports 4–20 mA analog and 24 VDC digital I/O.
  - Signals are standard for penicillin production.
------------------------------------------------------------------------
```
