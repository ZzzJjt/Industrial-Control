```
# I/O List for Feedwater Control in a Steam-Water Cycle

The following table provides an Input/Output (I/O) list for feedwater control in a steam-water cycle in a power plant.

| Name               | Signal Tag Number | Analog/Digital | Engineering Unit | Ranges                       | Setpoint | P&ID Reference |
|--------------------|-------------------|----------------|------------------|------------------------------|----------|----------------|
| Boiler Water Level | BWR_LVL_001       | Analog         | %                | 0 - 100 %                    | 50 %     | P01            |
| Feedwater Flow     | FWF_FLOW_001      | Analog         | m³/h              | 0 - 1000 m³/h                | N/A      | P02            |
| Feedwater Valve    | FWF_VALVE_001     | Analog         | %                | 0 - 100 %                    | N/A      | P02            |
| Feedwater Temp     | FWF_TEMP_001      | Analog         | °C                | 0 - 300 °C                   | N/A      | P03            |
| Feedwater Pump     | FWPUMP_001        | Digital        | On/Off            | Off: 0, On: 1                | N/A      | P04            |
| Deaerator Level    | DEA_LVL_001       | Analog         | %                | 0 - 100 %                    | 70 %     | P05            |
| Condenser Level    | CON_LVL_001       | Analog         | %                | 0 - 100 %                    | 30 %     | P06            |
| Condensate Pump    | CONPUMP_001       | Digital        | On/Off            | Off: 0, On: 1                | N/A      | P07            |
| Economizer Inlet   | ECON_INLET_001    | Analog         | °C                | 0 - 300 °C                   | N/A      | P08            |
| Economizer Outlet  | ECON_OUTLET_001   | Analog         | °C                | 0 - 300 °C                   | N/A      | P09            |
| Preheater Level    | PRE_LVL_001       | Analog         | %                | 0 - 100 %                    | 60 %     | P10            |
| Preheater Temp     | PRE_TEMP_001      | Analog         | °C                | 0 - 300 °C                   | N/A      | P11            |
| Feedwater Pressure | FWF_PRESS_001     | Analog         | Bar               | 0 - 200 Bar                  | N/A      | P12            |
| Deaerator Pressure | DEA_PRESS_001     | Analog         | Bar               | 0 - 200 Bar                  | N/A      | P13            |
| Condenser Pressure | CON_PRESS_001     | Analog         | Bar               | 0 - 200 Bar                  | N/A      | P14            |
| Economizer Valve   | ECON_VALVE_001    | Analog         | %                | 0 - 100 %                    | N/A      | P15            |
| Deaerator Valve    | DEA_VALVE_001     | Analog         | %                | 0 - 100 %                    | N/A      | P16            |
| Condensate Valve   | CON_VALVE_001     | Analog         | %                | 0 - 100 %                    | N/A      | P17            |
| Preheater Valve    | PRE_VALVE_001     | Analog         | %                | 0 - 100 %                    | N/A      | P18            |
| Feedwater Heater   | FWH_LEVEL_001     | Analog         | %                | 0 - 100 %                    | 80 %     | P19            |
| Feedwater Heater   | FWH_TEMP_001      | Analog         | °C                | 0 - 300 °C                   | N/A      | P20            |
| Feedwater Heater   | FWH_VALVE_001     | Analog         | %                | 0 - 100 %                    | N/A      | P21            |
| Feedwater Heater   | FWH_PRESS_001     | Analog         | Bar               | 0 - 200 Bar                  | N/A      | P22            |
| Boiler Feedwater   | BWR_FWF_001       | Analog         | m³/h              | 0 - 1000 m³/h                | N/A      | P23            |
| Boiler Pressure    | BWR_PRESS_001     | Analog         | Bar               | 0 - 200 Bar                  | N/A      | P24            |

This I/O list covers a range of signals essential for the control of feedwater in a steam-water cycle in a power plant. The setpoints provided are typical values for a given operation and can be adjusted according to specific process requirements.
```
