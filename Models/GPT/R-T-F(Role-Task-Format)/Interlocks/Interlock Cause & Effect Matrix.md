\documentclass{article}
\usepackage{booktabs}
\usepackage{geometry}
\geometry{margin=1in}
\begin{document}

\section*{Cause and Effect Matrix for Chemical Reactor Interlocks}

\begin{tabular}{@{}lcccccc@{}}
\toprule
\textbf{Cause} & \textbf{Close Feed Valve} & \textbf{Stop Agitator} & \textbf{Open Relief Valve} & \textbf{Shutdown Heater} & \textbf{Activate Alarm} & \textbf{Emergency Shutdown (ESD)} \\
\midrule
High Pressure (>10 bar)       & X & X & X & X & X & X \\
Low Temperature (<60\,°C)     &   &   &   & X & X &   \\
Low Level in Reactor          & X &   &   & X & X &   \\
High Temperature (>120\,°C)   & X & X & X & X & X & X \\
Agitator Motor Fault          &   & X &   &   & X &   \\
Heater Overcurrent            &   &   &   & X & X &   \\
Cooling Water Flow Loss       & X & X & X & X & X & X \\
Pressure Transmitter Fault    & X &   &   &   & X &   \\
Level Transmitter Fault       & X &   &   &   & X &   \\
Emergency Stop Button Pressed & X & X & X & X & X & X \\
\bottomrule
\end{tabular}

\end{document}
