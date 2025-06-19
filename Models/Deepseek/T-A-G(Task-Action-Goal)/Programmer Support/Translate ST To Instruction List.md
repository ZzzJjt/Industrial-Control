// Declare Variables
LD #FALSE          // Load FALSE
ST Mode            // Store in Mode
LD #FALSE          // Load FALSE
ST StartButton     // Store in StartButton
LD #FALSE          // Load FALSE
ST StopButton      // Store in StopButton
LD #FALSE          // Load FALSE
ST PickComplete    // Store in PickComplete
LD #FALSE          // Load FALSE
ST PlaceComplete   // Store in PlaceComplete
LD #FALSE          // Load FALSE
ST Running         // Store in Running

// Initialize Timer
LD #T#0s           // Load zero time
TON                // Initialize timer
LD #FALSE          // Load FALSE
ST DelayTimer.IN   // Store in DelayTimer.IN

// Main Control Logic
LABEL MainLoop

// Check Mode
LD Mode
JMPC AutoMode      // Jump if Mode is TRUE (Automatic)

// Manual Mode
LD StartButton
LD Running
XOR                // XOR to check if StartButton is TRUE and Running is FALSE
JMPZ ManualRunCheck // Jump if result is FALSE

// Start Process in Manual Mode
LD #TRUE
ST Running
LD #FALSE
ST PickComplete
LD #FALSE
ST PlaceComplete

ManualRunCheck:
LD StopButton
LD Running
AND                // AND to check if StopButton is TRUE and Running is TRUE
JMPZ PickOperationCheck // Jump if result is FALSE

// Stop Process in Manual Mode
LD #FALSE
ST Running

PickOperationCheck:
LD Running
LD PickComplete
XOR                // XOR to check if Running is TRUE and PickComplete is FALSE
JMPZ PlaceOperationCheck // Jump if result is FALSE

// Simulate Pick Operation in Manual Mode
LD #TRUE
ST PickComplete
LD #T#500ms        // Load 500 ms delay
TON                // Start timer
LD DelayTimer.Q
JMPZ PickOperationCheck // Repeat until timer expires

PlaceOperationCheck:
LD Running
LD PickComplete
AND                // AND to check if Running is TRUE and PickComplete is TRUE
LD PlaceComplete
XOR                // XOR to check if PlaceComplete is FALSE
JMPZ ResetOperationsCheck // Jump if result is FALSE

// Simulate Place Operation in Manual Mode
LD #TRUE
ST PlaceComplete
LD #T#500ms        // Load 500 ms delay
TON                // Start timer
LD DelayTimer.Q
JMPZ PlaceOperationCheck // Repeat until timer expires

ResetOperationsCheck:
LD Running
LD PickComplete
AND                // AND to check if Running is TRUE and PickComplete is TRUE
LD PlaceComplete
AND                // AND to check if PlaceComplete is TRUE
JMPZ MainLoop      // Jump back to main loop if not all conditions are met

// Reset Operations in Manual Mode
LD #FALSE
ST PickComplete
LD #FALSE
ST PlaceComplete
LD #FALSE
ST Running
JMP MainLoop       // Jump back to main loop

AutoMode:
LD StartButton
LD Running
XOR                // XOR to check if StartButton is TRUE and Running is FALSE
JMPZ AutoRunCheck  // Jump if result is FALSE

// Start Process in Automatic Mode
LD #TRUE
ST Running
LD #FALSE
ST PickComplete
LD #FALSE
ST PlaceComplete

AutoRunCheck:
LD StopButton
LD Running
AND                // AND to check if StopButton is TRUE and Running is TRUE
JMPZ AutoPickOperationCheck // Jump if result is FALSE

// Stop Process in Automatic Mode
LD #FALSE
ST Running

AutoPickOperationCheck:
LD Running
LD PickComplete
XOR                // XOR to check if Running is TRUE and PickComplete is FALSE
JMPZ AutoPlaceOperationCheck // Jump if result is FALSE

// Simulate Pick Operation in Automatic Mode
LD #TRUE
ST PickComplete
LD #T#500ms        // Load 500 ms delay
TON                // Start timer
LD DelayTimer.Q
JMPZ AutoPickOperationCheck // Repeat until timer expires

AutoPlaceOperationCheck:
LD Running
LD PickComplete
AND                // AND to check if Running is TRUE and PickComplete is TRUE
LD PlaceComplete
XOR                // XOR to check if PlaceComplete is FALSE
JMPZ AutoResetOperationsCheck // Jump if result is FALSE

// Simulate Place Operation in Automatic Mode
LD #TRUE
ST PlaceComplete
LD #T#500ms        // Load 500 ms delay
TON                // Start timer
LD DelayTimer.Q
JMPZ AutoPlaceOperationCheck // Repeat until timer expires

AutoResetOperationsCheck:
LD Running
LD PickComplete
AND                // AND to check if Running is TRUE and PickComplete is TRUE
LD PlaceComplete
AND                // AND to check if PlaceComplete is TRUE
JMPZ MainLoop      // Jump back to main loop if not all conditions are met

// Reset Operations in Automatic Mode
LD #FALSE
ST PickComplete
LD #FALSE
ST PlaceComplete
LD #FALSE
ST Running

MainLoop:
JMP MainLoop       // Infinite loop to keep scanning



