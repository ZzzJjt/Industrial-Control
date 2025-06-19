──────────────────────────── Steam-Water Cycle – Text-Based P&ID ────────────────────────────

[Condensate Tank] (TK01)
   |
   |---> [Condensate Pump] (P01)
   |        |
   |        |--> FT101 (Condensate Flow Transmitter)
   |                  |
   |                  --> FIC101 (Condensate Flow Controller)
   |                          |
   |                          --> FCV101 (Condensate Flow Control Valve)
   |
   |--> [Low-Pressure Economizer] (ECO01)
               |
               --> TT102 (ECO Outlet Temp Transmitter)
               |
               --> To Feedwater Header

[Feedwater Header]
   |
   |---> [Feedwater Pump] (P02)
   |        |
   |        |--> FT201 (Feedwater Flow Transmitter)
   |                  |
   |                  --> FIC201
   |                          |
   |                          --> FCV201
   |
   |--> [High-Pressure Economizer] (ECO02)
               |
               --> TT202 (ECO Outlet Temp Transmitter)
               |
               --> [Boiler Drum] (D01)
                         |
                         --> LT301 (Drum Level Transmitter)
                         |        |
                         |        --> LIC301 --> LV301
                         |
                         --> PT302 (Drum Pressure Transmitter)

[Drum Steam Outlet] --> [Superheater] (SH01)
                           |
                           --> TT401 (Superheater Outlet Temp)
                           |
                           --> [Turbine Inlet]

[Turbine Exhaust Steam] --> [Condenser] (C01)
                                |
                                --> PT501 (Condenser Pressure)
                                |
                                --> [Condensate Tank] (TK01) ←─── Loop closed

─────────────────────────────────────────────────────────────────────────────────────────────
Legend:
FT  = Flow Transmitter       FCV = Flow Control Valve
FIC = Flow Controller        TT  = Temperature Transmitter
LT  = Level Transmitter      LIC = Level Controller
LV  = Level Control Valve    PT  = Pressure Transmitter
