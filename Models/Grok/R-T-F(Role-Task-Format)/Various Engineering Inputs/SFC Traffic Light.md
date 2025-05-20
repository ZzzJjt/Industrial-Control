(* IEC 61131-3 Sequential Function Chart (SFC) for Traffic Light System *)
(* Defines Green, Yellow, Red steps with TON timers and continuous loop *)
(* Uses ASCII art for steps (+---+), transitions (-->, |), and timer conditions *)
(* Designed for documentation, PLC programming, and peer review *)

[Traffic Light SFC]

+--------------------+
| Step: Green Light  |
| GreenLight := TRUE |
| TON tGreen         |
| (PT := T#5s)       |
+--------------------+
        |
        | tGreen.Q
        v
+--------------------+
| Step: Yellow Light |
| YellowLight := TRUE|
| TON tYellow        |
| (PT := T#2s)       |
+--------------------+
        |
        | tYellow.Q
        v
+--------------------+
| Step: Red Light    |
| RedLight := TRUE   |
| TON tRed           |
| (PT := T#5s)       |
+--------------------+
        |
        | tRed.Q
        v
        +--> Loop to Green Light

(* Signal Descriptions *)
(* GreenLight: BOOL, Activates green light output *)
(* YellowLight: BOOL, Activates yellow light output *)
(* RedLight: BOOL, Activates red light output *)
(* tGreen: TON, Timer for Green step (5 seconds) *)
(* tYellow: TON, Timer for Yellow step (2 seconds) *)
(* tRed: TON, Timer for Red step (5 seconds) *)
(* tGreen.Q, tYellow.Q, tRed.Q: BOOL, Timer Done outputs triggering transitions *)

(* Control Logic *)
(* Green Light: Activates for 5s, transitions to Yellow when tGreen.Q = TRUE *)
(* Yellow Light: Activates for 2s, transitions to Red when tYellow.Q = TRUE *)
(* Red Light: Activates for 5s, loops back to Green when tRed.Q = TRUE *)

(* Notes *)
(* - Each step executes a TON timer with preset duration (PT) *)
(* - Transitions occur on timer .Q (Done) output, ensuring precise timing *)
(* - Continuous loop (Red --> Green) maintains cyclic operation *)
(* - ASCII layout uses +---+ for steps, | and --> for transitions *)
(* - Compliant with IEC 61131-3 SFC standards for PLC implementation *)
