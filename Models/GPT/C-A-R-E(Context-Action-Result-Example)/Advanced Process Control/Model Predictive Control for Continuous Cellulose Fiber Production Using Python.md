\documentclass{article}
\usepackage{booktabs}
\usepackage{geometry}
\geometry{a4paper, margin=1in}

\begin{document}

\section*{MPC Simulation Results for Continuous Cellulose Fiber Production}

\begin{table}[h!]
\centering
\begin{tabular}{cccc}
\toprule
\textbf{Time (hr)} & \textbf{Demand (tons/hr)} & \textbf{Inflow (tons/hr)} & \textbf{Tank Level (m\textsuperscript{3})} \\
\midrule
0   & 45 & 50 & 500 \\
1   & 48 & 50 & 502 \\
2   & 53 & 50 & 499 \\
3   & 55 & 52 & 497 \\
4   & 60 & 55 & 492 \\
5   & 65 & 60 & 487 \\
6   & 62 & 62 & 485 \\
7   & 58 & 60 & 487 \\
8   & 52 & 55 & 490 \\
9   & 47 & 50 & 493 \\
10  & 45 & 50 & 495 \\
\bottomrule
\end{tabular}
\caption{Model Predictive Control (MPC) Simulation Results showing demand tracking and tank level stability under variable outflow.}
\end{table}

\end{document}
