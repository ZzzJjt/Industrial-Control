\documentclass{article}
\usepackage{tabularx}
\usepackage{geometry}
\geometry{margin=1in}

\begin{document}

\section*{Rolling Mill I/O List}

\small
\begin{tabularx}{\textwidth}{|l|c|l|c|l|l|l|l|}
\hline
\textbf{Signal Name} & \textbf{I/O} & \textbf{Tagnumber} & \textbf{A/D} & \textbf{Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\
\hline
Roll Speed Feedback & I & AI001 & A & m/min & 0–800 & 600 & P\&ID-01 \\
Main Drive Current & I & AI002 & A & A & 0–400 & — & P\&ID-02 \\
Entry Tension & I & AI003 & A & kN & 0–100 & 50 & P\&ID-03 \\
Exit Tension & I & AI004 & A & kN & 0–100 & 60 & P\&ID-04 \\
Mill Pressure & I & AI005 & A & bar & 0–250 & 200 & P\&ID-05 \\
Strip Thickness & I & AI006 & A & mm & 0.2–10.0 & 2.5 & P\&ID-06 \\
Cooling Water Temp & I & AI007 & A & °C & 10–80 & — & P\&ID-07 \\
Entry Loop Position & I & AI008 & A & mm & 0–1000 & — & P\&ID-08 \\
Exit Loop Position & I & AI009 & A & mm & 0–1000 & — & P\&ID-09 \\
Strip Width & I & AI010 & A & mm & 800–2000 & — & P\&ID-10 \\
Roll Gap Setpoint & O & AO001 & A & mm & 0–10 & 2.5 & P\&ID-11 \\
Hydraulic Valve Cmd & O & AO002 & A & \% & 0–100 & — & P\&ID-12 \\
Speed Ref Main Motor & O & AO003 & A & m/min & 0–800 & 600 & P\&ID-13 \\
Cooling Valve Cmd & O & AO004 & A & \% & 0–100 & 70 & P\&ID-14 \\
Lubrication Valve Cmd & O & AO005 & A & \% & 0–100 & — & P\&ID-15 \\
Emergency Stop & I & DI001 & D & — & On/Off & — & P\&ID-16 \\
Entry Coil Present & I & DI002 & D & — & On/Off & — & P\&ID-17 \\
Exit Coil Present & I & DI003 & D & — & On/Off & — & P\&ID-18 \\
Roll Gap Limit Switch & I & DI004 & D & — & On/Off & — & P\&ID-19 \\
Cooling On Status & I & DI005 & D & — & On/Off & — & P\&ID-20 \\
Mill Running Status & I & DI006 & D & — & On/Off & — & P\&ID-21 \\
Entry Loop Sensor & I & DI007 & D & — & On/Off & — & P\&ID-22 \\
Exit Loop Sensor & I & DI008 & D & — & On/Off & — & P\&ID-23 \\
Main Motor Fault & I & DI009 & D & — & On/Off & — & P\&ID-24 \\
Drive Ready & I & DI010 & D & — & On/Off & — & P\&ID-25 \\
Entry Guide Actuate & O & DO001 & D & — & On/Off & — & P\&ID-26 \\
Exit Guide Actuate & O & DO002 & D & — & On/Off & — & P\&ID-27 \\
Cooling On Cmd & O & DO003 & D & — & On/Off & — & P\&ID-28 \\
Alarm Horn Cmd & O & DO004 & D & — & On/Off & — & P\&ID-29 \\
Warning Light Cmd & O & DO005 & D & — & On/Off & — & P\&ID-30 \\
\hline
\end{tabularx}
\normalsize

\end{document}
