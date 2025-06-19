(* IEC 61131-3 ASCII Function Block Diagram for Control Logic *)
(* Includes Analog Input, TON Timer, PID Control, and Analog Output *)
(* Signal Flow: Analog Input --> TON and PID --> Analog Output *)
(* Designed for design documentation and peer review *)

                  +-----------------+
                  |  Analog Input   |
                  |      IN        |
                  |     OUT: PV    |
                  +-----------------+
                           |
                           | PV
                           |
                           v
        +------------------+------------------+
        |                                     |
        | PV                                  | PV
        v                                     v
+-----------------+                  +-----------------+
|    TON Timer    |                  |    PID Control  |
|     IN: PV      |                  |     PV: PV      |
|    PT: T#5s     |                  |     SP: 100.0   |
|    ET: Elapsed  |                  |     CV: Output  |
|    Q: Done      |                  |                 |
+-----------------+                  +-----------------+
                                            |
                                            | CV
                                            v
                                    +-----------------+
                                    |  Analog Output  |
                                    |     IN: CV      |
                                    |     OUT         |
                                    +-----------------+

(* Signal Descriptions *)
(* PV: Process Variable (e.g., sensor reading, 0-100%) *)
(* PT: Preset Time for TON (e.g., 5 seconds) *)
(* ET: Elapsed Time output from TON *)
(* Q: Timer Done output (TRUE when ET >= PT) *)
(* SP: Setpoint for PID (e.g., desired value, 100.0) *)
(* CV: Control Variable (PID output, e.g., 0-100%) *)

(* Notes *)
(* - Analog Input block reads raw sensor data and outputs PV *)
(* - TON Timer uses PV as input (e.g., starts when PV > 0) *)
(* - PID block computes CV based on PV and SP, using internal PID algorithm *)
(* - Analog Output block converts CV to actuator signal *)
(* - Diagram uses ASCII (+---+, -->) for clarity and alignment *)
(* - Signal paths are branched to show PV feeding both TON and PID *)
