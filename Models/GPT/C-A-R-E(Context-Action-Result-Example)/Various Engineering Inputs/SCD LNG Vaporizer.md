[LNG Process Flow]
LNG Inlet 
  --> FT201                           // Flow Transmitter
  --> FIC201                          // Flow Indicator Controller
  --> FCV201                          // Flow Control Valve (controls LNG feed rate)
  --> Vaporizer Unit VAP01
  --> TT101 → TIC101 → TCV101        // Temp Control Loop for vaporizer outlet
  --> Gas Outlet

[Vaporizer Temperature Control Loop]
TT101 (Gas outlet temp sensor) 
  --> TIC101 (Temp PID Controller) 
  --> TCV101 (Heater steam control valve) 

[Feed Gas Pressure Monitoring]
PT102 (Outlet pressure) 
  --> PI102 (Pressure Indicator) 
  --> PSH102 --> ESD1                // Emergency Shutdown on High Pressure

[Heater Safety Interlocks]
TSH103 (Steam jacket high temp) 
  --> ESD2                           // Trip: Shut down heater on overtemp
TSL104 (Steam low temp) 
  --> Alarm A102                     // Warn: Steam temperature too low

[Flow Trip Logic]
FT201 (Flow < 5 kg/h)
  --> Alarm A101                     // Low flow warning
  --> FIC201 → FCV201 → Auto Close   // Trip valve on insufficient flow

[Emergency Stop Integration]
ESD Pushbutton PB105 
  --> ESD System → Trip FCV201, TCV101, and close LNG Inlet Block Valve

[Startup Sequence Dependencies]
- TIC101 must be within range (±5°C) before allowing FCV201 to open
- ESD1 and ESD2 must be clear
- Manual Reset required after any ESD trip
