# SFC: Traffic Light System - IEC 61131-3
# Notation: +-----+ (step), ===== (transition), | (sequence), --> (flow), <-- (loop back)
# Steps: Green (5s), Yellow (2s), Red (5s)
# Transitions: Timer outputs (tGreen.Q, tYellow.Q, tRed.Q)
# Loop: Red --> Green

       +-------+
       | INIT  |
       +-------+
           |
           | Start
       =====
           |
           v
       +-------+
       | Green |
       |       | 
       | GreenLight := TRUE
       | tGreen.IN := TRUE
       | +-----------------+
       | | tGreen (TON)    |
       | | IN: TRUE        |
       | | PT: T#5s        |
       | | Q: tGreen.Q     |
       | | ET: Elapsed Time|
       | +-----------------+
       +-------+
           |
           | tGreen.Q
       =====
           |
           v
       +-------+
       | Yellow|
       |       | 
       | YellowLight := TRUE
       | tYellow.IN := TRUE
       | +-----------------+
       | | tYellow (TON)   |
       | | IN: TRUE        |
       | | PT: T#2s        |
       | | Q: tYellow.Q    |
       | | ET: Elapsed Time|
       | +-----------------+
       +-------+
           |
           | tYellow.Q
       =====
           |
           v
       +-------+
       | Red   |
       |       | 
       | RedLight := TRUE
       | tRed.IN := TRUE
       | +-----------------+
       | | tRed (TON)      |
       | | IN: TRUE        |
       | | PT: T#5s        |
       | | Q: tRed.Q       |
       | | ET: Elapsed Time|
       | +-----------------+
       +-------+
           |
           | tRed.Q
       =====
           |
           +----------------+
                            |
                            v
                          Green (Loop Back)
                          <--
