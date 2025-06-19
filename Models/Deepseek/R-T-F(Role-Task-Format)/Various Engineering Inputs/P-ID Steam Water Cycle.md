+---------------------+
| Feedwater Tank      |
|                     |
|  LV101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Feedwater Pump      |
|                     |
|  FP101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Makeup Water Valve  |
|                     |
|  FCV101             |
+---------+-----------+
          |
          v
+---------+-----------+
| Feedwater Flow      |
| Transmitter         |
|                     |
|  FT101              |
+---------+-----------+
          |
          v
+---------+-----------+
| Feedwater Control   |
| Valve               |
|                     |
|  FCV102             |
+---------+-----------+
          |
          v
+---------+-----------+
| Boiler                  B01       |
|                               |
|  Drum Level Sensor     LT101    |
|                               |
|  Steam Outlet          SV101    |
|                               |
|  Feedwater Inlet       FI101    |
+---------+-----------+
          |
          v
+---------+-----------+
| Steam Turbine           ST101    |
|                               |
|  Condenser Inlet       CI101    |
+---------+-----------+
          |
          v
+---------+-----------+
| Condenser             CD101    |
|                               |
|  Condensate Outflow    CO101    |
|                               |
|  Cooling Water Inflow  CWI101   |
|                               |
|  Cooling Water Outflow CWO101   |
+---------+-----------+
          |
          v
+---------+-----------+
| Condensate Return     |
| Pump                  CRP101   |
|                               |
|  Condensate Flow       CRT101   |
|                               |
|  Condensate Return     CRV101   |
|  Valve                |
+---------+-----------+
          |
          v
+---------+-----------+
| Deaerator             DA101    |
|                               |
|  Deaerator Level     DLT101   |
|  Sensor               |
|                               |
|  Feedwater Outlet      FO101    |
+---------------------+

Control Loops:
LV101 --> FCV101 --> Feedwater Tank
Feedwater Tank --> FP101 --> Boiler B01
FP101 --> FCV102 --> Boiler B01
Boiler B01 --> FT101 --> FIC101 --> FCV102
Boiler B01 --> LT101 --> LIC101 --> LV101
CD101 --> CRT101 --> CRP101 --> DA101
DA101 --> DLT101 --> DLIC101 --> LV101
