                +-----------------+
                |  Analog Input   |
                |     Block       |
                |   OUT = AI      |
                +--------+--------+
                         |
                         v
              +----------+----------+
              |     TON Timer       |
              |     IN = AI         |
              |     Q = Q_TON       |
              |     ET = ET_TON     |
              +----------+----------+

                         |
                         v
                    (Parallel)
                         |
                         v

              +----------+----------+
              |      PID Block      |
              |     PV = AI         |
              |     SP = 50.0       |
              |     CV = CV_PID     |
              +----------+----------+
                         |
                         v
              +----------+----------+
              |   Analog Output     |
              |     IN = CV_PID     |
              +---------------------+
