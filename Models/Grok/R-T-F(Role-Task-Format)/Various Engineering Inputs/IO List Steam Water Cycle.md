| Signal Name                          | Signal Tag Number | Analog/Digital | Engineering Unit | Range            | Setpoint        | P&ID Reference |
|--------------------------------------|-------------------|----------------|------------------|------------------|-----------------|----------------|
| Feedwater Flow Feedback              | FT-101            | Analog         | t/h              | 0–200            | 150             | FW-PID-001     |
| Steam Drum Level Feedback            | LT-102            | Analog         | %                | 0–100            | 50              | FW-PID-002     |
| Feedwater Temperature Feedback       | TT-103            | Analog         | °C               | 0–250            | 120             | FW-PID-003     |
| Boiler Drum Pressure Feedback        | PT-104            | Analog         | bar              | 0–200            | 160             | FW-PID-004     |
| Feedwater Control Valve Position     | ZT-105            | Analog         | %                | 0–100            | 75              | FW-PID-005     |
| Economizer Inlet Flow Feedback       | FT-106            | Analog         | t/h              | 0–200            | 150             | FW-PID-006     |
| Deaerator Level Feedback             | LT-107            | Analog         | %                | 0–100            | 60              | FW-PID-007     |
| Feedwater Pump Speed Feedback        | ST-108            | Analog         | RPM              | 0–3600           | 3000            | FW-PID-008     |
| Feedwater Pump Discharge Pressure    | PT-109            | Analog         | bar              | 0–250            | 180             | FW-PID-009     |
| Steam Flow Feedback                  | FT-110            | Analog         | t/h              | 0–200            | 150             | FW-PID-010     |
| Feedwater Control Valve Command      | FC-201            | Analog         | %                | 0–100            | 75              | FW-PID-005     |
| Feedwater Pump Speed Command         | SC-202            | Analog         | %                | 0–100            | 80              | FW-PID-008     |
| Economizer Bypass Valve Position     | ZC-203            | Analog         | %                | 0–100            | 20              | FW-PID-011     |
| Deaerator Vent Valve Position        | VC-204            | Analog         | %                | 0–100            | 10              | FW-PID-012     |
| Feedwater Pump Start/Stop Command    | PC-205            | Digital        | N/A              | 0/1 (False/True) | 1               | FW-PID-008     |
| Feedwater Pump Run Status            | PS-301            | Digital        | N/A              | 0/1 (False/True) | 1               | FW-PID-008     |
| Low Drum Level Alarm                 | LA-302            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-002     |
| High Drum Level Alarm                | LA-303            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-002     |
| High Feedwater Temperature Alarm     | TA-304            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-003     |
| Low Feedwater Flow Alarm             | FA-305            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-001     |
| Feedwater Pump Trip Status           | PS-306            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-008     |
| Emergency Shutdown Status            | ES-307            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-013     |
| Deaerator Low Level Switch           | LS-308            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-007     |
| High Drum Pressure Alarm             | PA-309            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-004     |
| Feedwater Control Valve Open/Close   | ZC-401            | Digital        | N/A              | 0/1 (False/True) | 1               | FW-PID-005     |
| Economizer Bypass Valve Command      | VC-402            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-011     |
| Deaerator Vent Valve Command         | VC-403            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-012     |
| Low Drum Level Trip                  | LT-404            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-002     |
| High Feedwater Pressure Alarm        | PA-405            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-009     |
| Feedwater Pump Vibration Alarm       | VA-406            | Digital        | N/A              | 0/1 (False/True) | 0               | FW-PID-008     |

(* Notes *)
(* - Signal Tag Numbers follow format: XX-NNN (XX = type, e.g., FT=Flow, LT=Level, NNN=unique ID) *)
(* - Analog signals use 4-20 mA, scaled to engineering units *)
(* - Digital signals are 24 VDC, 0/1 for False/True *)
(* - Setpoints are typical operating values, adjustable via HMI *)
(* - P&ID References link to specific diagrams for traceability *)
(* - Table is formatted for Excel import (pipe-delimited) and PLC/HMI configuration *)
