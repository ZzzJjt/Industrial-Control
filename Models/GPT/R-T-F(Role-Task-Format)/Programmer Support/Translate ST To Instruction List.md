VAR
    StartButton : BOOL;
    ManualMode  : BOOL;
    AutoMode    : BOOL;
    ArmExtended : BOOL;
    ItemDetected: BOOL;
    ProcessActive : BOOL;
    StepTimer   : TON;
END_VAR

IF StartButton THEN
    IF ManualMode THEN
        ArmExtended := TRUE;
    ELSIF AutoMode THEN
        ProcessActive := TRUE;
    END_IF;
END_IF;

IF ProcessActive THEN
    StepTimer(IN := TRUE, PT := T#5s);
    IF StepTimer.Q THEN
        ArmExtended := FALSE;
        ProcessActive := FALSE;
    END_IF;
ELSE
    StepTimer(IN := FALSE);
END_IF;

// === StartButton pressed? ===
LD StartButton         // Load StartButton
JMPC CHECK_MODE        // If TRUE, jump to CHECK_MODE
JMP SKIP_LOGIC         // If FALSE, skip all

// === Check which mode is active ===
CHECK_MODE:
// Manual mode?
LD ManualMode
JMPC SET_ARM           // If TRUE, jump to extend arm
// Else check Auto mode
LD AutoMode
JMPC START_PROCESS     // If TRUE, activate process
JMP SKIP_LOGIC         // If neither, skip

// === Manual: Extend Arm ===
SET_ARM:
LD TRUE
ST ArmExtended         // ArmExtended := TRUE
JMP SKIP_LOGIC

// === Auto: Set ProcessActive ===
START_PROCESS:
LD TRUE
ST ProcessActive       // ProcessActive := TRUE

// === Evaluate ProcessActive ===
SKIP_LOGIC:
LD ProcessActive
JMPC RUN_TIMER         // If TRUE, go to timer logic
JMP STOP_TIMER         // If FALSE, disable timer

// === Activate TON Timer ===
RUN_TIMER:
LD TRUE
ST StepTimer.IN        // StepTimer(IN) := TRUE
LD T#5s
ST StepTimer.PT        // Preset Time
// Simulate TON block calling here

// === Timer Done? ===
LD StepTimer.Q
JMPC RESET_SEQUENCE    // If TRUE, go reset outputs
JMP END                // Otherwise, skip

// === Reset logic after timer ===
RESET_SEQUENCE:
LD FALSE
ST ArmExtended
ST ProcessActive
JMP END

// === Stop timer when not active ===
STOP_TIMER:
LD FALSE
ST StepTimer.IN        // Disable timer when idle

END:
NOP
