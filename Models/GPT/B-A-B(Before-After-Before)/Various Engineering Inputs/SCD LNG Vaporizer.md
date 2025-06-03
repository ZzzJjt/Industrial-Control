────────────────────── NORSOK-Style SCD – LNG Vaporizer System ──────────────────────

Process Flow:  
─────────────────────────────────────────────────────────────────────────────────────

LNG_FEED_LINE (Line L-100)  
   |  
   |--> FT201 (Flow Transmitter, U-610)  
         |  
         |--> FIC201 (Flow Indicating Controller, U-610)  
               |  
               |--> FCV201 (Flow Control Valve, L-120)  
                     |
                     |--> LNG_VAPORIZER (VZ101, Shell-and-Tube Type)  
                           |  
                           |--> TT101 (Temperature Transmitter, U-620)  
                                 |  
                                 |--> TIC101 (Temperature Indicating Controller, U-620)  
                                       |  
                                       |--> TCV101 (Heater Steam Control Valve, L-121)  
                                             |  
                                             |--> LNG_OUTLET_GAS (Line L-130)

─────────────────────────────────────────────────────────────────────────────────────

Safety & Shutdown Interlocks:  
─────────────────────────────────────────────────────────────────────────────────────

• High-High Pressure Protection:
    → PSH301 (Pressure Switch High-High, U-630)  
         → Trip Signal to ESD1 (Emergency Shutdown Logic)

• Emergency Shutdown Logic (ESD1) Triggers:
    - Close FCV201 (Shut off LNG feed)
    - Close TCV101 (Shut off steam supply to heater)
    - Activate Audible/Visual Alarm (ALM101)

─────────────────────────────────────────────────────────────────────────────────────

Legend & Notes:
─────────────────────────────────────────────────────────────────────────────────────
• FT = Flow Transmitter
• FIC = Flow Indicating Controller
• FCV = Flow Control Valve
• TT = Temperature Transmitter
• TIC = Temperature Indicating Controller
• TCV = Temperature Control Valve
• PSH = Pressure Switch High
• ESD = Emergency Shutdown Device
• U-610/U-620/U-630 = Control loop references
• L-120/L-121 = Line and valve references

─────────────────────────────────────────────────────────────────────────────────────
Control Logic Summary:
─────────────────────────────────────────────────────────────────────────────────────
1. Flow Control: FT201 measures LNG flow, FIC201 maintains setpoint via FCV201
2. Temperature Control: TT101 senses outlet temp, TIC101 adjusts steam via TCV101
3. Safety Interlock: If PSH301 detects pressure > limit, ESD1 executes trip sequence
─────────────────────────────────────────────────────────────────────────────────────
