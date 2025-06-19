\begin{table}[htbp]
\centering
\footnotesize
\begin{tabular}{|l|c|l|c|l|l|l|l|}
\hline
\textbf{Signal Name} & \textbf{I/O} & \textbf{Tag Number} & \textbf{A/D} & \textbf{Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\
\hline
EntryRollSpeed & IN & RM1001 & A & m/min & 0--1500 & 1200 & P\&ID-01 \\
RollGapFeedback & IN & RM1002 & A & mm & 0--50 & 22.5 & P\&ID-01 \\
RollGapSetpoint & OUT & RM1003 & A & mm & 0--50 & 22.5 & P\&ID-02 \\
CoolingValveOpen & OUT & RM2001 & D & -- & 0 or 1 & 1 & P\&ID-03 \\
TensionFeedback & IN & RM1004 & A & kN & 0--100 & 75 & P\&ID-01 \\
TensionSetpoint & OUT & RM1005 & A & kN & 0--100 & 75 & P\&ID-02 \\
MainMotorSpeed & OUT & RM3001 & A & RPM & 0--1800 & 1500 & P\&ID-04 \\
MotorCurrent & IN & RM3002 & A & A & 0--800 & -- & P\&ID-04 \\
RollerTemp & IN & RM1006 & A & °C & 0--200 & 120 & P\&ID-05 \\
CoolantFlowRate & IN & RM1007 & A & L/min & 0--500 & 300 & P\&ID-05 \\
CoolantValveCmd & OUT & RM2002 & D & -- & 0 or 1 & 1 & P\&ID-06 \\
StripWidthSensor & IN & RM1008 & A & mm & 500--1500 & 1000 & P\&ID-07 \\
StripThickness & IN & RM1009 & A & mm & 0.1--10 & 1.2 & P\&ID-07 \\
LubricationPumpCmd & OUT & RM2003 & D & -- & 0 or 1 & 1 & P\&ID-08 \\
EmergencyStop & IN & RM4001 & D & -- & 0 or 1 & 0 & P\&ID-09 \\
StripPresence & IN & RM4010 & D & -- & 0 or 1 & 1 & P\&ID-09 \\
RollGapManualAdj & IN & RM1010 & A & mm & -5--5 & 0 & P\&ID-10 \\
ExitRollSpeed & IN & RM1011 & A & m/min & 0--1500 & 1200 & P\&ID-11 \\
ExitTempSensor & IN & RM1012 & A & °C & 0--250 & 140 & P\&ID-11 \\
ExitCoolingValve & OUT & RM2004 & D & -- & 0 or 1 & 1 & P\&ID-12 \\
StripTensionOK & IN & RM4011 & D & -- & 0 or 1 & 1 & P\&ID-12 \\
HydraulicPressure & IN & RM1013 & A & bar & 0--250 & 180 & P\&ID-13 \\
HydraulicPumpCmd & OUT & RM2005 & D & -- & 0 or 1 & 1 & P\&ID-13 \\
RollPositionSensor & IN & RM1014 & A & mm & 0--100 & -- & P\&ID-14 \\
DriveReady & IN & RM4012 & D & -- & 0 or 1 & 1 & P\&ID-14 \\
MotorOverload & IN & RM4013 & D & -- & 0 or 1 & 0 & P\&ID-15 \\
LevelSensor & IN & RM1015 & A & \% & 0--100 & 85 & P\&ID-16 \\
TempHighAlarm & IN & RM4014 & D & -- & 0 or 1 & 0 & P\&ID-16 \\
PressureReliefValve & OUT & RM2006 & D & -- & 0 or 1 & 0 & P\&ID-17 \\
SystemStatus & OUT & RM5001 & D & -- & 0 or 1 & 1 & P\&ID-18 \\
\hline
\end{tabular}
\caption{I/O List for Rolling Mill Control System}
\label{tab:rolling_mill_io}
\end{table}
