1. Water Tank
   |
   |-- FT101 --> FIC101 --> Pump101  (Water flow control)
   |
   ▼

2. Nutrient Mixer
   |
   |-- LT102 --> LIC102             (Level control in mixer)
   |
   ▼

3. Sterilizer (Steam Heating Unit)
   |
   |-- TT201 --> TIC201 --> TCV201  (Temperature control loop)
   |-- PT202                        (Steam pressure monitoring)
   |
   ▼

4. Fermenter (Bioreactor)
   |
   |-- TT203 --> TIC203 --> JacketSteamValve   (Maintain 27–30°C)
   |-- pH201 --> PIC201 --> BasePump           (pH control: target 6.5)
   |-- DO204 --> AIC204 --> AirFlowValve       (Dissolved oxygen control)
   |-- LT205 --> LIC205                        (Level control for broth)
   |
   ▼

5. Separator (Centrifuge)
   |
   |-- FT301 --> FIC301 --> Motor301           (Flow rate control through separator)
   |
   ▼

6. Product Tank
   |
   |-- TT401                                  (Final product temperature monitoring)
   |-- LT402 --> HHL402 / LLL402              (High/Low level alarms)
   |
   ▼

7. Drain/Waste Stream
   |
   |-- PT501 --> PSH501                       (Overpressure interlock)
   |-- TCV502                                 (Temperature-controlled drain)
