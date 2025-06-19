LNG_INLET
  --> FT201        (Flow Transmitter)
      --> FIC201   (Flow Indicator Controller)
          --> FCV201 (Flow Control Valve)
              --> LNG_VAPORIZER
                  |
                  |--> TT101        (Temperature Transmitter)
                      --> TIC101    (Temperature Indicator Controller)
                          --> TCV101 (Steam/Heating Medium Control Valve)
                              |
                              --> VAPORIZED_GAS_OUTLET

FT201 (measures LNG inlet flow)
  --> FIC201 (compares to flow setpoint)
      --> FCV201 (regulates LNG flow into vaporizer)


TT101 (monitors LNG vaporizer outlet temperature)
  --> TIC101 (compares to temperature setpoint)
      --> TCV101 (regulates heating medium to maintain vaporization temperature)
