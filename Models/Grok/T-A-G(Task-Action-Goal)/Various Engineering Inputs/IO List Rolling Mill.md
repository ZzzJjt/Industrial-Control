+-----------------------------------+----------------+---------------+--------------+-----------------+--------------------+------------+---------------+
| Signal Name                       | Input/Output   | Signal Tagnumber | Analog/Digital | Engineering Unit | Range             | Setpoint   | P&ID Reference |
+-----------------------------------+----------------+---------------+--------------+-----------------+--------------------+------------+---------------+
| Roll Speed Feedback Stand 1       | Input          | SI-101        | Analog       | RPM             | 0–1200             | 1000       | RM-PID-001    |
| Roll Speed Feedback Stand 2       | Input          | SI-102        | Analog       | RPM             | 0–1200             | 1000       | RM-PID-001    |
| Hydraulic Pressure Stand 1        | Input          | PI-103        | Analog       | bar             | 0–300              | 250        | RM-PID-002    |
| Hydraulic Pressure Stand 2        | Input          | PI-104        | Analog       | bar             | 0–300              | 250        | RM-PID-002    |
| Motor Current Stand 1             | Input          | II-105        | Analog       | A               | 0–5000             | 4000       | RM-PID-003    |
| Motor Current Stand 2             | Input          | II-106        | Analog       | A               | 0–5000             | 4000       | RM-PID-003    |
| Strip Tension Measurement         | Input          | TI-107        | Analog       | kN              | 0–100              | 80         | RM-PID-004    |
| Strip Thickness Feedback          | Input          | TI-108        | Analog       | mm              | 0–10               | 2.0        | RM-PID-005    |
| Roll Gap Position Stand 1         | Input          | LI-109        | Analog       | mm              | 0–50               | 5.0        | RM-PID-006    |
| Roll Gap Position Stand 2         | Input          | LI-110        | Analog       | mm              | 0–50               | 5.0        | RM-PID-006    |
| Cooling Water Flow Rate           | Input          | FI-111        | Analog       | L/min           | 0–1000             | 800        | RM-PID-007    |
| Cooling Water Temperature         | Input          | TI-112        | Analog       | °C              | 0–100              | 30         | RM-PID-007    |
| Strip Temperature Entry           | Input          | TI-113        | Analog       | °C              | 0–1200             | 900        | RM-PID-008    |
| Strip Temperature Exit            | Input          | TI-114        | Analog       | °C              | 0–1000             | 700        | RM-PID-008    |
| Hydraulic Pump Pressure           | Input          | PI-115        | Analog       | bar             | 0–350              | 300        | RM-PID-009    |
| Emergency Stop Status             | Input          | XI-116        | Digital      | -               | 0/1 (OFF/ON)       | 0          | RM-PID-010    |
| Roll Motor Run Status Stand 1     | Input          | XI-117        | Digital      | -               | 0/1 (OFF/ON)       | 1          | RM-PID-003    |
| Roll Motor Run Status Stand 2     | Input          | XI-118        | Digital      | -               | 0/1 (OFF/ON)       | 1          | RM-PID-003    |
| Strip Presence Sensor             | Input          | XI-119        | Digital      | -               | 0/1 (NO/YES)       | 1          | RM-PID-004    |
| Hydraulic System Fault            | Input          | XI-120        | Digital      | -               | 0/1 (OK/FAULT)     | 0          | RM-PID-002    |
| Roll Speed Setpoint Stand 1       | Output         | SO-201        | Analog       | RPM             | 0–1200             | 1000       | RM-PID-001    |
| Roll Speed Setpoint Stand 2       | Output         | SO-202        | Analog       | RPM             | 0–1200             | 1000       | RM-PID-001    |
| Hydraulic Valve Control Stand 1   | Output         | YO-203        | Analog       | %               | 0–100              | 80         | RM-PID-002    |
| Hydraulic Valve Control Stand 2   | Output         | YO-204        | Analog       | %               | 0–100              | 80         | RM-PID-002    |
| Cooling Valve Control             | Output         | YO-205        | Analog       | %               | 0–100              | 75         | RM-PID-007    |
| Roll Gap Adjustment Stand 1       | Output         | YO-206        | Analog       | mm              | 0–50               | 5.0        | RM-PID-006    |
| Roll Gap Adjustment Stand 2       | Output         | YO-207        | Analog       | mm              | 0–50               | 5.0        | RM-PID-006    |
| Motor Start Command Stand 1       | Output         | YO-208        | Digital      | -               | 0/1 (OFF/ON)       | 1          | RM-PID-003    |
| Motor Start Command Stand 2       | Output         | YO-209        | Digital      | -               | 0/1 (OFF/ON)       | 1          | RM-PID-003    |
| Alarm Output                      | Output         | YO-210        | Digital      | -               | 0/1 (OFF/ON)       | 0          | RM-PID-010    |
+-----------------------------------+----------------+---------------+--------------+-----------------+--------------------+------------+---------------+
