```
# I/O List for Rolling Mill in Steel Production Plant

The following table provides an Input/Output (I/O) list for a rolling mill in a steel production plant. The list includes 30 lines detailing various signals involved in the operation of the rolling mill.

| Signal Name          | Input/Output | Signal Tag Number | Analog/Digital | Engineering Unit | Ranges                      | Setpoint | P&ID Reference |
|----------------------|--------------|-------------------|----------------|------------------|-----------------------------|----------|----------------|
| Mill Drive Motor     | Output       | MDMOTOR_001       | Digital         | On/Off           | Off: 0, On: 1               | N/A      | P01            |
| Motor Current        | Input        | MOTORCURR_001     | Analog          | Amps             | 0 - 2000 A                  | N/A      | P01            |
| Roll Gap Position    | Input        | ROLLGAPPOS_001    | Analog          | mm               | 0 - 500 mm                  | 250 mm   | P02            |
| Roll Gap Setpoint    | Output       | ROLLGAPSP_001     | Analog          | mm               | 0 - 500 mm                  | 250 mm   | P02            |
| Strip Thickness      | Input        | STRIPTHICKNESS_001| Analog          | mm               | 0 - 10 mm                   | N/A      | P03            |
| Strip Thickness SP   | Output       | STRIPTHICKNESS_SP_001| Analog          | mm               | 0 - 10 mm                   | 5 mm     | P03            |
| Lubrication Pump     | Output       | LUBPUMP_001       | Digital         | On/Off           | Off: 0, On: 1               | N/A      | P04            |
| Lubrication Pressure | Input        | LUBPRESSURE_001   | Analog          | Bar              | 0 - 10 Bar                  | N/A      | P04            |
| Cooling Water Flow   | Input        | CWFLOW_001        | Analog          | L/min            | 0 - 1000 L/min              | N/A      | P05            |
| Cooling Water Temp   | Input        | CWTMP_001         | Analog          | °C               | 0 - 50 °C                   | N/A      | P05            |
| Strip Tension        | Input        | STRIPTENSION_001  | Analog          | kN               | 0 - 100 kN                  | N/A      | P06            |
| Strip Tension Setpoint| Output      | STRIPTENSION_SP_001| Analog          | kN               | 0 - 100 kN                  | 50 kN    | P06            |
| Strip Speed          | Input        | STRIPSPEED_001    | Analog          | m/min            | 0 - 1000 m/min              | N/A      | P07            |
| Strip Speed Setpoint | Output       | STRIPSPEED_SP_001 | Analog          | m/min            | 0 - 1000 m/min              | 500 m/min| P07            |
| Guide Roll Position  | Input        | GUIDEPOS_001      | Analog          | mm               | 0 - 200 mm                  | N/A      | P08            |
| Guide Roll Setpoint  | Output       | GUIDESP_001       | Analog          | mm               | 0 - 200 mm                  | 100 mm   | P08            |
| Roll Bite Temp       | Input        | ROLLBITETMP_001   | Analog          | °C               | 0 - 300 °C                  | N/A      | P09            |
| Roll Bite Temp Setpoint| Output     | ROLLBITETMP_SP_001| Analog          | °C               | 0 - 300 °C                  | 150 °C   | P09            |
| Roll Bearing Temp    | Input        | ROLLBRTMP_001     | Analog          | °C               | 0 - 100 °C                  | N/A      | P10            |
| Hydraulic Pressure   | Input        | HYDPRESSURE_001   | Analog          | Bar              | 0 - 200 Bar                 | N/A      | P11            |
| Hydraulic Pump       | Output       | HYDPUMP_001       | Digital         | On/Off           | Off: 0, On: 1               | N/A      | P11            |
| Roll Feed Rate       | Input        | ROLLFEEDRATE_001  | Analog          | mm/s             | 0 - 100 mm/s                | N/A      | P12            |
| Roll Feed Rate Setpoint| Output    | ROLLFEEDRATE_SP_001| Analog          | mm/s             | 0 - 100 mm/s                | 50 mm/s  | P12            |
| Pinch Roll Pressure  | Input        | PINCHROLLPRESS_001| Analog          | Bar              | 0 - 50 Bar                  | N/A      | P13            |
| Pinch Roll Pressure SP| Output    | PINCHROLLPRESS_SP_001| Analog          | Bar              | 0 - 50 Bar                  | 25 Bar   | P13            |
| Roll Gap Force       | Input        | ROLLGAPFORCE_001  | Analog          | kN               | 0 - 5000 kN                 | N/A      | P14            |
| Roll Gap Force Setpoint| Output    | ROLLGAPFORCE_SP_001| Analog          | kN               | 0 - 5000 kN                 | 2500 kN  | P14            |
| Strip Edge Temp      | Input        | STRIPEDEGETMP_001 | Analog          | °C               | 0 - 300 °C                  | N/A      | P15            |
| Strip Edge Temp Setpoint| Output    | STRIPEDEGETMP_SP_001| Analog          | °C               | 0 - 300 °C                  | 150 °C   | P15            |
| Strip Surface Temp   | Input        | STRIPSURFTMP_001  | Analog          | °C               | 0 - 300 °C                  | N/A      | P16            |
| Strip Surface Temp Setpoint| Output| STRIPSURFTMP_SP_001| Analog          | °C               | 0 - 300 °C                  | 150 °C   | P16            |

This I/O list covers a wide array of signals essential for the operation of a rolling mill in a steel production plant, including motor control, position sensing, temperature measurement, hydraulic systems, and tension control. The setpoints provided are typical values for a given operation and can be adjusted according to specific process requirements.
```
