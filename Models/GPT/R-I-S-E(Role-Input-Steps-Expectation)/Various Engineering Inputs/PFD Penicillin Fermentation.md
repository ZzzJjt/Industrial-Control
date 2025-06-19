[WT01] Water Tank
   --> [FT101] Flow Transmitter
   --> [FIC101] Flow Controller
   --> [FCV101] Control Valve
   --> [NM01] Nutrient Mixer
         |
         +--> [TT102] Temp Transmitter
         --> [TIC102] Temp Controller
         --> [TCV102] Steam Control Valve
         |
         +--> [LT103] Level Transmitter

 [NM01] Nutrient Mixer
   --> [ST01] Sterilizer
         |
         +--> [TT104] Sterilization Temperature
         --> [TIC104] Temperature PID Control
         --> [TCV104] Steam Valve
         |
         +--> [PT105] Pressure Transmitter (monitoring only)

[ST01] Sterilized Media
   --> [FR01] Fermenter
         |
         +--> [TT201] Fermentation Temp
         --> [TIC201] Temp Controller
         --> [TCV201] Cooling Water Valve
         |
         +--> [pH201] pH Probe
         --> [PIC201] pH Controller
         --> [DP201A/B] Base/Acid Dosing Pumps
         |
         +--> [AT201] Agitator Speed Transmitter
         --> [AIC201] Agitation Speed Controller
         --> [VFD201] Variable Frequency Drive
         |
         +--> [LT202] Fermenter Level Transmitter

 [FR01] Fermenter Broth
   --> [SP01] Centrifugal Separator
         |
         +--> [FT301] Feed Flow Transmitter
         --> [FIC301] Flow Controller
         --> [FCV301] Control Valve
         |
         +--> [LT302] Separator Level Transmitter
   --> [PT01] Product Tank
         |
         +--> [TT303] Product Storage Temp
         --> [TIC303] Cooling Control Loop
         --> [TCV303] Jacket Cooling Valve
