3.1 Temperature Control

The primary objective of temperature control during the fermentation phase is to maintain optimal conditions for yeast activity while preventing overheating that could lead to yeast stress or death. The target temperature range is set between 32°C and 35°C. A PID controller (TIC-301) regulates the temperature using cooling jackets and heating elements. Temperature sensors (TT-301) continuously monitor the fermenter temperature. If the temperature exceeds 36°C, an alarm is triggered, and if it reaches 38°C, the emergency shutdown valve (ESD-301) is activated to isolate the fermenter and stop heating. Conversely, if the temperature drops below 30°C, additional heating is initiated to bring it back within the acceptable range.

3.2 pH Regulation

Maintaining the correct pH level is crucial for optimizing yeast growth and enzyme activity. The target pH range is set between 4.5 and 5.0. A pH analyzer (AIC-302) measures the pH of the fermenter broth, and a PID controller (AIC-302) adjusts the addition of acid or base through dosing pumps (DP-301, DP-302). If the pH falls below 4.0 or rises above 5.5, alarms are triggered. At more severe deviations (below 3.8 or above 5.8), the fermentation process is automatically shut down via ESD-302 to prevent irreparable damage to the yeast.

3.3 Inoculation

Proper inoculation with yeast is essential for initiating and maintaining efficient fermentation. The inoculation process involves adding a measured amount of yeast slurry to the fermenter at the beginning of the batch. A flow meter (FIC-303) monitors the yeast slurry flow rate, ensuring it meets the specified inoculation rate. Once the inoculation is complete, the flow is stopped, and the fermenter is sealed. An interlock ensures that the fermentation process does not start until the inoculation is verified by the operator. If the inoculation flow deviates significantly from the setpoint, an alarm is triggered, and the process is halted until corrected.

3.4 Dissolved Oxygen (DO) Control

Optimal dissolved oxygen levels are critical for aerobic yeast metabolism. The target DO level is maintained between 2% and 4%. An oxygen sensor (O2-301) measures the DO concentration in the fermenter broth. A PID controller (O2C-301) controls the agitation speed (using variable frequency drives, VFD-301) to maintain the desired DO level. If the DO level drops below 1%, an alarm is triggered, and the agitation speed is increased immediately. If the DO level remains low despite adjustments, the process is shut down via ESD-303 to prevent anaerobic fermentation and potential product spoilage.

3.5 Agitation Control

Agitation is vital for mixing the fermenter contents, ensuring uniform distribution of nutrients and gases. The target agitation speed is set between 500 and 700 RPM. A tachometer (AG-301) measures the actual agitation speed. A PID controller (AGC-301) adjusts the motor speed using a variable frequency drive (VFD-302) to maintain the setpoint. If the agitation speed falls below 400 RPM or exceeds 800 RPM, alarms are triggered. Severe deviations result in shutting down the agitator via ESD-304 to prevent incomplete mixing or mechanical failure.

3.6 Pressure Control

Maintaining proper pressure in the fermenter is important to prevent contamination and ensure safe operation. The target pressure is set at atmospheric pressure (0 bar gauge). A pressure transmitter (PT-301) monitors the fermenter pressure. A PID controller (PIC-302) controls the vent valve (VV-301) to maintain the setpoint. If the pressure exceeds 0.2 bar, an alarm is triggered, and the vent valve opens to relieve excess pressure. If the pressure continues to rise beyond 0.5 bar, the fermenter is isolated via ESD-305 to prevent overpressure hazards.

3.7 Nutrient Addition

Adding nutrients like nitrogen and phosphorus is necessary for supporting yeast growth. The nutrient addition process is controlled by flow meters (FIC-304, FIC-305) that measure the flow rates of nitrogen and phosphorus solutions. PID controllers (NFC-301, NFC-302) regulate the nutrient dosing pumps (DP-303, DP-304) to maintain the specified nutrient concentrations. If the nutrient flow deviates significantly from the setpoints, alarms are triggered, and the process is paused until corrections are made.
