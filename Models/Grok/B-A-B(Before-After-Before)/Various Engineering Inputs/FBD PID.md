```
IEC 61131-3 Function Block Diagram (FBD) for Ethanol Fermentation Control
------------------------------------------------------------------------
Description: Visualizes the interaction between Analog Input, TON Timer,
PID Controller, and Analog Output blocks for a fermentation temperature
control loop. The Analog Input (AI_Value) feeds the Timer (triggering a
check) and PID (process variable), with the PID output (CV) driving the
Analog Output (coolant valve position).

Components:
- Analog Input: Reads fermenter temperature (AI_Value).
- TON Timer: Triggers periodic checks (IN, ET, Q).
- PID Controller: Regulates temperature (PV, SP, CV).
- Analog Output: Sets coolant valve position (IN).

Signal Flow:
- AI_Value -> TON.IN (triggers timer if temperature exceeds threshold)
- AI_Value -> PID.PV (process variable for temperature control)
- PID.CV -> Analog Output.IN (control valve position)

ASCII Diagram:
+------------+          +------------+          +------------+
| AnalogInput|          | TON Timer  |          | PID        |
|            |          |            |          |            |
| OUT:AI_Value|----+----|IN         |          |PV         |
|            |    |     |ET         |          |SP:34.0    |
|            |    |     |Q          |          |CV         |
+------------+    |     +------------+          +------------+
                  |                            |
                  +----------------------------|-------> +------------+
                                               |        | AnalogOutput|
                                               |        |            |
                                               |        |IN         |
                                               +--------|            |
                                                        +------------+

Notes:
- AI_Value: Fermenter temperature (°C) from thermocouple (e.g., TT-301).
- TON.IN: Triggered if AI_Value > 35°C (e.g., to log high temperature).
- TON.ET: Elapsed time since IN = TRUE.
- TON.Q: TRUE when timer completes (e.g., T#5m for alarm).
- PID.PV: Process variable (AI_Value, temperature in °C).
- PID.SP: Setpoint (34.0°C, typical for fermentation).
- PID.CV: Control variable (0–100%, valve position).
- Analog Output.IN: Valve position for coolant flow (FV-301).
- Connections use ----> for signal flow, with + for splits.
- Diagram is compact, readable in code editors, and 61131-3-compliant.
- Assumes single-cycle execution; multi-cycle logic can be added for timers.
------------------------------------------------------------------------
```
