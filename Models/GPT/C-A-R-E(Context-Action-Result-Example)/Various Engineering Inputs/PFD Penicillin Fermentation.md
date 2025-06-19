[Media Preparation Line]
Water Tank WT101 
    --> Nutrient Mixer MX102 
        --> TT101 → TIC101 → Steam Valve (TC101) [Mix Temp Control]
        --> LT102 (Level Transmitter on mixer)
    --> Holding Tank HT103

[Sterilization Line]
HT103 
    --> Heat Sterilizer ST104 
        --> TT104 → TIC104 → Steam Control Valve (TC104) [Sterilization Temp Control]
        --> FT105 (Flow of sterilized media)
    --> Sterile Media Tank SMT106

[Fermentation Section]
SMT106 
    --> Fermenter V201
        --> TT201 → TIC201 → Jacket Valve (TC201) [Fermenter Temp Control]
        --> pH202 → PIC202 → Base Pump (PC202) [pH Control]
        --> LT203 (Level Transmitter in fermenter)
        --> Agitator AG204 (Speed via VFD; optionally controlled via speed setpoint SP204)
        --> DO205 → AIC205 (Dissolved Oxygen Analysis with Alarm)

[Separation Section]
V201 
    --> Cell Separator SEP301
        --> FT302 (Feed flow measurement)
    --> Filtrate to Product Collection Tank PCT401
        --> LT402 (Level Transmitter on PCT)
    --> Biomass to Waste Holding Tank WHT403

[Product Handling]
PCT401 
    --> Buffer Tank BT501 
        --> TT502 → TIC502 → Cooling Valve (TC502) [Cooling before downstream]
        --> PT503 (Pressure monitoring during transfer)
