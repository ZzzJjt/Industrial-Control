Signal Name                    | I/O   | Signal Tagnumber | A/D | Engineering Unit | Range          | Setpoint     | P&ID Reference
------------------------------|-------|------------------|-----|------------------|----------------|--------------|---------------
EntryRollSpeed                | IN    | RM1001           | A   | m/min            | 0–1500         | 1200         | P&ID-01
RollGapFeedback               | IN    | RM1002           | A   | mm               | 0–50           | 22.5         | P&ID-01
RollGapSetpoint               | OUT   | RM1003           | A   | mm               | 0–50           | 22.5         | P&ID-02
ExitRollSpeed                 | IN    | RM1004           | A   | m/min            | 0–1500         | 1220         | P&ID-01
TensionFeedbackEntry          | IN    | RM1005           | A   | kN               | 0–500          | 200          | P&ID-01
TensionFeedbackExit           | IN    | RM1006           | A   | kN               | 0–500          | 180          | P&ID-01
RollForce                     | IN    | RM1007           | A   | kN               | 0–10000        | 5000         | P&ID-01
RollMotorCurrent              | IN    | RM1008           | A   | A                | 0–2000         | 1500         | P&ID-03
RollMotorSpeedSetpoint        | OUT   | RM1009           | A   | RPM              | 0–1800         | 1600         | P&ID-03
RollMotorStart                | OUT   | RM1010           | D   | —                | 0 or 1         | 1            | P&ID-03
RollMotorStop                 | OUT   | RM1011           | D   | —                | 0 or 1         | 0            | P&ID-03
CoolingWaterFlow              | IN    | RM2001           | A   | m³/h             | 0–100          | 50           | P&ID-04
CoolingWaterValvePosition     | OUT   | RM2002           | A   | %                | 0–100          | 60           | P&ID-04
CoolingWaterTemp              | IN    | RM2003           | A   | °C               | 0–80           | 25           | P&ID-04
SlabTemperatureEntry          | IN    | RM2004           | A   | °C               | 0–1200         | 1000         | P&ID-05
SlabTemperatureExit           | IN    | RM2005           | A   | °C               | 0–1200         | 950          | P&ID-05
LubricantFlow                 | IN    | RM2006           | A   | L/min            | 0–20           | 10           | P&ID-06
LubricantValvePosition        | OUT   | RM2007           | A   | %                | 0–100          | 50           | P&ID-06
HydraulicPressure             | IN    | RM2008           | A   | bar              | 0–300          | 200          | P&ID-07
HydraulicPumpEnable           | OUT   | RM2009           | D   | —                | 0 or 1         | 1            | P&ID-07
StripThicknessFeedback        | IN    | RM2010           | A   | mm               | 0–10           | 2.5          | P&ID-08
StripThicknessSetpoint        | OUT   | RM2011           | A   | mm               | 0–10           | 2.5          | P&ID-08
StripWidthFeedback            | IN    | RM2012           | A   | mm               | 500–2000       | 1500         | P&ID-08
EmergencyStop                 | IN    | RM3001           | D   | —                | 0 or 1         | 0            | P&ID-09
HighTensionAlarm              | OUT   | RM3002           | D   | —                | 0 or 1         | 0            | P&ID-09
OverTemperatureAlarm          | OUT   | RM3003           | D   | —                | 0 or 1         | 0            | P&ID-09
LowHydraulicPressureAlarm     | OUT   | RM3004           | D   | —                | 0 or 1         | 0            | P&ID-09
RollGapAdjustEnable           | OUT   | RM3005           | D   | —                | 0 or 1         | 1            | P&ID-02
CoolingSystemFault            | IN    | RM3006           | D   | —                | 0 or 1         | 0            | P&ID-04
RunCommand                    | IN    | RM3007           | D   | —                | 0 or 1         | 1            | P&ID-09
