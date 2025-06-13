+---------------------+
| LNG Inlet           |
|                     |
|  FT201: Flow Transmitter     |
+---------+-----------+
          |
          v
+---------+-----------+
| FIC201: Flow Controller      |
+---------+-----------+
          |
          v
+---------+-----------+
| FCV201: Flow Control Valve   |
+---------+-----------+
          |
          v
+---------+-----------+
| Vaporizer             VZ101    |
|                     |
|  TT101: Temperature Transmitter |
|  PT201: Pressure Transmitter  |
+---------+-----------+
          |
          v
+---------+-----------+
| PGV201: Pressure Regulating Valve |
+---------+-----------+
          |
          v
+---------+-----------+
| Gas Outlet            |
|                     |
|  FG301: Flow Transmitter     |
+---------------------+

[Flow Control Loop]
FT201 --> FIC201 --> FCV201 --> Vaporizer VZ101

[Temperature Control Loop]
TT101 --> TIC101 --> TCV101 --> Heat Exchanger (if needed)

[Pressure Control Loop]
PT201 --> PIC201 --> PGV201 --> Gas Outlet

[Safety Interlocks]

High-Pressure Trip:
PT201 > Setpoint (e.g., 15 bar) --> ESD Logic --> Shutdown FCV201, PGV201

Low-Temperature Trip:
TT101 < Setpoint (e.g., -160°C) --> ESD Logic --> Shutdown FCV201, PGV201

High-Flow Trip:
FG301 > Setpoint (e.g., 100 m³/hr) --> ESD Logic --> Shutdown FCV201, PGV201

Emergency Stop Button:
ESD_Button_Pressed --> ESD Logic --> Shutdown FCV201, PGV201

Power Failure Interlock:
Power_Failure_Detected --> ESD Logic --> Shutdown FCV201, PGV201

Overheat Protection:
Heat_Exchanger_Overheat_Detected --> ESD Logic --> Shutdown FCV201, PGV201

Undercooling Protection:
Heat_Exchanger_Undercooling_Detected --> ESD Logic --> Shutdown FCV201, PGV201
