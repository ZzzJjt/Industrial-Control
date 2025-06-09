\documentclass{article}
\usepackage{longtable}
\usepackage{geometry}
\geometry{margin=1in}

\begin{document}

\section*{Rolling Mill I/O Specification Table}

\begin{longtable}{|l|c|l|c|l|c|c|l|}
\hline
\textbf{Signal Name} & \textbf{I/O} & \textbf{Tag Number} & \textbf{A/D} & \textbf{Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\
\hline
Roll Speed Feedback & Input  & RM1001 & Analog  & m/min & 0–1500 & N/A   & P\&ID-01 \\
Roll Speed Command  & Output & RM1002 & Analog  & m/min & 0–1500 & 1200  & P\&ID-01 \\
Main Motor Torque   & Input  & RM1003 & Analog  & Nm    & 0–10000 & N/A  & P\&ID-02 \\
Main Motor Command  & Output & RM1004 & Analog  & Nm    & 0–10000 & 8500 & P\&ID-02 \\
Entry Tension       & Input  & RM1005 & Analog  & N     & 0–100000 & N/A & P\&ID-03 \\
Exit Tension        & Input  & RM1006 & Analog  & N     & 0–100000 & N/A & P\&ID-03 \\
Tension Setpoint    & Output & RM1007 & Analog  & N     & 0–100000 & 75000 & P\&ID-03 \\
Gap Feedback        & Input  & RM1008 & Analog  & mm    & 0–20     & N/A  & P\&ID-04 \\
Gap Setpoint        & Output & RM1009 & Analog  & mm    & 0–20     & 12.5 & P\&ID-04 \\
Hydraulic Pressure  & Input  & RM1010 & Analog  & bar   & 0–250    & N/A  & P\&ID-05 \\
Cooling Valve Cmd   & Output & RM1011 & Digital & On/Off & N/A    & On   & P\&ID-06 \\
Cooling Temp        & Input  & RM1012 & Analog  & °C    & 20–80    & N/A  & P\&ID-06 \\
Entry Guide Pos     & Input  & RM1013 & Analog  & mm    & 0–100    & N/A  & P\&ID-07 \\
Exit Guide Pos      & Input  & RM1014 & Analog  & mm    & 0–100    & N/A  & P\&ID-07 \\
Strip Width         & Input  & RM1015 & Analog  & mm    & 500–2000 & N/A  & P\&ID-08 \\
Strip Thickness     & Input  & RM1016 & Analog  & mm    & 0.1–20   & N/A  & P\&ID-09 \\
Thickness Setpoint  & Output & RM1017 & Analog  & mm    & 0.1–20   & 2.0  & P\&ID-09 \\
Roll Vibration      & Input  & RM1018 & Analog  & mm/s  & 0–10     & N/A  & P\&ID-10 \\
Emergency Stop      & Input  & RM1019 & Digital & On/Off & N/A    & Off  & P\&ID-11 \\
Start Command       & Input  & RM1020 & Digital & On/Off & N/A    & On   & P\&ID-11 \\
Stop Command        & Input  & RM1021 & Digital & On/Off & N/A    & Off  & P\&ID-11 \\
Line Speed Ref      & Output & RM1022 & Analog  & m/min & 0–1500  & 1300 & P\&ID-01 \\
Interlock Status    & Input  & RM1023 & Digital & On/Off & N/A    & On   & P\&ID-12 \\
Drive Ready         & Input  & RM1024 & Digital & On/Off & N/A    & On   & P\&ID-12 \\
Overload Alarm      & Input  & RM1025 & Digital & On/Off & N/A    & Off  & P\&ID-13 \\
Strip Break Sensor  & Input  & RM1026 & Digital & On/Off & N/A    & Off  & P\&ID-14 \\
Oil Pressure        & Input  & RM1027 & Analog  & bar    & 0–100  & N/A  & P\&ID-15 \\
Oil Temp            & Input  & RM1028 & Analog  & °C     & 20–90  & N/A  & P\&ID-15 \\
Drive Fault Reset   & Output & RM1029 & Digital & On/Off & N/A    & On   & P\&ID-13 \\
Manual Mode Select  & Input  & RM1030 & Digital & On/Off & N/A    & Off  & P\&ID-16 \\
\hline
\end{longtable}

\end{document}
