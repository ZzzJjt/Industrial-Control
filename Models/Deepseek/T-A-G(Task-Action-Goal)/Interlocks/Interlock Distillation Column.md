+---------------------+
|      E-101          |
|    (Reboiler)       |
|                     |
|   +----------+      |
|   | HH         |      |
|   | Heat       |      |
|   | Supply     |      |
|   +----------+      |
|        ^            |
|        |            |
|        v            |
| +------+---------+  |
| |                    |  |
| |     C-101        |  |
| | (Distillation    |  |
| |  Column)         |  |
| |                    |  |
| +------+---------+  |
|        ^            |
|        |            |
|        v            |
| +------+---------+  |
| |                    |  |
| |     E-102        |  |
| |    (Condenser)   |  |
| |                    |  |
| +------+---------+  |
|        ^            |
|        |            |
|        v            |
|   +----------+      |
|   | LL         |      |
|   | Coolant    |      |
|   | Supply     |      |
|   +----------+      |
+---------------------+

Instruments and Actuators:
- PT-101: Pressure Transmitter (Top of C-101)
- TT-101: Temperature Transmitter (Inside E-101)
- LT-101: Level Transmitter (Inside C-101)
- FV-101: Feed Valve (Controls feed into C-101)
- PRV-101: Pressure Relief Valve (Relieves excess pressure from C-101)

Interlocks:
- If PT-101 > 120 psi, activate PRV-101
- If PT-101 < 50 psi, close FV-101
- If TT-101 > 180°C, shut off heat supply to E-101
