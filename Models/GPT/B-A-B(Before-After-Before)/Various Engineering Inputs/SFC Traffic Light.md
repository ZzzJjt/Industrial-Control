Sequential Function Chart – Traffic Light (IEC 61131-3 ASCII Format)

Start
  |
  v
+---------------+
|   S1: GREEN   |
| TON: tGreen   |
| PT := T#5s    |
+---------------+
        |
        | tGreen.Q (after 5s)
        v
+---------------+
|  S2: YELLOW   |
| TON: tYellow  |
| PT := T#2s    |
+---------------+
        |
        | tYellow.Q (after 2s)
        v
+---------------+
|   S3: RED     |
| TON: tRed     |
| PT := T#5s    |
+---------------+
        |
        | tRed.Q (after 5s)
        |
        +------------------+
                           |
                           v
                      +---------+
                      |  Loop   |
                      |   to    |
                      |  GREEN  |
                      +---------+
                           |
                           v
                     (Back to S1)
