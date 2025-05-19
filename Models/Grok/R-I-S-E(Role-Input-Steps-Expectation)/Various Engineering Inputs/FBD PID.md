+-----------------+
         |   AnalogInput   |
         | OUT: AI (REAL)  |
         +-----------------+
                   |
                   | AI
                   |
         +---------|---------+
         |         |         |
         v         v         |
+--------------------+  +-------------------+
|       TON         |  |       PID         |
| IN: AI (BOOL)     |  | PV: AI (REAL)     |
| PT: T#5s (TIME)   |  | SP: 50.0 (REAL)   |
| ET: ET (TIME)     |  | CV: CV (REAL)     |
| Q: Q (BOOL)       |  +-------------------+
+--------------------+             |
                                  | CV
                                  v
                           +-----------------+
                           |   AnalogOutput  |
                           | IN: CV (REAL)   |
                           +-----------------+

Signal Flow:
- AnalogInput.OUT (AI) --> TON.IN
- AnalogInput.OUT (AI) --> PID.PV
- PID.CV --> AnalogOutput.IN
Notes:
- TON triggers Q after AI is non-zero for 5 seconds (PT = T#5s).
- PID controls CV based on PV (AI) vs. SP (50.0).
- All blocks execute in a single scan cycle (<1 ms).
