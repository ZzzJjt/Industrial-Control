\begin{table}[htbp]
\centering
\caption{Cause and Process Action Matrix for Reactor Interlocks}
\begin{tabular}{|l|c|c|c|c|c|c|}
\hline
\textbf{Cause} & \textbf{Close Feed Valve} & \textbf{Stop Agitator} & \textbf{Open Relief Valve} & \textbf{Stop Heating} & \textbf{Trigger Alarm} & \textbf{Safe Shutdown} \\
\hline
High Pressure         & \checkmark &              & \checkmark & \checkmark & \checkmark &              \\
Low Temperature       &            &              &            &            & \checkmark & \checkmark   \\
High Temperature      & \checkmark &              &            & \checkmark & \checkmark &              \\
Level Sensor Failure  & \checkmark & \checkmark   &            &            & \checkmark & \checkmark   \\
Overfill              & \checkmark &              &            &            & \checkmark &              \\
Agitator Motor Fault  &            & \checkmark   &            &            & \checkmark & \checkmark   \\
Gas Leak Detected     & \checkmark & \checkmark   & \checkmark & \checkmark & \checkmark & \checkmark   \\
\hline
\end{tabular}
\label{tab:interlock_matrix}
\end{table}
