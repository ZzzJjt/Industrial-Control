[Feedwater System]
T01  -->  FT101  -->  FIC101  -->  FCV101  -->  P01  -->  Boiler B01
|        |         |            |             |
|        |         |            |             +--> PT102
|        |         |            +--> Flow Control Valve (FCV101)
|        |         +--> Flow Indicator Controller (FIC101)
|        +--> Flow Transmitter (FT101)
+--> Feedwater Tank

[Steam Generation]
Boiler B01  -->  Steam Drum D01
|              |--> LT103  -->  LIC103  -->  LV103
|              |     |         |           +--> Level Control Valve
|              |     |         +--> Level Indicator Controller
|              |     +--> Level Transmitter
|              +--> PT104  (Drum Pressure Transmitter)

[Steam to Turbine]
Steam Drum D01  -->  Steam Header H01  -->  Turbine T02
|                                    |
|                                    +--> PT105 (Steam Pressure Transmitter)

[Condensate Recovery]
Turbine T02  -->  Condenser C01
C01  -->  CT101 (Cooling Tower Return) --> Pump P03 --> T01
|                                      |
|                                      +--> TT106 (Condenser Temp Transmitter)
|--> LT107  -->  LIC107  -->  LV107
     |         |             +--> Level Control Valve
     |         +--> Level Controller
     +--> Level Transmitter

[Chemical Dosing System]
T01 --> Dosing Pump DP01 --> pH Analyzer AE101 --> Boiler B01
                           |
                           +--> AI108 (pH Transmitter)
