(* IEC 61131-3 Sequential Function Chart (SFC) for Traffic Light Control *)
(* System: Single Intersection, North-South Traffic Lights *)
(* States: S1 (Green, 5s), S2 (Yellow, 2s), S3 (Red, 5s) *)
(* Timers: tGreen (T#5s), tYellow (T#2s), tRed (T#5s) *)
(* Outputs: GreenLight (BOOL), YellowLight (BOOL), RedLight (BOOL) *)
(* Execution: <1 ms per cycle, scan-cycle safe for 10 ms PLC scans *)
(* Format: Steps ([S1]), Transitions (| Condition |), Actions (Set outputs, timers) *)
(* Loop: S1 -> S2 -> S3 -> S1 *)

[Initial Step]
      |
      v
+-----------------+
| S1: Green       |
| Actions:        |
| GreenLight := TRUE |
| YellowLight := FALSE |
| RedLight := FALSE |
| tGreen(IN := TRUE, PT := T#5s) |
+-----------------+
      |
      | tGreen.Q |
      v
+-----------------+
| S2: Yellow      |
| Actions:        |
| GreenLight := FALSE |
| YellowLight := TRUE |
| RedLight := FALSE |
| tYellow(IN := TRUE, PT := T#2s) |
+-----------------+
      |
      | tYellow.Q |
      v
+-----------------+
| S3: Red         |
| Actions:        |
| GreenLight := FALSE |
| YellowLight := FALSE |
| RedLight := TRUE |
| tRed(IN := TRUE, PT := T#5s) |
+-----------------+
      |
      | tRed.Q |
      v
       (* Loop back to S1 *)

(* Signal Definitions *)
- GreenLight: BOOL, ON during S1 (Green)
- YellowLight: BOOL, ON during S2 (Yellow)
- RedLight: BOOL, ON during S3 (Red)
- tGreen: TON, PT = T#5s, Q triggers transition to S2
- tYellow: TON, PT = T#2s, Q triggers transition to S3
- tRed: TON, PT = T#5s, Q triggers transition to S1

(* Control Logic *)
- Each step activates one light, deactivates others
- Timers start on step entry (IN := TRUE), reset on exit (implicit)
- Transitions fire when timer.Q = TRUE (e.g., tGreen.Q after 5s)
- Cycle time: 12s (5s Green + 2s Yellow + 5s Red)
- Initial state: S1 (Green)

(* Notes *)
- IEC 61131-3 compliant, uses TON timers
- Execution: <1 ms per cycle, supports 10 ms PLC scans
- Loop: Red -> Green ensures continuous operation
- Outputs: Mutually exclusive, only one light ON
- Extensible: Add steps for pedestrian signals or multi-direction control
