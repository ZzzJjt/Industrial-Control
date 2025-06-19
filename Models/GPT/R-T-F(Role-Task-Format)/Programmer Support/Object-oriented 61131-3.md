\documentclass[11pt]{article}
\usepackage[a4paper,margin=1in]{geometry}
\usepackage{booktabs}
\usepackage{hyperref}
\usepackage{longtable}
\usepackage{enumitem}
\usepackage{titlesec}

\titleformat{\section}{\normalfont\Large\bfseries}{\thesection}{1em}{}

\title{\textbf{OSCAT Mathematical Function Blocks Reference Guide}}
\author{For IEC 61131-3 Programmers}
\date{}

\begin{document}

\maketitle

\section*{Introduction}

This reference guide lists mathematical function blocks available in the \textbf{OSCAT} (Open Source Community for Automation Technology) library, organized by category. Each entry includes the function name, a description, typical industrial use cases, and any constraints relevant to input type or range.

\section*{Trigonometric Functions}

\begin{longtable}{@{}p{3.5cm}p{5.5cm}p{5cm}@{}}
\toprule
\textbf{Function} & \textbf{Description} & \textbf{Typical Use Case} \\
\midrule
\texttt{SIN(x)} & Sine of angle in radians & Signal generation, motor control \\
\texttt{COS(x)} & Cosine of angle & Phase angle computation \\
\texttt{TAN(x)} & Tangent of angle & Flow modeling, rotary mechanics \\
\texttt{ASIN(x)} & Inverse sine, domain: $[-1,1]$ & Signal decoding, position control \\
\texttt{ACOS(x)} & Inverse cosine, domain: $[-1,1]$ & Phase reconstruction \\
\texttt{ATAN(x)} & Inverse tangent & Direction control \\
\texttt{ATAN2(y, x)} & Arctangent using x and y & 2D vector orientation \\
\bottomrule
\end{longtable}

\section*{Algebraic Functions}

\begin{longtable}{@{}p{3.5cm}p{5.5cm}p{5cm}@{}}
\toprule
\textbf{Function} & \textbf{Description} & \textbf{Typical Use Case} \\
\midrule
\texttt{ABS(x)} & Absolute value of x & Signal normalization \\
\texttt{SIGN(x)} & Sign of x (-1, 0, 1) & Directional logic \\
\texttt{MIN(x, y)} & Minimum of two values & Lower limit enforcement \\
\texttt{MAX(x, y)} & Maximum of two values & Overrange protection \\
\texttt{LIMIT(x, lo, hi)} & Clamp x within [lo, hi] & Safety bounding of control signals \\
\texttt{ROUND(x)} & Round to nearest integer & Integer actuator commands \\
\texttt{TRUNC(x)} & Truncate decimal part & Discrete process modeling \\
\texttt{MOD(x, y)} & Modulo remainder of x/y & Periodic control logic \\
\bottomrule
\end{longtable}

\section*{Statistical Functions}

\begin{longtable}{@{}p{3.5cm}p{5.5cm}p{5cm}@{}}
\toprule
\textbf{Function} & \textbf{Description} & \textbf{Typical Use Case} \\
\midrule
\texttt{MEAN\_ARRAY(arr)} & Mean value of array elements & Noise reduction, trend smoothing \\
\texttt{STDDEV\_ARRAY(arr)} & Standard deviation of array & Quality monitoring \\
\texttt{MIN\_ARRAY(arr)} & Smallest value in array & Alarm/limit detection \\
\texttt{MAX\_ARRAY(arr)} & Largest value in array & Peak detection \\
\bottomrule
\end{longtable}

\section*{Exponential and Logarithmic Functions}

\begin{longtable}{@{}p{3.5cm}p{5.5cm}p{5cm}@{}}
\toprule
\textbf{Function} & \textbf{Description} & \textbf{Typical Use Case} \\
\midrule
\texttt{EXP(x)} & $e^x$ exponential function & Process growth/decay modeling \\
\texttt{LN(x)} & Natural logarithm & Pressure/pH computation \\
\texttt{LOG(x, b)} & Log base \texttt{b} of x & Custom scaling transformations \\
\texttt{POWER(x, y)} & $x^y$ power function & Nonlinear gain applications \\
\bottomrule
\end{longtable}

\section*{Array Operations}

\begin{longtable}{@{}p{3.5cm}p{5.5cm}p{5cm}@{}}
\toprule
\textbf{Function} & \textbf{Description} & \textbf{Typical Use Case} \\
\midrule
\texttt{SUM\_ARRAY(arr)} & Sum of all array values & Mass/flow totalization \\
\texttt{AVG\_FILTER(arr)} & Moving average filter & Real-time signal smoothing \\
\texttt{DIFF\_ARRAY(a1, a2)} & Subtract arrays element-wise & Vector error analysis \\
\texttt{SCALE\_ARRAY(arr, k)} & Multiply all elements by factor $k$ & Calibration or gain scaling \\
\bottomrule
\end{longtable}

\end{document}
