# Piping and Instrumentation Diagram for Distillation Column System

# Equipment
- C-101: Distillation Column
- E-101: Reboiler, heated by steam
- E-102: Condenser, cooled by water

# Sensors
- PT-101: Pressure Transmitter, C-101 top, 0–150 psi
- TT-101: Temperature Transmitter, C-101 bottom, 0–200°C
- LT-101: Level Transmitter, C-101 bottom, 0–100%

# Actuators
- FV-101: Feed Valve, controls feed to C-101, 0–100% open
- PRV-101: Pressure Relief Valve, C-101 top, open/closed
- HV-101: Heating Valve, steam to E-101, 0–100% open

# Piping
- P-101: Feed Line, raw material --> FV-101 --> C-101
- P-102: Vapor Line, C-101 top --> E-102
- P-103: Condensate Line, E-102 --> reflux/product
- P-104: Bottoms Line, C-101 bottom --> E-101
- P-105: Steam Line, steam source --> HV-101 --> E-101
- P-106: Relief Line, C-101 --> PRV-101 --> flare

# Control Loops
- Pressure:
  - PT-101 --> PRV-101: Open if P > 120 psi
  - PT-101 --> FV-101: Close if P < 50 psi
- Temperature:
  - TT-101 --> HV-101: Close if T > 180°C
- Level:
  - LT-101 --> FV-101: Adjust for level control (normal operation)

# Notes
- All sensors use 4–20mA signals.
- Actuators have feedback signals for position confirmation.
- Interlocks are implemented in PLC via Structured Text.
