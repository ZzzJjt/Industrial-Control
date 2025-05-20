| Signal Name                          | Input/Output | Signal Tagnumber | Analog/Digital | Engineering Unit | Range            | Setpoint        | P&ID Reference |
|--------------------------------------|--------------|------------------|----------------|------------------|------------------|-----------------|----------------|
| Main Roll Motor Speed Feedback       | Input        | ST-101           | Analog         | RPM              | 0–1200           | 1000            | RM-PID-001     |
| Roll Gap Position Feedback           | Input        | PT-102           | Analog         | mm               | 0–50             | 10              | RM-PID-002     |
| Hydraulic Pressure Feedback          | Input        | PT-103           | Analog         | bar              | 0–200            | 150             | RM-PID-003     |
| Strip Tension Feedback               | Input        | TT-104           | Analog         | kN               | 0–100            | 50              | RM-PID-004     |
| Cooling Water Flow Feedback          | Input        | FT-105           | Analog         | L/min            | 0–500            | 400             | RM-PID-005     |
| Roll Temperature Feedback            | Input        | TT-106           | Analog         | °C               | 0–300            | 80              | RM-PID-006     |
| Strip Thickness Feedback             | Input        | MT-107           | Analog         | mm               | 0–10             | 2               | RM-PID-007     |
| Main Roll Motor Speed Command        | Output       | SC-201           | Analog         | %                | 0–100            | 80              | RM-PID-001     |
| Roll Gap Position Command            | Output       | PC-202           | Analog         | mm               | 0–50             | 10              | RM-PID-002     |
| Hydraulic Pressure Command           | Output       | PC-203           | Analog         | bar              | 0–200            | 150             | RM-PID-003     |
| Cooling Valve Position Command       | Output       | FC-204           | Analog         | %                | 0–100            | 75              | RM-PID-005     |
| Strip Tension Control Command        | Output       | TC-205           | Analog         | %                | 0–100            | 60              | RM-PID-004     |
| Emergency Stop Status                | Input        | ES-301           | Digital        | N/A              | 0/1 (False/True) | 0               | RM-PID-008     |
| Roll Motor Run Status                | Input        | MS-302           | Digital        | N/A              | 0/1 (False/True) | 1               | RM-PID-001     |
| Hydraulic Pump Status                | Input        | PS-303           | Digital        | N/A              | 0/1 (False/True) | 1               | RM-PID-003     |
| Cooling System Status                | Input        | CS-304           | Digital        | N/A              | 0/1 (False/True) | 1               | RM-PID-005     |
| Strip Presence Sensor                | Input        | SS-305           | Digital        | N/A              | 0/1 (False/True) | 1               | RM-PID-009     |
| High Temperature Alarm               | Input        | TA-306           | Digital        | N/A              | 0/1 (False/True) | 0               | RM-PID-006     |
| Low Hydraulic Pressure Alarm         | Input        | PA-307           | Digital        | N/A              | 0/1 (False/True) | 0               | RM-PID-003     |
| Strip Break Detection                | Input        | SB-308           | Digital        | N/A              | 0/1 (False/True) | 0               | RM-PID-004     |
| Emergency Stop Command               | Output       | EC-401           | Digital        | N/A              | 0/1 (False/True) | 0               | RM-PID-008     |
| Roll Motor Start/Stop Command        | Output       | MC-402           | Digital        | N/A              | 0/1 (False/True) | 1               | RM-PID-001     |
| Hydraulic Pump Start/Stop Command    | Output       | PC-403           | Digital        | N/A              | 0/1 (False/True) | 1               | RM-PID-003     |
| Cooling Pump Start/Stop Command      | Output       | CC-404           | Digital        | N/A              | 0/1 (False/True) | 1               | RM-PID-005     |
| Strip Tension Alarm Command          | Output       | TC-405           | Digital        | N/A              | 0/1 (False/True) | 0               | RM-PID-004     |
| Roll Gap Limit Switch                | Input        | LS-309           | Digital        | N/A              | 0/1 (False/True) | 0               | RM-PID-002     |
| Lubricant Flow Feedback              | Input        | FT-310           | Analog         | L/min            | 0–50             | 30              | RM-PID-010     |
| Lubricant Valve Command              | Output       | FC-406           | Analog         | %                | 0–100            | 60              | RM-PID-010     |
| Strip Speed Feedback                 | Input        | ST-311           | Analog         | m/min            | 0–600            | 500             | RM-PID-011     |
| Strip Speed Command                  | Output       | SC-407           | Analog         | %                | 0–100            | 85              | RM-PID-011     |

(* Notes *)
(* - Signal Tagnumbers follow format: XX-NNN (XX = type, e.g., ST=Speed, PT=Position, NNN=unique ID) *)
(* - Analog signals use 4-20 mA or 0-10 V, scaled to engineering units *)
(* - Digital signals are 24 VDC, 0/1 for False/True *)
(* - Setpoints are typical operating values, adjustable via HMI *)
(* - P&ID References link to specific diagrams for traceability *)
(* - Table is formatted for Excel import (pipe-delimited) and HMI/PLC configuration *)
