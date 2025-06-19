\documentclass[a4paper,12pt]{article}
\usepackage{geometry}
\geometry{margin=1in}
\usepackage{booktabs}
\usepackage{multirow}
\usepackage{amsmath}
\usepackage{noto}

\begin{document}

% Title
\title{Chemical Reactor Interlock Matrix}
\author{Process Safety Team}
\date{May 12, 2025}
\maketitle

% Introduction
\section*{Introduction}
This document presents an extended cause and process action matrix for the interlock system of a chemical reactor. The matrix maps hazardous conditions (causes) to safety actions, ensuring comprehensive coverage of critical risks such as overpressure, overheating, and equipment failure. Each row represents a cause, and each column represents a safety action, with an 'X' indicating activation of the action for the corresponding cause.

% Matrix
\section*{Cause and Process Action Matrix}
\begin{table}[h]
\centering
\caption{Chemical Reactor Interlock Matrix}
\small
\begin{tabular}{l|cccccccc}
\toprule
\textbf{Cause} & \textbf{Close Feed Valve} & \textbf{Open Vent Valve} & \textbf{Activate Cooling} & \textbf{Stop Agitator} & \textbf{Open Drain Valve} & \textbf{Shut Down Reactor} & \textbf{Trigger Alarm} & \textbf{Activate Emergency Stop} \\
\midrule
High Pressure (>100 barg) & X & X & & & & X & X & \\
Low Pressure (<5 barg) & X & & & & & & X & \\
High Temperature (>200°C) & X & X & X & & & X & X & \\
Low Temperature (<50°C) & X & & & & & & X & \\
High Level (>90\%) & X & & & X & X & X & X & \\
Low Level (<10\%) & X & & & X & & & X & \\
Level Sensor Fault & X & & & X & & X & X & X \\
Flow Anomaly (Feed >500 L/min) & X & & & & & X & X & \\
Temperature Sensor Fault & X & X & X & & & X & X & X \\
Pressure Sensor Fault & X & X & & & & X & X & X \\
\bottomrule
\end{tabular}
\end{table}

% Analysis
\section*{Analysis}
The matrix was analyzed to ensure comprehensive safety coverage and optimize interlock logic:

\begin{itemize}
    \item \textbf{Coverage Validation}: Each cause triggers at least one safety action, ensuring no hazardous condition is unaddressed. For example, "High Pressure" activates multiple actions (Close Feed Valve, Open Vent Valve, Shut Down Reactor, Trigger Alarm) to mitigate risk.
    \item \textbf{Redundancy Check}: Some causes (e.g., sensor faults) trigger overlapping actions (e.g., Shut Down Reactor and Trigger Alarm). This redundancy is intentional to enhance safety but can be reviewed to prioritize critical actions if response time is a concern.
    \item \textbf{Appropriateness}: Actions are tailored to the cause. For instance, "High Temperature" activates cooling and venting, while "Low Level" stops the agitator to prevent mechanical damage.
    \item \textbf{Optimization Opportunities}: The "Trigger Alarm" action is used in all cases, which may lead to operator overload. Consider prioritizing alarms for critical conditions (e.g., sensor faults, high pressure) and using secondary notifications for less urgent issues (e.g., low temperature).
\end{itemize}

% Usage Guidelines
\section*{Usage Guidelines}
\begin{itemize}
    \item \textbf{Integration}: Implement the matrix in the PLC using IEC 61131-3 logic (e.g., Structured Text) to map sensor inputs to actuator outputs based on the matrix.
    \item \textbf{Operator Training}: Use the matrix as a training tool to familiarize operators with interlock responses.
    \item \textbf{Review and Updates}: Regularly review the matrix during safety audits to incorporate new hazards or process changes.
    \item \textbf{Logging}: Log all triggered actions and causes for post-incident analysis and compliance reporting.
\end{itemize}

\end{document}
