          Analog Input
          +-----------+
          |           |
          |    AI     |
          |           |
          |   OUT-----+----+
          +-----------+    |
                           |
                           | 
                           v
                +-----------------+
                |                 |
                |      PID        |
          +-----| PV              |
          |     |                 |
          |     | SP (Setpoint)   |
          |     |                 |
          |     |       CV--------+---->
          |     +-----------------+    |
          |                            |
          |                            |
          |                            v
          |                    Analog Output
          |                    +-----------+
          |                    |           |
          |                    |    AO     |
          |                    |           |
          |                    +-----------+
          |
          v
   +----------------+
   |                |
   |     TON        |
   |                |
   |  IN            |
   |  PT (Preset)   |
   |                |
   |      ET        |
   +----------------+
