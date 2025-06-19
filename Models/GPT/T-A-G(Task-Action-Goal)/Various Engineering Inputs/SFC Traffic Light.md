          +---------+
          |  Step 1 |
          | GREEN   |
          +----+----+
               |
               | tGreen.Q (5s elapsed)
               v
          +---------+
          |  Step 2 |
          | YELLOW  |
          +----+----+
               |
               | tYellow.Q (2s elapsed)
               v
          +---------+
          |  Step 3 |
          |   RED   |
          +----+----+
               |
               | tRed.Q (5s elapsed)
               v
           Loop to
          Step 1


    VAR
    tGreen  : TON;   // 5s timer for GREEN
    tYellow : TON;   // 2s timer for YELLOW
    tRed    : TON;   // 5s timer for RED
END_VAR


// Transition from GREEN to YELLOW
Transition_1 := tGreen.Q;

// Transition from YELLOW to RED
Transition_2 := tYellow.Q;

// Transition from RED to GREEN
Transition_3 := tRed.Q;
