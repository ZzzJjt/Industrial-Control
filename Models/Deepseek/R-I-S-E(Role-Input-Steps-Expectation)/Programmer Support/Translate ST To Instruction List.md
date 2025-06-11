VAR
    Mode: INT := 0; // 0 = Manual, 1 = Automatic
    ButtonStart: BOOL;
    ButtonStop: BOOL;
    ConveyorOn: BOOL;
    ArmDown: BOOL;
END_VAR

IF Mode = 0 THEN // Manual Mode
    IF ButtonStart THEN
        ConveyorOn := TRUE;
        WAIT FOR 2s; // Simulated delay for simplicity
        ArmDown := TRUE;
    ELSIF ButtonStop THEN
        ArmDown := FALSE;
        ConveyorOn := FALSE;
    END_IF;
ELSIF Mode = 1 THEN // Automatic Mode
    ConveyorOn := TRUE;
    WAIT FOR 5s;
    ArmDown := TRUE;
    WAIT FOR 3s;
    ArmDown := FALSE;
    ConveyorOn := FALSE;
END_IF;

// Variable Declarations are handled by the environment, so not directly in IL

// Entry point or main loop start
LBL MAIN_LOOP

// Check if Mode is Manual (0)
LD %MW0      // Load Mode value into accumulator
L 0          // Load immediate value 0
==           // Compare Mode == 0
JMPC MANUAL_MODE // Jump to MANUAL_MODE if true

// If not manual, check if automatic (Mode == 1)
L 1          // Load immediate value 1
==           // Compare Mode == 1
JMPC AUTO_MODE // Jump to AUTO_MODE if true

JMP MAIN_LOOP // Loop back to start if neither condition met

LBL MANUAL_MODE
// Manual Mode Logic
LD %QX0.0    // Load ButtonStart state
JCNZ START_BUTTON_PRESSED // Jump if button pressed

LD %QX0.1    // Load ButtonStop state
JCNZ STOP_BUTTON_PRESSED // Jump if stop button pressed

JMP EXIT_MANUAL_MODE // Exit manual mode processing

LBL START_BUTTON_PRESSED
S %QW0.0     // Set ConveyorOn to TRUE
// Simulate WAIT FOR 2s using timer or counter mechanism
// This would require setting up a timer/counter elsewhere in the program
// For demonstration, assume TON is used
CALL TON_2S  // Call timer function/block for 2 seconds
LD %T0       // Load timer done status
JCNZ AFTER_WAIT_2S // Jump if timer done

JMP EXIT_MANUAL_MODE // Continue waiting if not done

LBL AFTER_WAIT_2S
S %QW0.1     // Set ArmDown to TRUE
JMP EXIT_MANUAL_MODE

LBL STOP_BUTTON_PRESSED
R %QW0.1     // Reset ArmDown to FALSE
R %QW0.0     // Reset ConveyorOn to FALSE
JMP EXIT_MANUAL_MODE

LBL EXIT_MANUAL_MODE
JMP MAIN_LOOP // Return to main loop

LBL AUTO_MODE
// Automatic Mode Logic
S %QW0.0     // Set ConveyorOn to TRUE
// Simulate WAIT FOR 5s using timer/counter
CALL TON_5S  // Timer for 5 seconds
LD %T1       // Load timer done status
JCNZ AFTER_WAIT_5S // Jump if timer done

JMP EXIT_AUTO_MODE // Continue waiting if not done

LBL AFTER_WAIT_5S
S %QW0.1     // Set ArmDown to TRUE
// Simulate WAIT FOR 3s using timer/counter
CALL TON_3S  // Timer for 3 seconds
LD %T2       // Load timer done status
JCNZ AFTER_WAIT_3S // Jump if timer done

JMP EXIT_AUTO_MODE // Continue waiting if not done

LBL AFTER_WAIT_3S
R %QW0.1     // Reset ArmDown to FALSE
R %QW0.0     // Reset ConveyorOn to FALSE
JMP EXIT_AUTO_MODE

LBL EXIT_AUTO_MODE
JMP MAIN_LOOP // Return to main loop
