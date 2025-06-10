Water Tank (WT-101)
   |
   +--> FT101 --> FIC101 --> CV101 (Flow Control Valve)
   |
Nutrient Mixer (MX-102)
   |
   +--> pH102 --> pHIC102 --> CV102 (Acid/Base Dosing Valve)
   +--> TT103 --> TIC103 --> CV103 (Heating/Cooling Valve)
   |
Sterilizer (ST-103)
   |
   +--> TT104 --> TIC104 --> TCV104 (Steam Control Valve)
   +--> PT105 (Pressure Transmitter)
   |
Fermenter (FR-104)
   |
   +--> pH106 --> pHIC106 --> CV106 (Acid/Base Dosing Valve)
   +--> TT107 --> TIC107 --> TCV107 (Cooling Water Valve)
   +--> AT108 --> AIC108 --> M108 (Agitator Motor Speed Controller)
   +--> FT109 (Air Flow Measurement)
   |
Separator (SP-105)
   |
   +--> LT110 --> LIC110 --> CV110 (Level Control Valve)
   +--> TT111 (Outlet Temperature Measurement)
   |
Product Tank (PT-106)
   |
   +--> LT112 --> LIC112 --> CV112 (Discharge Valve)

Control Philosophy:
- Temperature in Sterilizer and Fermenter is precisely regulated by PID loops controlling steam (TCV104) and cooling water (TCV107).
- pH is maintained through automated acid/base dosing valves (CV102, CV106), driven by continuous PID controllers monitoring real-time pH measurements.
- Agitation speed in the fermenter (M108) is modulated based on dissolved oxygen feedback to maintain optimal microbial activity.
- Level control loops ensure stable processing conditions and efficient product separation in Separator and Product Tank.
