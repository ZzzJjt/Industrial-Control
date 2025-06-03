\begin{table}[htbp]
\centering
\caption{Feedwater Control I/O List – Steam-Water Cycle}
\scriptsize
\begin{tabular}{|l|l|c|l|l|l|l|}
\hline
\textbf{Signal Name}            & \textbf{Signal Tag Number} & \textbf{A/D} & \textbf{Engineering Unit} & \textbf{Range} & \textbf{Setpoint} & \textbf{P\&ID Ref} \\
\hline
Feedwater Flow Transmitter      & FT-201                      & Analog       & kg/h                      & 0–100000       & 75000             & P\&ID-201 \\
Drum Level Transmitter          & LT-202                      & Analog       & \%                        & 0–100          & 50                & P\&ID-202 \\
Feedwater Control Valve         & CV-203                      & Analog       & \%                        & 0–100          & 65                & P\&ID-203 \\
Boiler Drum Pressure Transmitter& PT-204                      & Analog       & bar                       & 0–160          & 130               & P\&ID-204 \\
Feed Pump Status                & DI-205                      & Digital      & –                         & ON/OFF         & –                 & P\&ID-205 \\
Pump Start Command              & DO-206                      & Digital      & –                         & ON/OFF         & ON                & P\&ID-206 \\
Pump Speed Feedback             & AI-207                      & Analog       & RPM                       & 0–3000         & 2400              & P\&ID-207 \\
Pump Speed Command              & AO-208                      & Analog       & RPM                       & 0–3000         & 2500              & P\&ID-208 \\
Low Drum Level Alarm            & AL-209                      & Digital      & –                         & ON/OFF         & –                 & P\&ID-209 \\
High Drum Level Alarm           & AL-210                      & Digital      & –                         & ON/OFF         & –                 & P\&ID-210 \\
Make-up Water Valve             & CV-211                      & Analog       & \%                        & 0–100          & 50                & P\&ID-211 \\
Make-up Water Flow Transmitter & FT-212                      & Analog       & kg/h                      & 0–10000        & 4000              & P\&ID-212 \\
Economizer Inlet Temp Transmitter & TT-213                    & Analog       & °C                        & 0–250          & 220               & P\&ID-213 \\
Economizer Outlet Temp Transmitter & TT-214                  & Analog       & °C                        & 0–300          & 270               & P\&ID-214 \\
Condensate Tank Level          & LT-215                      & Analog       & \%                        & 0–100          & 80                & P\&ID-215 \\
Tank Low Level Alarm           & AL-216                      & Digital      & –                         & ON/OFF         & –                 & P\&ID-216 \\
Feed Line Pressure Transmitter & PT-217                      & Analog       & bar                       & 0–100          & 60                & P\&ID-217 \\
Feedwater Isolation Valve      & CV-218                      & Digital      & –                         & OPEN/CLOSE     & OPEN              & P\&ID-218 \\
Control Valve Position Feedback& PI-219                      & Analog       & \%                        & 0–100          & –                 & P\&ID-219 \\
Feed Pump Overload Alarm       & AL-220                      & Digital      & –                         & ON/OFF         & –                 & P\&ID-220 \\
\hline
\end{tabular}
\end{table}
