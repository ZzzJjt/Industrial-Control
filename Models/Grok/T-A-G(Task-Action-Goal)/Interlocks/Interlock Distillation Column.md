Piping and Instrumentation Diagram (P&ID) - Distillation Column System
System Overview
The system consists of a distillation column (C-101) with a reboiler (E-101) and condenser (E-102), processing a feed stream to separate components. Instrumentation and actuators ensure control and safety.
Components

C-101: Distillation Column
Processes feed stream into distillate (top) and bottoms (bottom).
Equipped with trays or packing for vapor-liquid contact.


E-101: Reboiler
Provides heat to C-101 via steam or electric heating.
Controlled to maintain column temperature.


E-102: Condenser
Cools and condenses vapor from C-101 top.
Uses cooling water or refrigerant.



Instrumentation

PT-101: Pressure Transmitter
Location: Top of C-101
Measures column pressure (psi).
Range: 0–200 psi


TT-101: Temperature Transmitter
Location: Bottom of C-101 (near reboiler)
Measures column temperature (°C).
Range: 50–250°C


LT-101: Level Transmitter
Location: Bottom of C-101
Measures liquid level in column sump (%).
Range: 0–100%



Actuators

FV-101: Feed Valve
Location: Feed line to C-101
Type: Pneumatic control valve
Controls feed flow rate.


PRV-101: Pressure Relief Valve
Location: Top of C-101
Type: Spring-loaded relief valve
Opens to vent excess pressure to a safe location.



Piping and Flow

Feed Stream: Enters C-101 via FV-101.
Vapor Stream: Exits C-101 top to E-102 (condenser).
Condensate: Returns from E-102 to C-101 or distillate storage.
Bottoms Stream: Exits C-101 bottom, heated by E-101 (reboiler).
Vent Line: Connected to PRV-101 for pressure relief.

Control Functions

Temperature Control:
TT-101 provides input to a PID controller.
Controller adjusts E-101 heat input to maintain setpoint (e.g., 150°C).


Level Control:
LT-101 provides input to a PID controller.
Controller adjusts bottoms pump speed to maintain sump level (e.g., 50%).


Feed Control:
FV-101 adjusts feed flow based on process demand (manual or cascaded control).



Safety Interlocks

High Pressure (>120 psi):
PT-101 triggers PRV-101 to open, venting excess pressure.


Low Pressure (<50 psi):
PT-101 closes FV-101 to stop feed, preventing vacuum or instability.


High Temperature (>180°C):
TT-101 shuts off E-101 heat supply to prevent thermal runaway.



Text-based P&ID Notation
[Feed Stream] --> [FV-101] --> [C-101 Distillation Column]
                                  |
                                  | (PT-101: Pressure)
                                  | (TT-101: Temperature)
                                  | (LT-101: Level)
                                  |
                                  |--> [PRV-101] --> [Vent]
                                  |
[Vapor] --> [E-102 Condenser] --> [Condensate/Distillate]
                                  |
[Bottoms] --> [E-101 Reboiler] --> [Bottoms Product]

Notes

All instrumentation is connected to the PLC for monitoring and control.
Safety interlocks are implemented in the PLC to ensure rapid response.
PRV-101 vents to a safe location (e.g., flare or scrubber system).
E-101 heat supply is assumed to be electrically controlled for simplicity.

