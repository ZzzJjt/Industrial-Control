\documentclass{article}
\usepackage{amsmath}
\usepackage{listings}
\usepackage{geometry}
\geometry{margin=1in}
\title{IEC 61131-3 SplineInterpolator Function Block}
\begin{document}

\maketitle

\section*{Function Block Interface}
\textbf{Name:} \texttt{SplineInterpolator}

\begin{lstlisting}[language=Pascal, basicstyle=\ttfamily\small]
FUNCTION_BLOCK SplineInterpolator
VAR_INPUT
    X         : ARRAY[1..N] OF REAL;       // Known x-values
    Y         : ARRAY[1..N] OF REAL;       // Corresponding y-values
    CoefA     : ARRAY[1..N-1] OF REAL;     // Spline coefficients a
    CoefB     : ARRAY[1..N-1] OF REAL;     // Spline coefficients b
    CoefC     : ARRAY[1..N-1] OF REAL;     // Spline coefficients c
    CoefD     : ARRAY[1..N-1] OF REAL;     // Spline coefficients d
    TargetX   : REAL;                      // X to interpolate
END_VAR
VAR_OUTPUT
    InterpolatedY : REAL;                 // Resulting interpolated Y
    Valid         : BOOL;                 // True if interpolation succeeded
END_VAR
VAR
    i : INT;
    dx : REAL;
END_VAR
\end{lstlisting}

\section*{Interpolation Logic}

\begin{enumerate}
  \item Loop to find interval $i$ such that $X[i] \leq \texttt{TargetX} < X[i+1]$
  \item Evaluate:
  \[
  \texttt{dx} = \texttt{TargetX} - X[i]
  \]
  \[
  \texttt{InterpolatedY} = \texttt{CoefA[i]} + \texttt{CoefB[i]} \cdot dx + \texttt{CoefC[i]} \cdot dx^2 + \texttt{CoefD[i]} \cdot dx^3
  \]
  \item Set \texttt{Valid := TRUE}; else clamp or flag if out of range
\end{enumerate}

\section*{Edge Case Handling}
\begin{itemize}
  \item Clamp \texttt{TargetX} to within \texttt{X[1]} and \texttt{X[N]} to avoid access errors.
  \item Require $N \geq 3$ for spline interpolation.
\end{itemize}

\section*{Performance Considerations}
\begin{itemize}
  \item Precompute \texttt{CoefA, B, C, D} offline or during initialization.
  \item One-pass evaluation: only one segment is computed per scan.
  \item No dynamic memory; constant loop bounds ensure real-time predictability.
\end{itemize}

\end{document}
