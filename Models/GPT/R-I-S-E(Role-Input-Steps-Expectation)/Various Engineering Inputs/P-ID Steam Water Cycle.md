[FWP01] Feedwater Pump
   --> [FT101] Feedwater Flow Transmitter
   --> [FIC101] Flow Controller
   --> [FCV101] Feedwater Control Valve
   --> [B01] Boiler Inlet

  [B01] Boiler
   --> [TT102] Steam Temperature Transmitter
   --> [TIC102] Temperature Controller
   --> [TCV102] Superheater Temperature Control Valve
   --> [SD01] Steam Drum
         |
         +--> [LT103] Drum Level Transmitter
         --> [LIC103] Level Controller
         --> [LCV103] Level Control Valve (to maintain drum water level)

         [SD01] Steam Drum
   --> [PT104] Main Steam Pressure Transmitter
   --> [PIC104] Pressure Controller
   --> [PCV104] Main Steam Pressure Control Valve
   --> [T01] Steam Turbine

[T01] Steam Turbine Exhaust
   --> [CT01] Condenser
         |
         +--> [TT105] Condensate Temp Transmitter
         --> [LT106] Condensate Level Transmitter
         --> [LIC106] Condensate Level Controller
         --> [PCV106] Condensate Drain Control Valve
