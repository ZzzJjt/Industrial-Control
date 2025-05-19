# Sequential Function Chart (SFC) for Traffic Light System
# IEC 61131-3 Compliant

       +-----------------+
       | INITIAL_STEP    |
       | Start           |
       +-----------------+
               |
               V
       +-----------------+
       | STEP_1          |
       | Green_Light     |
       |                 |
       | TON tGreen      |
       | PT := T#5s      |
       | OUT: Green_ON   |
       +-----------------+
               |
               | (tGreen.Q)
               V
       +=================+
       | TRANSITION_1     |
       | tGreen.Q = TRUE |
       +=================+
               |
               V
       +-----------------+
       | STEP_2          |
       | Yellow_Light    |
       |                 |
       | TON tYellow     |
       | PT := T#2s      |
       | OUT: Yellow_ON  |
       +-----------------+
               |
               | (tYellow.Q)
               V
       +=================+
       | TRANSITION_2     |
       | tYellow.Q = TRUE|
       +=================+
               |
               V
       +-----------------+
       | STEP_3          |
       | Red_Light       |
       |                 |
       | TON tRed        |
       | PT := T#5s      |
       | OUT: Red_ON     |
       +-----------------+
               |
               | (tRed.Q)
               V
       +=================+
       | TRANSITION_3     |
       | tRed.Q = TRUE   |
       +=================+
               |
               +----------------+
                                |
                                V
                                Loop back to STEP_1
