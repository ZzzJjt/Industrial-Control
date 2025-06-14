VAR
    Mode : BOOL;           // TRUE = Auto, FALSE = Manual
    Start : BOOL;
    Picked : BOOL;
    Placed : BOOL;
    Timer1 : TON;
END_VAR

IF Mode THEN // Automatic
    IF Start THEN
        Timer1(IN := TRUE, PT := T#3s);
        IF Timer1.Q THEN
            Picked := TRUE;
            Placed := TRUE;
        END_IF
    ELSE
        Timer1(IN := FALSE);
    END_IF
ELSE // Manual Mode
    Picked := TRUE;
    Placed := TRUE;
END_IF

      ; === IL Variable Declarations ===
      ; Mode        - BOOL (Auto/Manual selector)
      ; Start       - BOOL (Start button)
      ; Picked      - BOOL (Output)
      ; Placed      - BOOL (Output)
      ; Timer1      - TON instance with PT = T#3s

      ; === BEGIN MAIN LOGIC ===

      LD      Mode                ; Check if in Auto mode
      JMPC    AUTO_MODE           ; If TRUE, jump to auto control
      JMP     MANUAL_MODE         ; Else jump to manual mode

AUTO_MODE:
      LD      Start
      JMPC    START_TRUE
      ; Start is FALSE → reset timer
      LD      FALSE
      ST      Timer1.IN
      JMP     END_PROGRAM

START_TRUE:
      ; Start is TRUE → enable timer
      LD      TRUE
      ST      Timer1.IN

      ; TON logic is assumed to be handled externally by the PLC system scan

      LD      Timer1.Q            ; Check if timer expired
      JMPC    SET_OUTPUTS
      JMP     END_PROGRAM

SET_OUTPUTS:
      LD      TRUE
      ST      Picked
      ST      Placed
      JMP     END_PROGRAM

MANUAL_MODE:
      LD      TRUE
      ST      Picked
      ST      Placed

END_PROGRAM:
      ; End of logic
      NOP
