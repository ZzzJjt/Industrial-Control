\documentclass[a4paper,12pt]{article}
\usepackage{geometry}
\geometry{margin=1in}
\usepackage{booktabs}
\usepackage{multirow}
\usepackage{amsmath}
\usepackage{noto}

\begin{document}

% Title
\title{Ammonium Nitrate Prilling Station Safety Interlocks}
\author{Prilling Station Safety Team}
\date{May 13, 2025}
\maketitle

% Introduction
\section*{Introduction}
This document outlines a comprehensive set of safety interlocks for an ammonium nitrate prilling station, addressing critical safety and operational conditions due to the explosive nature of the material. Each interlock specifies a trigger condition and corresponding safety action to prevent hazardous events such as overpressure, overheating, or equipment malfunctions. The interlocks are designed for implementation in a Programmable Logic Controller (PLC) or Distributed Control System (DCS) using IEC 61131-3 Structured Text or Function Block Diagrams, ensuring fail-safe operation, latching logic, and compliance with standards like OSHA, ATEX, and NFPA.

% Interlock Table
\section*{Interlock List}
\begin{table}[h]
\centering
\caption{Ammonium Nitrate Prilling Station Safety Interlocks}
\small
\begin{tabular}{l|l|l}
\toprule
\textbf{Interlock} & \textbf{Trigger Condition} & \textbf{Safety Action} \\
\midrule
Overtemperature & Prill tower temperature > 180°C & Shut down prill tower \\
High Pressure & Process pressure > 10 bar & Open vent valve, isolate equipment \\
Cooling Air Flow Loss & Airflow < 5000 m³/h & Stop melt feed \\
Ammonium Nitrate Level High & Prill bucket level > 90\% & Halt production \\
Melt Pump Failure & Melt pump flow < 100 L/min & Shut down prilling head \\
Scrubber Failure & Scrubber pressure drop < 50 Pa & Stop prill tower \\
Emergency Stop & Manual E-stop activated & Shut down all major systems \\
Power Failure & Loss of electrical power & Transition to safe state (valves closed, systems off) \\
Explosion Vent Activation & Explosion vent opened & Trigger alarm, isolate upstream systems \\
Inert Gas Loss & Nitrogen pressure < 0.5 bar & Shut down sensitive zones \\
\bottomrule
\end{tabular}
\end{table}

% Integration and Implementation
\section*{Integration and Implementation}
The interlocks are implemented in a safety-rated PLC or DCS using IEC 61131-3 Structured Text or Function Block Diagrams. Key implementation strategies include:

\begin{itemize}
    \item \textbf{Fail-Safe Design}: Interlocks default to safe states (e.g., valves closed, systems off) on sensor failure, power loss, or invalid data. For example, the Power Failure interlock ensures all critical valves close automatically.
    \item \textbf{Latching Logic}: Interlocks like Overtemperature and Ammonium Nitrate Level High use latching to maintain safe states until manual reset, preventing automatic restarts after faults.
    \item \textbf{Redundancy}: Critical sensors (e.g., temperature, pressure) employ redundant inputs with 2-out-of-3 voting logic to avoid false trips or missed faults.
    \item \textbf{HMI and Alarms}: Each interlock triggers a high-priority alarm on the Human-Machine Interface (HMI), displaying trigger conditions and actions. For example, Explosion Vent Activation logs the event and alerts operators immediately.
    \item \textbf{Testing and Validation}: Interlocks are tested during commissioning and periodic maintenance using simulated faults (e.g., setting temperature > 180°C) to verify response times and actions.
    \item \textbf{Documentation}: Interlock logic, thresholds, and actions are documented for audits and compliance with OSHA, ATEX, and NFPA standards.
\end{itemize}

% Safety Contributions
\section*{Safety Contributions}
The interlocks form a layered safety approach, protecting personnel, equipment, and the environment:

\begin{itemize}
    \item \textbf{Preventing Explosions and Fires}: Overtemperature, High Pressure, and Explosion Vent Activation interlocks mitigate thermal and pressure-related risks, critical for ammonium nitrate’s explosive potential.
    \item \textbf{Ensuring Operational Stability}: Cooling Air Flow Loss, Melt Pump Failure, and Scrubber Failure interlocks stop operations during critical system failures, preventing cascading issues.
    \item \textbf{Protecting Against Overfilling}: Ammonium Nitrate Level High interlock halts production to avoid bucket overflow, reducing spillage and equipment strain.
    \item \textbf{Environmental Compliance}: Scrubber Failure and Explosion Vent Activation interlocks ensure emissions are controlled, meeting environmental regulations.
    \item \textbf{Manual and Automatic Safeguards}: Emergency Stop and Power Failure interlocks provide immediate shutdown capabilities, while Inert Gas Loss protects sensitive zones from oxygen exposure.
\end{itemize}

% Usage Guidelines
\section*{Usage Guidelines}
\begin{itemize}
    \item \textbf{PLC/DCS Implementation}: Program interlocks in Structured Text, e.g., \texttt{IF TowerTemp > 180.0 THEN PrillTowerShutdown := TRUE; END\_IF}. Use function blocks for modularity and reusability.
    \item \textbf{Sensor Calibration}: Calibrate sensors (e.g., temperature to °C, pressure to bar) to match trigger thresholds, ensuring accuracy in harsh prilling environments.
    \item \textbf{HMI Integration}: Display interlock statuses, sensor values, and alarms on the HMI. Log events for post-incident analysis and regulatory reporting.
    \item \textbf{Compliance}: Align with OSHA 1910, ATEX Directive 2014/34/EU, and NFPA 495 for handling energetic materials, documenting interlocks for safety audits.
    \item \textbf{Operator Training}: Train operators on interlock triggers, responses, and manual reset procedures to enhance safety awareness.
    \item \textbf{Periodic Testing}: Conduct regular tests under simulated fault conditions to validate interlock functionality and response times.
\end{itemize}

\end{document}
