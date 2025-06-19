\begin{table}[htbp]
\centering
\caption{Cause and Effect Matrix for Chemical Reactor Interlocks}
\begin{tabular}{|l|c|c|c|c|c|}
\hline
\textbf{Cause (Sensor Event)} & \textbf{Trigger Alarm} & \textbf{Close Feed Valve} & \textbf{Stop Feed Pump} & \textbf{Activate Relief System} & \textbf{Isolate Reactor} \\
\hline
High Pressure           & \checkmark & \checkmark & \checkmark & \checkmark &              \\
Low Pressure            & \checkmark &            & \checkmark &            &              \\
High Temperature        & \checkmark & \checkmark &            &            & \checkmark   \\
Low Temperature         & \checkmark &            &            &            &              \\
High Reactor Level      & \checkmark & \checkmark & \checkmark &            &              \\
Low Reactor Level       & \checkmark &            &            &            & \checkmark   \\
Sensor Failure (Any)    & \checkmark & \checkmark & \checkmark &            & \checkmark   \\
\hline
\end{tabular}
\label{tab:reactor_interlocks}
\end{table}
