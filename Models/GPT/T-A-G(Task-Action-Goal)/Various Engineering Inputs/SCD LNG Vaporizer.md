LNG Storage Tank
   |
   |---> FT101 ------> FCV101 ------> LNG Inlet Line
                              |
                              ▼
                           Vaporizer (Heated Water Bath)
                              |
         TT101 -----> TIC101 -----> TCV101 (Steam to Bath)
                              |
                              ▼
                           Outlet Gas Line
                              |
         FT201 -----> FIC201 -----> FCV201 (Gas Flow Control)
                              |
                              ▼
                          Export Pipeline


PSH301 (High Pressure Switch) → ESD1 → Close TCV101 (Steam Inlet Valve)
Action: Shut steam to vaporizer to reduce heat input.
Trip Limit: > 6.0 bar(g)

TSH302 (High Temp Switch) → ESD2 → Close FCV101 (LNG Inlet Valve)
Action: Block LNG flow into vaporizer to prevent thermal runaway.
Trip Limit: > 75 °C

LSL303 (Low Level Switch) → Alarm & Block TCV101
Action: Prevent steam heating without sufficient water medium.
