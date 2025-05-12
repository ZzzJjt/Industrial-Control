```
| Causes (C)              | Close Feed Valve (CFV) | Start Emergency Cooling (SEC) | Open Pressure Relief Valve (OPRV) | Stop Heating (SH) | Trigger Alarm (TA) |
|-------------------------|------------------------|-------------------------------|-----------------------------------|-------------------|-------------------|
| C1: High Temperature (> T_max) |                      | X                             |                                    | X                 | X                |
| C2: Low Temperature (< T_min) |                      |                               |                                    | X                 | X                |
| C3: High Pressure (> P_max)  |                      |                               | X                                 |                   | X                |
| C4: Low Pressure (< P_min)   |                      |                               |                                    |                   | X                |
| C5: High Level (> L_max)     | X                     | X                             |                                    |                   | X                |
| C6: Low Level (< L_min)      | X                     |                               |                                    |                   | X                |
| C7: Gas Leak Detected        | X                     |                               | X                                 |                   | X                |
| C8: Power Failure            | X                     |                               |                                    |                   | X                |
| C9: Control System Failure   | X                     |                               |                                    |                   | X                |
```
