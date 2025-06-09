        +-----------+
        |  Step S1  |       Green Light ON
        |  GREEN    |
        +-----------+
              |
              | tGreen.Q = TRUE (after 5s)
              v
        +-----------+
        |  Step S2  |       Yellow Light ON
        |  YELLOW   |
        +-----------+
              |
              | tYellow.Q = TRUE (after 2s)
              v
        +-----------+
        |  Step S3  |       Red Light ON
        |   RED     |
        +-----------+
              |
              | tRed.Q = TRUE (after 5s)
              v
        +-----------+
        |  Step S1  |       Loop back to GREEN
        |  GREEN    |
        +-----------+
