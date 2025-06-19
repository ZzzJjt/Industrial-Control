\usepackage{array}
\usepackage{amssymb}  % for \checkmark
\usepackage{longtable}

% Cause and Effect Matrix
\begin{table}[htbp]
\centering
\caption{Cause and Effect Matrix – Chemical Reactor Interlock Logic}
\renewcommand{\arraystretch}{1.3}
\begin{tabular}{|p{4.5cm}|c|c|c|c|c|c|}
\hline
\textbf{Cause (Sensor Condition)} & \textbf{Close Feed Valve} & \textbf{Stop Recirculation Pump} & \textbf{Start Emergency Cooling} & \textbf{Vent Reactor} & \textbf{Shutdown Heater} & \textbf{High-Priority Alarm} \\
\hline
High Reactor Pressure (≥ P\_high)     & \checkmark & \checkmark &        & \checkmark & \checkmark & \checkmark \\
Low Reactor Pressure (≤ P\_low)       & \checkmark &            &        &            &            & \checkmark \\
High Reactor Temperature (≥ T\_high)  & \checkmark & \checkmark & \checkmark &            & \checkmark & \checkmark \\
Low Reactor Temperature (≤ T\_low)    &            &            &        &            & \checkmark & \checkmark \\
High Reactor Level (≥ L\_high)        & \checkmark & \checkmark &        & \checkmark &            & \checkmark \\
Low Reactor Level (≤ L\_low)          & \checkmark &            &        &            & \checkmark & \checkmark \\
Cooling Water Flow Low               & \checkmark & \checkmark &        &            & \checkmark & \checkmark \\
Emergency Stop Activated              & \checkmark & \checkmark & \checkmark & \checkmark & \checkmark & \checkmark \\
PLC Communication Failure             & \checkmark & \checkmark & \checkmark & \checkmark & \checkmark & \checkmark \\
\hline
\end{tabular}
\label{tab:reactor-cause-effect}
\end{table}

\vspace{1em}

% Explanation Section
\noindent\textbf{Explanation of Interlock Effects:}
\begin{itemize}
  \item \textbf{Close Feed Valve:} Prevents further reactants from entering the reactor during abnormal states, reducing pressure or temperature escalation.
  \item \textbf{Stop Recirculation Pump:} Halts mixing or flow that may exacerbate unsafe reactor conditions, especially in thermal or pressure upsets.
  \item \textbf{Start Emergency Cooling:} Rapidly reduces reactor temperature using an independent cooling loop in response to overheating or loss of cooling water flow.
  \item \textbf{Vent Reactor:} Relieves internal pressure buildup to a flare or scrubber system to avoid rupture or explosion.
  \item \textbf{Shutdown Heater:} Prevents further heating when temperature or level signals indicate unsafe conditions or instrument faults.
  \item \textbf{High-Priority Alarm:} Notifies operators of urgent abnormal process conditions that require immediate manual response or acknowledgement.
\end{itemize}
