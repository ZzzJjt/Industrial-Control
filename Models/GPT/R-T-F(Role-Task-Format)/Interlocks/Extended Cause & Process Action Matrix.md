\documentclass{article}
\usepackage{tabularx}
\usepackage{geometry}
\usepackage{booktabs}
\usepackage{array}
\usepackage{xcolor}

\geometry{margin=1in}
\renewcommand{\arraystretch}{1.5}

\begin{document}

\section*{Extended Cause–Action Matrix for Chemical Reactor Interlocks}

\begin{tabularx}{\textwidth}{>{\raggedright\arraybackslash}p{5.5cm} *{6}{>{\centering\arraybackslash}X}}
\toprule
\textbf{Cause \textbackslash\ Action} &
\textbf{Isolate Feed} &
\textbf{Stop Agitator} &
\textbf{Open Relief Valve} &
\textbf{Shutdown Heater} &
\textbf{Activate Alarm} &
\textbf{Emergency Shutdown (ESD)} \\
\midrule
High Pressure (>10~bar) & X & X & X & X & X & X \\
Low Temperature (<60\,\textdegree C) &  &  &  & X & X & \\
Sensor Fault – Pressure Transmitter & X &  &  &  & X & \\
Sensor Fault – Temperature Transmitter & X &  &  &  & X & \\
Heater Overcurrent &  &  &  & X & X & X \\
Agitator Motor Fault &  & X &  &  & X & \\
Cooling Failure & X & X & X & X & X & X \\
Manual ESD Pushbutton Activated & X & X & X & X & X & X \\
Reactor Level Too High & X &  &  &  & X & \\
Relief Valve Feedback Fault &  &  &  &  & X & \\
\bottomrule
\end{tabularx}

\vspace{1em}
\noindent
\textbf{Legend:} X = Action is triggered by the corresponding cause

\end{document}
