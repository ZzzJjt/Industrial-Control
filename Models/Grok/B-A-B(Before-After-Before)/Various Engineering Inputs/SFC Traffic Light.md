```
IEC 61131-3 Sequential Function Chart (SFC) for Traffic Light System
-----------------------------------------------------------------
Description: Represents a standard traffic light system in ASCII art, following
IEC 61131-3 SFC syntax. The system cycles through Green (5s), Yellow (2s), and
Red (5s) states, with timer-driven transitions in a continuous loop. The chart
is designed for plain-text environments, ensuring clarity for documentation,
version control, or early-stage logic planning.

Notation:
- Steps: +----+ blocks with step names (e.g., |S1_Green|).
- Transitions: ====> with conditions (e.g., tGreen.Q).
- Flow: | for vertical sequence, ---- for connections.
- Loop: Return from Red to Green via tRed.Q transition.
- Timers: TON (e.g., tGreen, PT=T#5s) for delays.
- Actions: Each step sets light outputs (e.g., GreenLight := TRUE) and starts timer.

ASCII SFC Diagram:
=================

       +-------------+
       | S1_Green    |
       | TON: tGreen |
       | PT: T#5s    |
       | GreenLight  |
       +-------------+
              |
       tGreen.Q ====> (Green timer done)
              |
       +-------------+
       | S2_Yellow  |
       | TON: tYellow|
       | PT: T#2s    |
       | YellowLight |
       +-------------+
              |
       tYellow.Q ====> (Yellow timer done)
              |
       +-------------+
       | S3_Red     |
       | TON: tRed  |
       | PT: T#5s    |
       | RedLight   |
       +-------------+
              |
       tRed.Q ====> (Red timer done)
              |
              +-----------------------------+
                                            |
                                            v
                                       +-------------+
                                       | S1_Green    |
                                       +-------------+

Control Logic:
- S1_Green:
  - Actions: GreenLight := TRUE; YellowLight := FALSE; RedLight := FALSE; tGreen(IN := TRUE, PT := T#5s).
  - Transition: tGreen.Q (timer done after 5s) --> S2_Yellow.
- S2_Yellow:
  - Actions: GreenLight := FALSE; YellowLight := TRUE; RedLight := FALSE; tYellow(IN := TRUE, PT := T#2s).
  - Transition: tYellow.Q (timer done after 2s) --> S3_Red.
- S3_Red:
  - Actions: GreenLight := FALSE; YellowLight := FALSE; RedLight := TRUE; tRed(IN := TRUE, PT := T#5s).
  - Transition: tRed.Q (timer done after 5s) --> S1_Green.

Notes:
- Steps: S1_Green, S2_Yellow, S3_Red represent traffic light states.
- Timers: tGreen (5s), tYellow (2s), tRed (5s) drive transitions.
- Outputs: GreenLight, YellowLight, RedLight (BOOL) control light states.
- Transitions: tGreen.Q, tYellow.Q, tRed.Q are timer completion flags (TRUE when PT reached).
- Flow: Continuous loop (Green --> Yellow --> Red --> Green).
- IEC 61131-3 Compliance: Uses SFC structure with steps, transitions, and TON timers.
- Safety: Ensures only one light is active per step (e.g., GreenLight exclusive).
- Usage: Guides PLC logic development, HMI configuration, and documentation.
- Format: Compact ASCII, readable in code editors (e.g., Notepad, VS Code).
- Assumptions:
  - Single traffic light system (one direction); multi-direction systems require additional SFCs.
  - Timers use TON (Timer On Delay) with T#5s, T#2s durations.
  - PLC supports BOOL outputs and TON timers per IEC 61131-3.
  - Sequence assumes standard timing (adjustable for specific requirements).
-----------------------------------------------------------------
```
