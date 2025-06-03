\begin{table}[htbp]
\centering
\caption{Extended Cause and Process Action Interlock Matrix for Chemical Reactor}
\renewcommand{\arraystretch}{1.3}
\begin{tabular}{|p{3.5cm}|c|c|c|c|c|c|c|c|}
\hline
\textbf{Cause Description} & \textbf{A1} \\small{Close Feed} & \textbf{A2} \\small{Vent Reactor} & \textbf{A3} \\small{Start Cooling} & \textbf{A4} \\small{Stop Agitator} & \textbf{A5} \\small{Close Outlet} & \textbf{A6} \\small{Lock Setpoints} & \textbf{A7} \\small{Alarm} & \textbf{A8} \\small{ESD} \\
\hline
C1: Pressure High             & ✔️ & ✔️ &     & ✔️ & ✔️ & ✔️ & ✔️ &     \\
C2: Pressure Low              & ✔️ &     &     &     & ✔️ & ✔️ & ✔️ &     \\
C3: Temperature High          & ✔️ &     & ✔️ & ✔️ &     & ✔️ & ✔️ &     \\
C4: Temperature Low           &     &     &     &     &     &     & ✔️ &     \\
C5: Level Sensor Fault        & ✔️ &     &     &     &     &     & ✔️ &     \\
C6: Cooling Flow Low          & ✔️ &     & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ &     \\
C7: Feed Valve Stuck Open     & ✔️ &     &     &     &     & ✔️ & ✔️ &     \\
C8: Agitator Motor Fault      & ✔️ &     & ✔️ &     & ✔️ & ✔️ & ✔️ &     \\
C9: Emergency Stop Activated  & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ \\
C10: PLC Heartbeat Lost       & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ & ✔️ \\
\hline
\end{tabular}
\label{tab:interlock-matrix}
\end{table}
