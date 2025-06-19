Certainly! Below is a structured I/O list table with 30 signals for a rolling mill. Each row represents one signal and includes the specified columns: Signal Name, Input/Output, Signal Tagnumber, Analog/Digital, Engineering Unit, Range, Setpoint, and P&ID Reference.

### Rolling Mill I/O List

| Signal Name          | I/O   | Tag Number | A/D    | Engineering Unit | Range        | Setpoint       | P&ID Ref |
|----------------------|-------|------------|--------|--------------------|--------------|----------------|----------|
| EntryRollSpeed       | IN    | RM1001     | A      | m/min              | 0-1500       | 1200           | P&ID-01  |
| RollGapFeedback      | IN    | RM1002     | A      | mm                 | 0-50         | 22.5           | P&ID-01  |
| RollGapSetpoint      | OUT   | RM1003     | A      | mm                 | 0-50         | 22.5           | P&ID-02  |
| CoolingValveOpen     | OUT   | RM2001     | D      | -                  | 0 or 1       | 1              | P&ID-03  |
| ExitRollSpeed        | IN    | RM1004     | A      | m/min              | 0-1500       | 1200           | P&ID-04  |
| MotorDriveStatus     | IN    | RM2002     | D      | -                  | 0 or 1       | 1              | P&ID-05  |
| MotorTorque          | IN    | RM1005     | A      | kN                 | 0-500        | -              | P&ID-06  |
| MotorCurrent         | IN    | RM1006     | A      | A                  | 0-1000       | -              | P&ID-07  |
| OilPressure          | IN    | RM1007     | A      | bar                | 0-10         | 6              | P&ID-08  |
| OilTemperature       | IN    | RM1008     | A      | °C                 | 0-100        | 40             | P&ID-09  |
| HydraulicPressure    | IN    | RM1009     | A      | bar                | 0-20         | 15             | P&ID-10  |
| TensionFeedback      | IN    | RM1010     | A      | kN                 | 0-500        | 250            | P&ID-11  |
| TensionSetpoint      | OUT   | RM1011     | A      | kN                 | 0-500        | 250            | P&ID-12  |
| StripThickness       | IN    | RM1012     | A      | mm                 | 0-20         | 1.5            | P&ID-13  |
| StripWidth           | IN    | RM1013     | A      | mm                 | 0-300        | 250            | P&ID-14  |
| StripSurfaceTemp     | IN    | RM1014     | A      | °C                 | 0-300        | 200            | P&ID-15  |
| CoilerMotorSpeed     | OUT   | RM2003     | A      | m/min              | 0-500        | 300            | P&ID-16  |
| CoilerTension        | IN    | RM1015     | A      | kN                 | 0-500        | 200            | P&ID-17  |
| CoilerPosition       | IN    | RM1016     | A      | mm                 | 0-1000       | 500            | P&ID-18  |
| ShearActuatorPos     | IN    | RM1017     | A      | mm                 | 0-500        | 250            | P&ID-19  |
| ShearActuatorCmd     | OUT   | RM2004     | A      | mm                 | 0-500        | 250            | P&ID-20  |
| FanSpeed             | OUT   | RM2005     | A      | Hz                 | 0-1000       | 500            | P&ID-21  |
| AirflowRate          | IN    | RM1018     | A      | m³/min             | 0-100        | 50             | P&ID-22  |
| WaterFlowRate        | IN    | RM1019     | A      | m³/min             | 0-100        | 40             | P&ID-23  |
| WaterTemperature     | IN    | RM1020     | A      | °C                 | 0-100        | 30             | P&ID-24  |
| LubricantLevel       | IN    | RM1021     | A      | mm                 | 0-1000       | 500            | P&ID-25  |
| LubricantPressure    | IN    | RM1022     | A      | bar                | 0-10         | 5              | P&ID-26  |
| EmergencyStopButton  | IN    | RM2006     | D      | -                  | 0 or 1       | 0              | P&ID-27  |
| SafetyDoorClosed     | IN    | RM2007     | D      | -                  | 0 or 1       | 1              | P&ID-28  |
| PowerSupplyStatus    | IN    | RM2008     | D      | -                  | 0 or 1       | 1              | P&ID-29  |
| SystemMode           | OUT   | RM2009     | D      | -                  | 0 or 1       | 0 (Manual)     | P&ID-30  |

### Explanation of the I/O List

- **Signal Name:** Describes the purpose of the signal.
- **I/O:** Indicates whether the signal is an input (`IN`) or output (`OUT`).
- **Tag Number:** Unique identifier for each signal.
- **A/D:** Specifies if the signal is analog (`A`) or digital (`D`).
- **Engineering Unit:** The unit of measurement for the signal.
- **Range:** The valid range of values for the signal.
- **Setpoint:** The target value for control purposes (where applicable).
- **P&ID Ref:** Reference to the Process and Instrumentation Diagram (P&ID) for the signal.

This I/O list serves as a comprehensive reference for system design, PLC programming, field wiring, and control documentation, ensuring clear communication between mechanical, electrical, and automation teams.
