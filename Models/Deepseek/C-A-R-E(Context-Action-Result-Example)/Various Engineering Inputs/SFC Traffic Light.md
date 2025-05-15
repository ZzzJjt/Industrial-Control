Certainly! Below is an ASCII art Sequential Function Chart (SFC) for a traffic light system using IEC 61131-3 conventions. The chart includes three main steps: Green, Yellow, and Red lights, with transitions based on timer conditions using TON timers to enforce proper delays.

### ASCII Art SFC for Traffic Light System

```
[<title="ASCII_SFC_Traffic_Light">]
[Step1] Green Light
TON tGreen (PT := T#5s)
|
V (tGreen.Q)
[Step2] Yellow Light
TON tYellow (PT := T#2s)
|
V (tYellow.Q)
[Step3] Red Light
TON tRed (PT := T#5s)
|
V (tRed.Q)
--> Loop back to Step1
```

### Detailed Explanation of the ASCII SFC

- **Steps:**
  - **[Step1] Green Light:** Represents the state when the green light is active.
  - **[Step2] Yellow Light:** Represents the state when the yellow light is active.
  - **[Step3] Red Light:** Represents the state when the red light is active.

- **Timers:**
  - **TON tGreen (PT := T#5s):** A TON (On-Delay Timer) that starts when the green light step begins and times for 5 seconds.
  - **TON tYellow (PT := T#2s):** A TON timer that starts when the yellow light step begins and times for 2 seconds.
  - **TON tRed (PT := T#5s):** A TON timer that starts when the red light step begins and times for 5 seconds.

- **Transitions:**
  - **V (tGreen.Q):** When the green light timer (`tGreen`) completes its 5-second delay, it transitions to the yellow light step.
  - **V (tYellow.Q):** When the yellow light timer (`tYellow`) completes its 2-second delay, it transitions to the red light step.
  - **V (tRed.Q):** When the red light timer (`tRed`) completes its 5-second delay, it transitions back to the green light step, creating a continuous cycle.

### Visual Representation

Here is a more detailed visual representation to enhance readability:

```
[<title="ASCII_SFC_Traffic_Light_Visual">]
+-------------------+
| [Step1] Green Light |
| TON tGreen        |
| PT := T#5s        |
+-------------------+
          |
          V (tGreen.Q)
+-------------------+
| [Step2] Yellow Light|
| TON tYellow       |
| PT := T#2s        |
+-------------------+
          |
          V (tYellow.Q)
+-------------------+
| [Step3] Red Light   |
| TON tRed          |
| PT := T#5s        |
+-------------------+
          |
          V (tRed.Q)
          |
          +----------->
          |             |
          |             |
          +-------------+
              |
              v
      [Loop back to Step1]
```

This ASCII SFC provides a clear and structured representation of the traffic light control logic, suitable for code planning before PLC implementation, control logic documentation, and teaching or presenting IEC 61131-3 concepts in plain text environments.
