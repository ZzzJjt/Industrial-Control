\documentclass{article}
\usepackage{geometry}
\usepackage{booktabs}
\usepackage{longtable}
\usepackage{pdflscape}
\usepackage{multirow}
\usepackage{array}
\usepackage{xcolor}

\geometry{a4paper, margin=1in}
\setlength{\extrarowheight}{1pt}

\begin{document}

\section*{Extended Cause and Process Action Matrix for Chemical Reactor Interlock System}

\begin{landscape}
\begin{longtable}{>{\raggedright\arraybackslash}p{5cm} | *{7}{>{\centering\arraybackslash}p{3cm}}}
\toprule
\textbf{Cause / Fault Condition} &
\textbf{Close Feed Valve} &
\textbf{Close Product Valve} &
\textbf{Start Emergency Cooling} &
\textbf{Stop Agitator} &
\textbf{Shut Down Heater} &
\textbf{Trigger Alarm} &
\textbf{Emergency Shutdown (ESD)} \\
\midrule
\endfirsthead

\multicolumn{8}{c}{{\textit{Continued from previous page}}} \\
\toprule
\textbf{Cause / Fault Condition} &
\textbf{Close Feed Valve} &
\textbf{Close Product Valve} &
\textbf{Start Emergency Cooling} &
\textbf{Stop Agitator} &
\textbf{Shut Down Heater} &
\textbf{Trigger Alarm} &
\textbf{Emergency Shutdown (ESD)} \\
\midrule
\endhead

\midrule
\multicolumn{8}{r}{{\textit{Continued on next page}}} \\
\endfoot

\bottomrule
\endlastfoot

Pressure > Max Limit &
\cellcolor{gray!20}✓ & 
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
 & 
\cellcolor{gray!20}✓ & 
\cellcolor{gray!20}✓ & 
\cellcolor{gray!20}✓ \\

Temperature > Max Limit &
\cellcolor{gray!20}✓ &
 & 
\cellcolor{gray!20}✓ &
 & 
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Level > High High &
\cellcolor{gray!20}✓ &
 & 
 & 
 & 
 & 
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Reactor Agitator Failure &
\cellcolor{gray!20}✓ &
 & 
 & 
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
 \\

Sensor Fault (Pressure, Temp, Level) &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
 & 
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Loss of Cooling Media &
\cellcolor{gray!20}✓ &
 & 
 & 
 & 
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Uncontrolled Reaction Rate &
\cellcolor{gray!20}✓ &
 & 
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Power Failure Detected &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

\end{longtable}
\end{landscape}

\section*{Fail-Safe and Detection Logic Notes}
\begin{itemize}
  \item \textbf{Sensor Failures}: Any sensor returning invalid data triggers fallback mode and ESD.
  \item \textbf{Failsafe Design}: All valves are designed to fail in closed position. Heater and agitator default to OFF.
  \item \textbf{Auditing and Traceability}: All interlocks and responses are logged with timestamps and cause codes for post-event analysis.
\end{itemize}

\end{document}
