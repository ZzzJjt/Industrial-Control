\documentclass[a4paper,12pt]{article}
\usepackage{geometry}
\usepackage{booktabs}
\usepackage{longtable}
\usepackage{lscape}
\usepackage{multirow}
\usepackage{xcolor}
\usepackage{enumitem}
\geometry{margin=1in}
\setlength{\extrarowheight}{1pt}

\begin{document}

\title{Cause and Effect Interlock Matrix for Chemical Reactor Safety}
\author{}
\date{}
\maketitle

\section*{1. Cause and Effect Matrix}

\begin{landscape}
\begin{longtable}{>{\raggedright\arraybackslash}p{6cm} | *{6}{>{\centering\arraybackslash}p{3.5cm}}}
\toprule
\textbf{Cause (Abnormal Condition)} &
\textbf{Close Feed Valve} &
\textbf{Shut Down Heater} &
\textbf{Stop Agitator} &
\textbf{Isolate Cooling Line} &
\textbf{Trigger Alarm} &
\textbf{Emergency Shutdown (ESD)} \\
\midrule
\endfirsthead

\multicolumn{7}{c}{\textit{Continued from previous page}} \\
\toprule
\textbf{Cause (Abnormal Condition)} &
\textbf{Close Feed Valve} &
\textbf{Shut Down Heater} &
\textbf{Stop Agitator} &
\textbf{Isolate Cooling Line} &
\textbf{Trigger Alarm} &
\textbf{Emergency Shutdown (ESD)} \\
\midrule
\endhead

\midrule
\multicolumn{7}{r}{\textit{Continued on next page}} \\
\endfoot

\bottomrule
\endlastfoot

Reactor Pressure > High Limit &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
 &
 &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Reactor Temperature > High Limit &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
 &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Low Reactor Level (Dry Run Risk) &
\cellcolor{gray!20}✓ &
 &
\cellcolor{gray!20}✓ &
 &
\cellcolor{gray!20}✓ &
 \\

Cooling Line Flow = 0 &
 &
\cellcolor{gray!20}✓ &
 &
 &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Sensor Failure (Temp or Pressure) &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
 &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

Unexpected Reaction Rate Spike &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ &
\cellcolor{gray!20}✓ \\

\end{longtable}
\end{landscape}

\section*{2. Explanation of Interlock Matrix Logic}

\subsection*{Purpose and Function}

The cause and effect matrix provides a structured overview of how specific process deviations in the chemical reactor automatically trigger predefined safety actions. These actions are executed through PLC logic, valves, actuators, or alarms to mitigate hazardous conditions and protect both equipment and personnel.

\subsection*{Logic Flow}

\begin{itemize}[leftmargin=2em]
  \item Each row corresponds to a possible abnormal event (cause), detected via sensors or derived logic.
  \item Each column defines a potential response (effect) that the interlock system can execute.
  \item A marked cell (✓) shows the link between a cause and the action it triggers.
\end{itemize}

\subsection*{Safety Benefits}

\begin{itemize}[leftmargin=2em]
  \item \textbf{Fast Automated Response:} Immediate execution of interlocks prevents escalation of abnormal conditions.
  \item \textbf{Mitigation by Action:} 
  \begin{itemize}
    \item Shutting the feed valve prevents overpressure and runaway reactions.
    \item Stopping the heater and agitator halts reaction progression.
    \item Isolating cooling helps detect loss-of-cooling conditions and prevent false recovery.
    \item Alarms and ESD notify operators and initiate full plant protection if required.
  \end{itemize}
  \item \textbf{Redundancy and Coverage:} Sensor faults themselves trigger protective shutdowns, ensuring fail-safe behavior.
\end{itemize}

\subsection*{Importance of Completeness}

A complete and clearly documented matrix:
\begin{itemize}[leftmargin=2em]
  \item Serves as a reference during HAZOP reviews, safety audits, and system validation.
  \item Reduces ambiguity for operators and developers alike.
  \item Forms the basis for implementing PLC logic, loop drawings, and alarm handling sequences.
\end{itemize}

\section*{3. Conclusion}

This interlock matrix forms a critical part of the overall safety instrumented system (SIS) for the reactor. It ensures that for every potential unsafe condition, the system has a programmed, traceable, and testable response—supporting plant uptime, regulatory compliance, and most importantly, safe operation.

\end{document}
