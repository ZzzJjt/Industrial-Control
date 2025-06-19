Motor Interlock Function Block Diagram
Overview
The MotorInterlock function block prevents a motor from starting when associated equipment is running, ensuring safe process coordination. The Function Block Diagram (FBD) is described below for implementation in an IEC 61131-3-compliant PLC environment (e.g., Codesys, Siemens TIA Portal).
FBD Structure
Inputs

Execute: BOOL
Enables the interlock logic (TRUE = Active).


Equipment1Running: BOOL
Status of Equipment 1 (TRUE = Running, FALSE = Stopped).


Equipment2Running: BOOL
Status of Equipment 2 (TRUE = Running, FALSE = Stopped).


Equipment3Running: BOOL
Status of Equipment 3 (TRUE = Running, FALSE = Stopped).



Outputs

AllowStart: BOOL
Permits motor startup (TRUE = Allowed, FALSE = Blocked).


Alarm: BOOL
Signals when startup is blocked due to running equipment.


Error: BOOL
Indicates invalid input conditions.


ErrorID: DWORD
Provides specific error codes (e.g., 0x80010000 for invalid inputs).



Internal Logic
The FBD consists of the following components, connected in sequence:

NOT Gates:

Three NOT gates (NOT1, NOT2, NOT3) invert the inputs Equipment1Running, Equipment2Running, and Equipment3Running, respectively.
Output of NOT1 = NOT Equipment1Running
Output of NOT2 = NOT Equipment2Running
Output of NOT3 = NOT Equipment3Running


AND Gate (AND1):

Inputs: Outputs of NOT1, NOT2, NOT3, and Execute.
Output: AllEquipmentStopped (TRUE if all equipment is stopped and Execute is TRUE).
Logic: AllEquipmentStopped := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running AND Execute


Output Assignment:

AllowStart is directly assigned the output of AND1: AllowStart := AllEquipmentStopped.
Alarm is set when Execute is TRUE and AllowStart is FALSE: Alarm := Execute AND NOT AllowStart.


Error Detection:

An OR gate (OR1) checks for invalid input states (e.g., undefined signals, assuming FALSE indicates valid stopped state).
Inputs to OR1: Check if any input is neither TRUE nor FALSE (simplified as FALSE for fail-safe).
Output: ErrorCondition.
Error is set: Error := ErrorCondition.
ErrorID is assigned: ErrorID := 16#80010000 if Error is TRUE, else 0.



FBD Textual Representation
[Execute] ----\
[Equipment1Running] --> [NOT1] ---\
[Equipment2Running] --> [NOT2] ----> [AND1] --> [AllowStart]
[Equipment3Running] --> [NOT3] ---/
                                 |
                                 |--> [NOT4] --> [AND2] --> [Alarm]
                                 |              /
[Execute] ----------------------/
                                 
[Equipment1Running] --\
[Equipment2Running] ----> [OR1] --> [Error]
[Equipment3Running] --/
                                 
[Error] --> [ErrorID := 16#80010000 if Error, else 0]

Notes

Visualization: In an FBD editor, place NOT gates for each equipment input, feed their outputs and Execute into an AND gate for AllowStart, and use an additional AND gate with Execute and inverted AllowStart for Alarm. Error logic uses an OR gate for input validation.
Fail-Safe: If Execute is FALSE or inputs are invalid, AllowStart defaults to FALSE, preventing motor startup.
HMI/SCADA: AllowStart, Alarm, Error, and ErrorID are logged and displayed on the HMI for operator awareness.

