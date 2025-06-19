\documentclass[a4paper,12pt]{article}
\usepackage{geometry}
\geometry{margin=1in}
\usepackage{booktabs}
\usepackage{multirow}
\usepackage{amsmath}
\usepackage{noto}

\begin{document}

% Title
\title{Gas Turbine Safety Interlocks}
\author{Power Plant Safety Team}
\date{May 13, 2025}
\maketitle

% Introduction
\section*{Introduction}
This document outlines a comprehensive set of safety interlocks for the safe operation of a gas turbine in a power plant. Each interlock is defined with its trigger condition and corresponding safety action, designed to mitigate risks such as overheating, overspeed, and loss of lubrication. The interlocks are intended for implementation in a safety Programmable Logic Controller (PLC) or Distributed Control System (DCS) using IEC 61131-3 programming standards, ensuring fail-safe operation, redundancy, and operator visibility.

% Interlock Table
\section*{Interlock List}
\begin{table}[h]
\centering
\caption{Gas Turbine Safety Interlocks}
\small
\begin{tabular}{l|l|l}
\toprule
\textbf{Interlock} & \textbf{Trigger Condition} & \textbf{Safety Action} \\
\midrule
Overtemperature & Exhaust gas temperature > 650°C & Shut down turbine \\
Overspeed & Rotor speed > 105\% nominal & Trigger emergency stop \\
Overpressure & Combustion chamber pressure > 30 bar & Open pressure relief valve \\
Low Lubrication Pressure & Oil pressure < 1.5 bar & Stop turbine \\
High Vibration & Vibration > 10 mm/s & Shut down turbine \\
Flame Failure & Combustion flame lost & Stop fuel flow, raise alarm \\
Low Fuel Gas Pressure & Gas pressure < 2 bar & Close fuel valve, stop turbine \\
Low Cooling Water Flow & Water flow < 200 L/min & Shut down turbine \\
Compressor Surge & Surge event detected & Open bypass valve or reduce load \\
Emergency Stop & Manual override activated & Shut down turbine, isolate fuel \\
\bottomrule
\end{tabular}
\end{table}

% Integration and Implementation
\section*{Integration and Implementation}
The interlocks are implemented in a safety PLC or DCS using IEC 61131-3 Structured Text or Function Block Diagrams. Each interlock monitors real-time sensor inputs and triggers safety actions when thresholds are exceeded. Key implementation strategies include:

\begin{itemize}
    \item \textbf{Fail-Safe Design}: Interlocks are programmed to default to safe states (e.g., turbine shutdown, valve closed) on sensor failure or power loss. For example, the Low Lubrication Pressure interlock stops the turbine if oil pressure data is invalid.
    \item \textbf{Redundancy}: Critical sensors (e.g., temperature, speed) use redundant inputs with voting logic (e.g., 2-out-of-3) to prevent false trips or missed faults.
    \item \textbf{Hardwired and Programmed Logic}: The Emergency Stop interlock is hardwired for immediate response, while others are programmed for flexibility and diagnostics.
    \item \textbf{Alarms and HMI}: All interlocks trigger alarms on the Human-Machine Interface (HMI), displaying trigger conditions and actions for operator awareness. For example, Flame Failure raises a high-priority alarm alongside stopping fuel flow.
    \item \textbf{Testing and Maintenance}: Interlocks are tested during commissioning and regular maintenance intervals using simulated inputs to verify response times and actions.
\end{itemize}

% Safety Contributions
\section*{Safety Contributions}
The interlocks protect the gas turbine and ensure safe operation by:
\begin{itemize}
    \item \textbf{Preventing Equipment Damage}: Overtemperature, Overspeed, and High Vibration interlocks shut down the turbine to avoid thermal or mechanical failure.
    \item \textbf{Mitigating Hazardous Conditions}: Overpressure and Flame Failure interlocks prevent explosions or fires by relieving pressure or stopping fuel flow.
    \item \textbf{Ensuring Operational Stability}: Low Lubrication Pressure, Low Fuel Gas Pressure, and Low Cooling Water Flow interlocks stop the turbine to maintain critical systems, preventing cascading failures.
    \item \textbf{Responding to Dynamic Faults}: Compressor Surge interlock stabilizes operation by adjusting load or opening bypass valves, avoiding damage to compressor blades.
    \item \textbf{Providing Manual Control}: Emergency Stop interlock allows operators to instantly isolate the turbine during unforeseen events.
\end{itemize}

% Usage Guidelines
\section*{Usage Guidelines}
\begin{itemize}
    \item \textbf{PLC/DCS Implementation}: Program interlocks in Structured Text, e.g., \texttt{IF ExhaustTemp > 650.0 THEN TurbineShutdown := TRUE; END\_IF}. Use function blocks for modularity.
    \item \textbf{Sensor Calibration}: Ensure sensors (e.g., pressure, vibration) are calibrated and scaled correctly to match trigger thresholds.
    \item \textbf{HMI Integration}: Display interlock status, sensor values, and alarms on the HMI for real-time monitoring.
    \item \textbf{Compliance}: Align with safety standards (e.g., IEC 61511) and document interlocks for HAZOP and SIL assessments.
    \item \textbf{Operator Training}: Train operators on interlock triggers and responses to enhance situational awareness.
\end{itemize}

\end{document}
