\documentclass{article}
\usepackage{geometry}
\usepackage{longtable}
\usepackage{booktabs}
\geometry{margin=1in}

\title{Interlock System for Ammonium Nitrate Prilling Station}
\date{}
\begin{document}

\maketitle

\section*{Overview}
To ensure safe operation of an ammonium nitrate prilling station, a series of interlocks are implemented. Each interlock is designed to detect abnormal or unsafe conditions and trigger predefined automated actions to protect personnel, equipment, and the environment.

\section*{Interlock Matrix}

\begin{longtable}{@{}llp{7cm}@{}}
\toprule
\textbf{Interlock Name} & \textbf{Trigger Condition} & \textbf{Automated Response} \\
\midrule
Overtemperature Interlock & Prill tower temp > safety limit & Shut down prill tower, latch shutdown, raise alarm \\
High Pressure Interlock & Melt system pressure > 12 bar & Open relief valve, stop melt pump, latch shutdown \\
Low Cooling Air Flow Interlock & Airflow < minimum threshold & Halt melt feed, disable prilling head, raise alarm \\
High Product Level Interlock & Prill level > high limit & Stop production feed, alert operator \\
Pump/Feeder Failure Interlock & Pump/feeder status = fault or flow = 0 & Stop prilling head, hold system state, latch fault \\
Scrubber Failure Interlock & Scrubber feedback = failed & Shut off emissions source, isolate tower exhaust \\
Emergency Stop Interlock & Manual E-stop pressed & Immediate shutdown of entire system, inhibit all I/O \\
Inerting Gas Loss Interlock & Inert gas pressure < minimum & Stop melt feed, disable process zones needing inerting \\
\bottomrule
\end{longtable}

\section*{Reset Conditions}
All interlocks are latched until the following criteria are met:
\begin{itemize}
    \item The triggering fault is cleared (e.g., airflow restored, pressure normalized).
    \item Manual reset is issued by an authorized operator.
    \item All sensors report stable and valid conditions.
\end{itemize}

\section*{Fail-Safe Design Considerations}
\begin{itemize}
    \item Any sensor failure defaults to safe action (e.g., stop, close, alarm).
    \item Valve and pump feedbacks are verified to confirm actuation.
    \item Control logic is periodically tested and validated.
\end{itemize}

\section*{Use Case Example}
If the cooling air fan fails, the system:
\begin{enumerate}
    \item Detects airflow < threshold.
    \item Shuts down the melt feed and disables the prilling head.
    \item Latches the interlock and issues an operator alarm.
    \item Requires manual reset after air flow is confirmed.
\end{enumerate}

\end{document}
