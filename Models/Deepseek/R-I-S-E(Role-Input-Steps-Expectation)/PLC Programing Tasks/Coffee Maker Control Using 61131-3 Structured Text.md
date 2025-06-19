FUNCTION_BLOCK FB_CoffeeMachineController
VAR_INPUT
    EmergencyStop : BOOL; // Immediately halts all operations
    Start : BOOL; // Initiates the drink preparation process
    CoffeeMilk : BOOL; // Selects coffee with milk
    CoffeeOnly : BOOL; // Selects coffee without milk
    MixerLevelFull : BOOL; // TRUE when the mixer is full
END_VAR

VAR_OUTPUT
    CoffeeValve : BOOL := FALSE; // Controls coffee valve
    MilkValve : BOOL := FALSE; // Controls milk valve
    OutputValve : BOOL := FALSE; // Controls output valve
    Mixer : BOOL := FALSE; // Controls mixer motor
    MachineState : INT := 0; // Tracks current state of the machine
END_VAR

VAR
    Timer : TON; // Timer for mixing duration
    State : INT := 0; // Internal state tracking
END_VAR

// Initialize timer parameters
Timer(IN := FALSE, PT := T#4S);

// Handle Emergency Stop
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    OutputValve := FALSE;
    Mixer := FALSE;
    Timer(IN := FALSE); // Stop any ongoing timer
    State := 0; // Reset state machine to IDLE
    MachineState := -1; // Indicate emergency stop condition
    EXIT;
END_IF;

// Main state machine logic
CASE State OF
    0: // IDLE state
        IF Start AND (CoffeeMilk OR CoffeeOnly) THEN
            IF CoffeeMilk THEN
                CoffeeValve := TRUE;
                MilkValve := TRUE;
            ELSIF CoffeeOnly THEN
                CoffeeValve := TRUE;
            END_IF;
            State := 1;
        END_IF;

    1: // Filling tanks
        IF MixerLevelFull THEN
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Timer(IN := TRUE); // Start timer for mixing
            State := 2;
        END_IF;

    2: // Mixing
        IF Timer.Q THEN
            Mixer := FALSE;
            Timer(IN := FALSE);
            OutputValve := TRUE;
            State := 3;
        ELSE
            Mixer := TRUE;
        END_IF;

    3: // Dispensing
        // Assume dispensing completes immediately upon opening the output valve
        OutputValve := FALSE;
        State := 0; // Return to IDLE after dispensing
        MachineState := 1; // Ready for next operation

    ELSE:
        State := 0; // Default to IDLE if in an unknown state
END_CASE;

PROGRAM PLC_PRG
VAR
    CoffeeCtrl : FB_CoffeeMachineController;

    // Simulated inputs
    EmergencyStopBtn : BOOL := FALSE;
    StartBtn : BOOL := FALSE;
    CoffeeWithMilkBtn : BOOL := FALSE;
    CoffeeOnlyBtn : BOOL := FALSE;
    MixerFullSensor : BOOL := FALSE;

    // Outputs
    CoffeeVlv : BOOL;
    MilkVlv : BOOL;
    OutputVlv : BOOL;
    MixMotor : BOOL;
    Status : INT;
END_VAR

// Call the controller function block
CoffeeCtrl(
    EmergencyStop := EmergencyStopBtn,
    Start := StartBtn,
    CoffeeMilk := CoffeeWithMilkBtn,
    CoffeeOnly := CoffeeOnlyBtn,
    MixerLevelFull := MixerFullSensor
);

// Map outputs
CoffeeVlv := CoffeeCtrl.CoffeeValve;
MilkVlv := CoffeeCtrl.MilkValve;
OutputVlv := CoffeeCtrl.OutputValve;
MixMotor := CoffeeCtrl.Mixer;
Status := CoffeeCtrl.MachineState;
