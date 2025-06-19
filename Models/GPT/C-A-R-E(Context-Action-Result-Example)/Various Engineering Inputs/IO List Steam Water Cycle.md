\begin{table}[htbp]
\centering
\footnotesize
\begin{tabular}{|l|l|c|l|l|l|l|}
\hline
\textbf{Signal Name} & \textbf{Tag Number} & \textbf{A/D} & \textbf{Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\
\hline
Drum Level Transmitter & FW1001 & A & \% & 0--100 & 50.0 & P\&ID-FW-01 \\
Feedwater Flow Transmitter & FW1002 & A & t/h & 0--120 & 80.0 & P\&ID-FW-02 \\
Feedwater Control Valve & FWCV1003 & A & \% & 0--100 & Auto & P\&ID-FW-02 \\
Pump Start Feedback & FWPS1004 & D & -- & 0 or 1 & 1 & P\&ID-FW-03 \\
Drum Level High Alarm & FWAL1005 & D & -- & 0 or 1 & 0 & P\&ID-FW-01 \\
Drum Level Low Alarm & FWAL1006 & D & -- & 0 or 1 & 0 & P\&ID-FW-01 \\
Pump Running Status & FWPS1007 & D & -- & 0 or 1 & 1 & P\&ID-FW-03 \\
Pump Control Command & FWCMD1008 & D & -- & 0 or 1 & Auto & P\&ID-FW-03 \\
Control Valve Position Feedback & FWFB1009 & A & \% & 0--100 & -- & P\&ID-FW-02 \\
Feedwater Temperature & FWTEMP1010 & A & °C & 20--250 & 180 & P\&ID-FW-04 \\
Boiler Steam Flow Feedback & FWSF1011 & A & t/h & 0--200 & 80.0 & P\&ID-FW-05 \\
Level Setpoint Input & FWSP1012 & A & \% & 0--100 & 50.0 & P\&ID-FW-01 \\
Flow Setpoint Input & FWSP1013 & A & t/h & 0--120 & 80.0 & P\&ID-FW-02 \\
PID Controller Output & FWPID1014 & A & \% & 0--100 & -- & P\&ID-FW-06 \\
High-High Drum Level Trip & FWTRIP1015 & D & -- & 0 or 1 & 0 & P\&ID-FW-01 \\
Low-Low Drum Level Trip & FWTRIP1016 & D & -- & 0 or 1 & 0 & P\&ID-FW-01 \\
Main Feedwater Pump Current & FWCUR1017 & A & A & 0--500 & -- & P\&ID-FW-03 \\
Control Valve Manual Override & FWMAN1018 & D & -- & 0 or 1 & 0 & P\&ID-FW-02 \\
System Enable Command & FWSYS1019 & D & -- & 0 or 1 & 1 & P\&ID-FW-06 \\
Emergency Stop Input & FWESTOP1020 & D & -- & 0 or 1 & 0 & P\&ID-FW-07 \\
\hline
\end{tabular}
\caption{Feedwater Control Loop I/O List for Steam-Water Cycle}
\label{tab:feedwater_io_list}
\end{table}
