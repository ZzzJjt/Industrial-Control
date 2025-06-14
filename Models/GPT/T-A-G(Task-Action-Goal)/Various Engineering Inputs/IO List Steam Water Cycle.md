\begin{table}[H]
\centering
\caption{Feedwater Control System I/O List}
\begin{tabular}{|l|l|l|l|l|l|l|}
\hline
\textbf{Signal Name} & \textbf{Tag Number} & \textbf{Analog/Digital} & \textbf{Engineering Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\ \hline
Drum Level Transmitter 1 & LT-101 & Analog & \% & 0--100 & 50 & PFD-01 \\ \hline
Drum Level Transmitter 2 & LT-102 & Analog & \% & 0--100 & 50 & PFD-01 \\ \hline
Feedwater Flow Transmitter 1 & FT-201 & Analog & t/h & 0--150 & 120 & PFD-02 \\ \hline
Feedwater Flow Transmitter 2 & FT-202 & Analog & t/h & 0--150 & 120 & PFD-02 \\ \hline
Feedwater Control Valve 1 & FV-301 & Digital & - & Open/Close & Open & PFD-03 \\ \hline
Feedwater Control Valve 2 & FV-302 & Digital & - & Open/Close & Open & PFD-03 \\ \hline
Pump A Running Status & PS-401 & Digital & - & On/Off & On & PFD-04 \\ \hline
Pump B Running Status & PS-402 & Digital & - & On/Off & On & PFD-04 \\ \hline
Pump A Current Sensor & CS-403 & Analog & A & 0--100 & 65 & PFD-04 \\ \hline
Pump B Current Sensor & CS-404 & Analog & A & 0--100 & 65 & PFD-04 \\ \hline
Drum Pressure Transmitter & PT-501 & Analog & bar & 0--100 & 85 & PFD-01 \\ \hline
High Drum Level Alarm & LAH-601 & Digital & - & Active/Inactive & Inactive & PFD-01 \\ \hline
Low Drum Level Alarm & LAL-602 & Digital & - & Active/Inactive & Inactive & PFD-01 \\ \hline
Low Feedwater Flow Alarm & FAL-701 & Digital & - & Active/Inactive & Inactive & PFD-02 \\ \hline
Motor A Overload Trip & TR-801 & Digital & - & Trip/Reset & Reset & PFD-04 \\ \hline
\end{tabular}
\end{table}
