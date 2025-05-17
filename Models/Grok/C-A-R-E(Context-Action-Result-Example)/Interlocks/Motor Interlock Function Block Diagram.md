Motor Interlock Function Block Diagram Description
Overview
This Function Block Diagram (FBD) represents the MotorInterlock function block for a chemical mixing system, ensuring a motor starts only when interdependent equipment is inactive, per IEC 61131-3.
Diagram Components

Function Block: MotorInterlock

Shape: Rectangle with inputs on left, output on right.
Label: MotorInterlock centered inside block.


Inputs (left side, vertically aligned):

Equipment1Running: BOOL (e.g., agitator running, TRUE = active).
Equipment2Running: BOOL (e.g., feed pump running, TRUE = active).
Equipment3Running: BOOL (e.g., conveyor running, TRUE = active).


Internal Logic (inside block):

NOT Gates: Three NOT gates, one per input.
NOT Equipment1Running
NOT Equipment2Running
NOT Equipment3Running


AND Gate: Three-input AND gate combining NOT gate outputs.
Input 1: NOT Equipment1Running
Input 2: NOT Equipment2Running
Input 3: NOT Equipment3Running
Output: AllowStart




Output (right side):

AllowStart: BOOL (TRUE when all equipment inactive, allowing motor start).



Connections

Each input (Equipment1Running, etc.) connects to a NOT gate.
Each NOT gate output connects to an input of the three-input AND gate.
The AND gate output connects directly to AllowStart.

Logic Description

AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running
If any input is TRUE (equipment running), its NOT gate outputs FALSE, setting AllowStart := FALSE.
Only when all inputs are FALSE (equipment inactive) do all NOT gates output TRUE, setting AllowStart := TRUE via the AND gate.

Notes

Standards: Conforms to IEC 61131-3 FBD conventions.
Usage: Prevents motor startup if agitator, feed pump, or conveyor is active, ensuring safe sequencing.
Assumptions:
Inputs are reliable digital signals (TRUE = running, FALSE = stopped).
Output AllowStart interfaces with motor control logic (e.g., start relay).


Use Case: In a mixing system, AllowStart = FALSE if agitator (Equipment1Running) or feed pump (Equipment2Running) is active, allowing startup only when all are stopped.

