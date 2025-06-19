       [Step: Green Light]
             |
       TON tGreen (PT := T#5s)
             |
       Transition: tGreen.Q
             |
             V
       [Step: Yellow Light]
             |
       TON tYellow (PT := T#2s)
             |
       Transition: tYellow.Q
             |
             V
       [Step: Red Light]
             |
       TON tRed (PT := T#5s)
             |
       Transition: tRed.Q
             |
             V
      +-------------------+
      | Loop back to Green|
      +-------------------+
             |
             +---> [Step: Green Light]
