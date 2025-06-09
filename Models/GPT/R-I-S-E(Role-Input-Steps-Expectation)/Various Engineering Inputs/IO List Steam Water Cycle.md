\documentclass{article}
\usepackage{longtable}
\usepackage{geometry}
\geometry{margin=1in}

\begin{document}

\section*{Feedwater Control System – I/O Specification Table}

\begin{longtable}{|l|l|c|l|c|c|l|}
\hline
\textbf{Signal Name} & \textbf{Signal Tag Number} & \textbf{A/D} & \textbf{Engineering Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\
\hline
Drum Level Transmitter (LT)      & FW1001 & Analog  & \%       & 0–100     & 50       & P\&ID-FW01 \\
Drum Level Low-Alarm Switch (LSL) & FW1002 & Digital & On/Off   & N/A       & N/A      & P\&ID-FW01 \\
Feedwater Flow Transmitter (FT)  & FW1003 & Analog  & t/h      & 0–500     & 350      & P\&ID-FW02 \\
Feedwater Control Valve Position (CV) & FW1004 & Analog  & \%       & 0–100     & Auto     & P\&ID-FW03 \\
Feedwater Control Valve Open Command & FW1005 & Digital & On/Off   & N/A       & On       & P\&ID-FW03 \\
Feedwater Control Valve Close Command & FW1006 & Digital & On/Off   & N/A       & Off      & P\&ID-FW03 \\
Boiler Drum Pressure Transmitter & FW1007 & Analog  & bar      & 0–150     & 80       & P\&ID-FW04 \\
Feedwater Pump 1 Start Command   & FW1008 & Digital & On/Off   & N/A       & On       & P\&ID-FW05 \\
Feedwater Pump 1 Running Status & FW1009 & Digital & On/Off   & N/A       & On       & P\&ID-FW05 \\
Feedwater Pump 2 Start Command   & FW1010 & Digital & On/Off   & N/A       & Off      & P\&ID-FW06 \\
Feedwater Pump 2 Running Status & FW1011 & Digital & On/Off   & N/A       & Off      & P\&ID-FW06 \\
Feedwater Pump Speed Feedback   & FW1012 & Analog  & RPM      & 0–3600    & 3000     & P\&ID-FW05 \\
Control Loop Output to CV       & FW1013 & Analog  & \%       & 0–100     & Auto     & P\&ID-FW03 \\
Low Drum Level Trip Interlock   & FW1014 & Digital & On/Off   & N/A       & Off      & P\&ID-FW01 \\
Feedwater Temperature Transmitter & FW1015 & Analog  & °C       & 20–240    & 120      & P\&ID-FW07 \\
Feedwater Header Pressure Transmitter & FW1016 & Analog & bar     & 0–100     & 40       & P\&ID-FW07 \\
Boiler Master Setpoint          & FW1017 & Analog  & \%       & 0–100     & 65       & P\&ID-FW08 \\
Pump Seal Pressure Monitor      & FW1018 & Analog  & bar      & 0–25      & 10       & P\&ID-FW09 \\
Pump Motor Overload Alarm       & FW1019 & Digital & On/Off   & N/A       & Off      & P\&ID-FW09 \\
Feedwater Isolation Valve Status & FW1020 & Digital & Open/Closed & N/A     & Open     & P\&ID-FW10 \\
\hline
\end{longtable}

\end{document}
