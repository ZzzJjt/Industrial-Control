LNG Inlet
   |
   +--> FT101 --> FIC101 --> FCV101 --> LNG Vaporizer (VZ-101)

LNG Vaporizer (VZ-101)
   |
   +--> TT102 --> TIC102 --> TCV102 (Heating Medium Valve)
   +--> PT103 --> PIC103 --> PCV103 (Pressure Control Valve)
   |
Gas Outlet
   |
   +--> FT104 --> FIC104 --> FCV104 --> Distribution System

[Temperature Control Loop]
Measurement: TT102
Controller: TIC102
Control Element: TCV102 (Heating Medium Valve)

[Flow Control Loop]
Measurement: FT101
Controller: FIC101
Control Element: FCV101 (LNG Inlet Valve)

Measurement: FT104
Controller: FIC104
Control Element: FCV104 (Outlet Gas Valve)

[Pressure Control Loop]
Measurement: PT103
Controller: PIC103
Control Element: PCV103 (Pressure Control Valve)

[Safety Interlocks]
- High Pressure (>15 bar): Initiates automatic closure of LNG inlet valve (FCV101)
- High Temperature (>10°C above setpoint): Activates emergency shutdown of heating medium valve (TCV102)
- Low Flow (<20% nominal flow): Alarm and potential shutdown of vaporizer (VZ-101)
