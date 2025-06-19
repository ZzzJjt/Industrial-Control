FUNCTION_BLOCK CoffeeMachineControl
{ CMC: Coffee Machine Control Logic }
{ Author: Automation Engineer }
{ Date: 2025-05-13 }

(*
    Description:
    Implements safe and state-based control logic for a coffee machine with:
    - Three tanks: Coffee, Milk, Mixer
    - Three valves: Coffee, Milk, Output
    - Two drink options: Coffee Only, Coffee + Milk
    - Emergency Stop override
    - Failsafe detection for mixer fault and overflow

    Operation Sequence:
    1. Fill mixer with selected ingredients
    2. Mix for 4 seconds
    3. Dispense via output valve

    Safety Priority:
    Emergency Stop takes precedence over all other states.
    Mixer Fault or Overflow stops the process safely.
*)

TYPE E_CoffeeState:
(
    IDLE = 0,
    FILLING = 1,
    MIXING = 2,
    DISPENSING = 3
);
END_TYPE

VAR_INPUT
    StartButton : BOOL := FALSE;
    CoffeeOnlyButton : BOOL := FALSE;
    CoffeeMilkButton : BOOL := FALSE;
    EmergencyStop : BOOL := FALSE;
    MixerLevelFull : BOOL := FALSE;
    MixerFault : BOOL := FALSE;
END_VAR

VAR_OUTPUT
    CoffeeValve : BOOL := FALSE;
    MilkValve : BOOL := FALSE;
    OutputValve : BOOL := FALSE;
    Mixer : BOOL := FALSE;
END_VAR

// Internal variables
VAR
    State : E_CoffeeState := IDLE;
    Timer_Mix : TON;
END_VAR

// Emergency Stop always overrides everything
IF EmergencyStop THEN
    State := IDLE;
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    Mixer := FALSE;
    OutputValve := FALSE;
    Timer_Mix(IN:=FALSE, PT:=T#4s);
    EXIT; // Exit early if emergency stop is active
END_IF;

CASE State OF

    IDLE:
        (*
            Waiting for start signal.
            Reset all outputs except safety interlocks.
        *)
        CoffeeValve := FALSE;
        MilkValve := FALSE;
        Mixer := FALSE;
        OutputValve := FALSE;

        IF StartButton THEN
            IF CoffeeMilkButton THEN
                CoffeeValve := TRUE;
                MilkValve := TRUE;
                State := FILLING;
            ELSIF CoffeeOnlyButton THEN
                CoffeeValve := TRUE;
                MilkValve := FALSE;
                State := FILLING;
            END_IF;
        END_IF;

    FILLING:
        (*
            Fill mixer tank until full.
            If mixer fault or overflow occurs, stop immediately.
        *)
        IF MixerFault OR NOT MixerLevelFull THEN
            // Optional: Trigger alarm or log event
            State := IDLE;
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            EXIT;
        END_IF;

        IF MixerLevelFull THEN
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := TRUE;
            Timer_Mix(IN:=TRUE, PT:=T#4s);
            State := MIXING;
        END_IF;

    MIXING:
        (*
            Run mixer for 4 seconds.
            If mixer fault detected, stop immediately.
        *)
        IF MixerFault THEN
            State := IDLE;
            Mixer := FALSE;
            EXIT;
        END_IF;

        IF Timer_Mix.Q THEN
            Mixer := FALSE;
            OutputValve := TRUE;
            State := DISPENSING;
        END_IF;

    DISPENSING:
        (*
            Dispense the mixture.
            Optionally add timer to close valve automatically.
        *)
        // OutputValve remains open
        // Could add timeout logic to return to IDLE
        ;

END_CASE;


PROGRAM PLC_PRG
VAR
    StartBtn AT %IX100.0 : BOOL;
    CoffeeOnlyBtn AT %IX100.1 : BOOL;
    CoffeeMilkBtn AT %IX100.2 : BOOL;
    EmergStop AT %IX100.3 : BOOL;
    MixerFull AT %IX100.4 : BOOL;
    MixerFailed AT %IX100.5 : BOOL;

    CoffeeMachine: CoffeeMachineControl;

    // Outputs
    CoffeeValveOut AT %QX0.0 : BOOL;
    MilkValveOut AT %QX0.1 : BOOL;
    OutputValveOut AT %QX0.2 : BOOL;
    MixerMotorOut AT %QX0.3 : BOOL;
END_VAR

// Run the coffee machine control logic
CoffeeMachine(
    StartButton := StartBtn,
    CoffeeOnlyButton := CoffeeOnlyBtn,
    CoffeeMilkButton := CoffeeMilkBtn,
    EmergencyStop := EmergStop,
    MixerLevelFull := MixerFull,
    MixerFault := MixerFailed
);

// Map outputs
CoffeeValveOut := CoffeeMachine.CoffeeValve;
MilkValveOut := CoffeeMachine.MilkValve;
OutputValveOut := CoffeeMachine.OutputValve;
MixerMotorOut := CoffeeMachine.Mixer;
