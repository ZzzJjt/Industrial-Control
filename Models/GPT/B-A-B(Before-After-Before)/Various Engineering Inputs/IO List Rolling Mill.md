\begin{table}[htbp]
\centering
\caption{Rolling Mill I/O List}
\scriptsize
\begin{tabular}{|l|c|l|c|l|l|l|l|}
\hline
\textbf{Signal Name} & \textbf{I/O} & \textbf{Tag Number} & \textbf{A/D} & \textbf{Engineering Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\
\hline
Entry Tension         & Input  & TE-101  & Analog  & kN       & 0–200       & 120     & P\&ID-01 \\
Roll Gap              & Output & MV-102  & Analog  & mm       & 0–10        & 4.5     & P\&ID-02 \\
Mill Speed Feedback   & Input  & FE-103  & Analog  & m/min    & 0–1000      & 600     & P\&ID-03 \\
Main Drive Current    & Input  & AE-104  & Analog  & A        & 0–500       & –       & P\&ID-04 \\
Hydraulic Pressure    & Input  & PE-105  & Analog  & bar      & 0–250       & 160     & P\&ID-05 \\
Work Roll Temp        & Input  & TE-106  & Analog  & °C       & 0–300       & 180     & P\&ID-06 \\
Cooling Valve Command & Output & CV-107  & Analog  & \%       & 0–100       & 65      & P\&ID-07 \\
Entry Loop Height     & Input  & LE-108  & Analog  & mm       & 0–1000      & 500     & P\&ID-08 \\
Exit Tension          & Input  & TE-109  & Analog  & kN       & 0–200       & 130     & P\&ID-09 \\
Roll Force            & Input  & FE-110  & Analog  & kN       & 0–2000      & 1500    & P\&ID-10 \\
Work Roll Vibration   & Input  & VE-111  & Analog  & mm/s     & 0–50        & –       & P\&ID-11 \\
Motor Overload Trip   & Input  & DI-112  & Digital & –        & ON/OFF      & –       & P\&ID-12 \\
Oil Pump Status       & Input  & DI-113  & Digital & –        & ON/OFF      & –       & P\&ID-13 \\
Emergency Stop        & Input  & DI-114  & Digital & –        & ON/OFF      & –       & P\&ID-14 \\
Lubrication OK        & Input  & DI-115  & Digital & –        & ON/OFF      & –       & P\&ID-15 \\
Entry Guide Position  & Input  & PE-116  & Analog  & mm       & 0–100       & 60      & P\&ID-16 \\
Exit Guide Position   & Input  & PE-117  & Analog  & mm       & 0–100       & 65      & P\&ID-17 \\
Screwdown Position    & Output & MV-118  & Analog  & mm       & 0–200       & 125     & P\&ID-18 \\
Roll Chock Status     & Input  & DI-119  & Digital & –        & OPEN/CLOSE  & –       & P\&ID-19 \\
Strip Width Sensor    & Input  & WE-120  & Analog  & mm       & 600–1600    & 1200    & P\&ID-20 \\
Water Flowrate        & Input  & FE-121  & Analog  & L/min    & 0–500       & 250     & P\&ID-21 \\
Roll Cool Temp        & Input  & TE-122  & Analog  & °C       & 0–100       & 55      & P\&ID-22 \\
Exit Loop Height      & Input  & LE-123  & Analog  & mm       & 0–1000      & 480     & P\&ID-23 \\
Mill Start Command    & Output & DO-124  & Digital & –        & ON/OFF      & ON      & P\&ID-24 \\
Mill Stop Command     & Output & DO-125  & Digital & –        & ON/OFF      & OFF     & P\&ID-25 \\
Drive Ready Feedback  & Input  & DI-126  & Digital & –        & ON/OFF      & –       & P\&ID-26 \\
Tension Setpoint      & Output & AO-127  & Analog  & kN       & 0–200       & 120     & P\&ID-27 \\
Roll Speed Command    & Output & AO-128  & Analog  & m/min    & 0–1000      & 650     & P\&ID-28 \\
Coiler Status         & Input  & DI-129  & Digital & –        & ON/OFF      & –       & P\&ID-29 \\
Hydraulic Tank Level  & Input  & LE-130  & Analog  & \%       & 0–100       & 80      & P\&ID-30 \\
\hline
\end{tabular}
\end{table}
