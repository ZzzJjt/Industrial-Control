```
| Causes (C)         | Emergency Shutdown (ESD) | Cool Down System Activation (CDS) | Pressure Relief Valve Opening (PRV) | Heating System Deactivation (HSD) | Alarm Activation (AA) |
|---------------------|--------------------------|-----------------------------------|-------------------------------------|-----------------------------------|-----------------------|
| C1: High Temperature (> T_max) | X                        | X                                 |                                      | X                          | X                     |
| C2: Low Temperature (< T_min) |                          |                                   |                                      | X                          | X                     |
| C3: High Pressure (> P_max)   | X                        |                                   | X                                  |                            | X                     |
| C4: Low Pressure (< P_min)    |                          |                                   |                                     |                            | X                     |
| C5: High Level (> L_max)      | X                        | X                                 |                                      |                            | X                     |
| C6: Low Level (< L_min)       |                          |                                   |                                      |                            | X                     |
| C7: Gas Leak Detected         | X                        |                                   | X                                  |                            | X                     |
| C8: Power Failure             | X                        |                                   |                                     |                            | X                     |
| C9: Control System Failure    | X                        |                                   |                                     |                            | X                     |
```
