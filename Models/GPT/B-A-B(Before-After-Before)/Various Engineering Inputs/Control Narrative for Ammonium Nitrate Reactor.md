📘 Control Narrative: Ammonium Nitrate Reactor Stage  
Process: Continuous neutralization of nitric acid (HNO₃) with ammonia (NH₃) in a CSTR  
Objective: Maintain safe, stable, and efficient reaction to produce ammonium nitrate solution for downstream prilling or evaporation

──────────────────────────────────────────────
1. Overview  
──────────────────────────────────────────────  
This reactor operates as a Continuous Stirred Tank Reactor (CSTR), where gaseous or liquid ammonia is introduced to nitric acid. The neutralization reaction is exothermic, requiring tight control over temperature, pressure, and stoichiometry to ensure safety and product quality.

──────────────────────────────────────────────  
2. Critical Control Setpoints  
──────────────────────────────────────────────  
| Parameter                  | Setpoint / Range         | Controller Tag |
|---------------------------|--------------------------|----------------|
| Reaction Temperature      | 175 °C ± 2 °C            | TIC-101        |
| Reactor Pressure          | 4.8 bar ± 0.2 bar        | PIC-102        |
| NH₃:HNO₃ Flow Ratio       | 1.01:1 (molar)           | FIC-103 & FIC-104 |
| pH of Reactor Effluent    | 6.0–6.5                  | pHIC-105       |
| Reactor Liquid Level      | 60–80%                   | LIC-106        |

──────────────────────────────────────────────  
3. Instrumentation  
──────────────────────────────────────────────  
- **Temperature Sensor (TT-101):** Monitors reactor temperature  
- **Pressure Transmitter (PT-102):** Measures reactor pressure  
- **Flow Meters (FT-103 / FT-104):** Measure ammonia and nitric acid feed rates  
- **pH Analyzer (pH-105):** Measures effluent pH  
- **Level Transmitter (LT-106):** Monitors reactor liquid level  
- **High-Level Switch (LSH-107):** Triggers shutdown if reactor overfills  
- **Low-Level Switch (LSL-108):** Triggers shutdown to prevent dry running  

──────────────────────────────────────────────  
4. Control Logic  
──────────────────────────────────────────────  
**Startup Sequence**  
1. Confirm interlocks are clear and reactor level is within safe range (60–80%).  
2. Initiate acid feed via FIC-104.  
3. Ramp up ammonia flow via FIC-103 to reach target 1.01:1 molar ratio.  
4. Activate agitator and reactor jacket heating to maintain 175 °C.  
5. Monitor pressure via PIC-102 and maintain below 5.0 bar using relief or vent control.  
6. Begin pH control loop using minor ammonia flow trimming to maintain pH in 6.0–6.5 range.

**Normal Operation**  
- TIC-101 maintains reactor temperature using jacket steam and cooling water modulation.  
- PIC-102 regulates backpressure via vent or control valve.  
- Flow controllers FIC-103 (NH₃) and FIC-104 (HNO₃) adjust to maintain stoichiometric ratio.  
- pHIC-105 adjusts fine ammonia flow if pH deviates.  
- LIC-106 controls feed or drain rates to maintain liquid level.

**Shutdown Conditions (Auto Interlocks)**  
- TT-101 > 185 °C → Emergency coolant injection + feed shutdown  
- PT-102 > 5.2 bar → Feed cutoff + vent valve open  
- pH-105 outside 5.0–7.0 for >30s → Feed cutoff  
- LSH-107 or LSL-108 triggered → Emergency stop and alarm  
- Manual ESD input → Total feed and agitation shutdown  

──────────────────────────────────────────────  
5. Remarks  
──────────────────────────────────────────────  
- All critical interlocks are hardwired to local PLC for fail-safe action.  
- HMI displays current readings, trends, alarms, and interlock status.  
- Data is logged for process traceability and quality assurance.

Let me know if you need the corresponding **IEC 61131-3 control code**, or a **P&ID annotation** based on this narrative.
