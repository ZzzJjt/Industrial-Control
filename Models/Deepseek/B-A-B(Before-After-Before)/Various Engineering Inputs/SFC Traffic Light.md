Here is a **Sequential Function Chart (SFC)** representation of a **traffic light control system**, using ASCII art and following the structure defined by **IEC 61131-3**. This layout uses textual blocks to represent:

- **Steps**: Green, Yellow, Red  
- **Transitions**: Based on timer outputs  
- **Timers**: TON (On-Delay Timer) for each step  
- **Sequence logic**: Loops continuously through Green → Yellow → Red → Green  

---

# 🚦 IEC 61131-3 Sequential Function Chart (SFC): Traffic Light Control

```
            +-------------------+
            |     Initial Step  |
            |     (Start State) |
            +-------------------+
                      |
                      v
            +-----------------------+     tGreen.Q (T#5s)
            |       Step S1:        |--------------------------+
            |     GREEN LIGHT ON    |                          |
            |   TON: tGreen (T#5s) |                          |
            +-----------------------+                          |
                      |                                        |
                      v                                        v
            +-----------------------+     tYellow.Q (T#2s)          +-----------------------+
            |       Step S2:        |--------------------------+    |       Step S3:        |
            |    YELLOW LIGHT ON    |                           |    |      RED LIGHT ON     |
            |  TON: tYellow (T#2s) |                           |    |    TON: tRed (T#5s)   |
            +-----------------------+                           |    +-----------------------+
                      |                                        |              |
                      +----------------------------------------+              |
                                    Transition: tYellow.Q                     |
                                                                              v
                                                                        +---------------+
                                                                        | Final Step    |
                                                                        | (Loop Back)   |
                                                                        +---------------+
                                                                              ↑
                                                              Transition: tRed.Q (T#5s)
```

---

## 🔁 Sequence Flow Description

| Step | Action                         | Duration | Next Transition         |
|------|--------------------------------|----------|--------------------------|
| S1   | GREEN light ON                 | 5 sec    | `tGreen.Q = TRUE`        |
| S2   | YELLOW light ON                | 2 sec    | `tYellow.Q = TRUE`       |
| S3   | RED light ON                   | 5 sec    | `tRed.Q = TRUE`          |

> After S3 completes, the sequence returns to **S1 (GREEN)**, forming a continuous loop.

---

## ⏱️ Timer Definitions (TON – On-Delay Timer)

| Timer    | Purpose                        | Preset Time |
|----------|--------------------------------|-------------|
| tGreen   | Delay before switching to Yellow | T#5s        |
| tYellow  | Delay before switching to Red    | T#2s        |
| tRed     | Delay before switching to Green  | T#5s        |

---

## 🧾 Optional: IEC 61131-3 Structured Text Snippet (for Reference)

If you're implementing this in a PLC using **Structured Text**, here's a simplified version of how the timers and transitions could be implemented:

```pascal
PROGRAM PLC_PRG
VAR
    tGreen, tYellow, tRed: TON;
    state: INT := 0; // 0=Green, 1=Yellow, 2=Red
END_VAR

CASE state OF
    0: // GREEN
        GreenLight := TRUE;
        YellowLight := FALSE;
        RedLight := FALSE;
        tGreen(IN := TRUE, PT := T#5s);
        IF tGreen.Q THEN
            tGreen(IN := FALSE);
            state := 1;
        END_IF;

    1: // YELLOW
        GreenLight := FALSE;
        YellowLight := TRUE;
        RedLight := FALSE;
        tYellow(IN := TRUE, PT := T#2s);
        IF tYellow.Q THEN
            tYellow(IN := FALSE);
            state := 2;
        END_IF;

    2: // RED
        GreenLight := FALSE;
        YellowLight := FALSE;
        RedLight := TRUE;
        tRed(IN := TRUE, PT := T#5s);
        IF tRed.Q THEN
            tRed(IN := FALSE);
            state := 0;
        END_IF;
END_CASE;
```

---

This textual SFC provides a clear and structured way to represent the **sequential behavior of a traffic light system**, suitable for documentation, planning, or implementation in an IEC 61131-3 compliant PLC environment.

Would you like this exported as Markdown, extended with **manual override logic**, **pedestrian crosswalk integration**, or formatted into a full **GRAFCET / SFC XML import**?
