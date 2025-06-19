IEC 61131-3 Structured Text Programming Reference
Introduction
This reference guide provides a comprehensive overview of the IEC 61131-3 Structured Text (ST) language, a high-level, text-based programming language for industrial automation and PLCs. Designed for automation engineers and programmers, it includes all reserved keywords, standard data types, key function blocks, and code examples to support development, debugging, and learning. The guide is organized for quick lookup, onboarding new team members, and ensuring standards-compliant, maintainable code across automation projects.
1. Reserved Keywords
IEC 61131-3 defines reserved keywords for control flow, data types, constants, operators, and program organization. These are case-insensitive and cannot be used as identifiers.
1.1 Control Flow Keywords



Keyword
Description



IF
Conditional statement for branching (e.g., IF condition THEN ... END_IF).


THEN
Marks the start of the IF condition’s execution block.


ELSE
Specifies alternative execution block if IF condition is false.


ELSIF
Additional conditional branch within an IF statement.


CASE
Multi-way branching based on a selector value (e.g., CASE state OF ...).


OF
Defines case labels in a CASE statement.


FOR
Loop with a counter (e.g., FOR i := 1 TO 10 DO ... END_FOR).


TO
Specifies the upper bound in a FOR loop.


BY
Defines the step size in a FOR loop (e.g., BY 2).


DO
Marks the start of a loop’s execution block.


WHILE
Loop that continues while a condition is true (e.g., WHILE cond DO ...).


REPEAT
Loop that executes until a condition is true (e.g., REPEAT ... UNTIL).


UNTIL
Specifies the termination condition for a REPEAT loop.


EXIT
Exits a loop immediately (e.g., breaks out of FOR, WHILE, or REPEAT).


RETURN
Exits a function, function block, or program, optionally returning a value.


Example: Control Flow
VAR
    state : INT := 0;
    counter : INT;
END_VAR

IF state = 0 THEN
    state := 1;
ELSIF state = 1 THEN
    state := 2;
ELSE
    state := 0;
END_IF;

CASE state OF
    0: counter := 0;
    1: counter := counter + 1;
    2: counter := counter + 10;
END_CASE;

FOR counter := 1 TO 5 BY 1 DO
    IF counter = 3 THEN
        EXIT; // Break loop
    END_IF;
END_FOR;

1.2 Data Type Keywords



Keyword
Description



BOOL
Boolean value (TRUE or FALSE).


BYTE
8-bit unsigned integer (0 to 255).


WORD
16-bit unsigned integer (0 to 65,535).


DWORD
32-bit unsigned integer (0 to 4,294,967,295).


LWORD
64-bit unsigned integer.


SINT
8-bit signed integer (-128 to 127).


INT
16-bit signed integer (-32,768 to 32,767).


DINT
32-bit signed integer (-2,147,483,648 to 2,147,483,647).


LINT
64-bit signed integer.


USINT
8-bit unsigned integer (0 to 255).


UINT
16-bit unsigned integer (0 to 65,535).


UDINT
32-bit unsigned integer (0 to 4,294,967,295).


ULINT
64-bit unsigned integer.


REAL
32-bit floating-point number (IEEE 754 single precision).


LREAL
64-bit floating-point number (IEEE 754 double precision).


TIME
Duration (e.g., T#5s for 5 seconds).


DATE
Calendar date (e.g., D#2025-05-19).


TIME_OF_DAY
Time of day (e.g., TOD#10:57:00).


DATE_AND_TIME
Combined date and time (e.g., DT#2025-05-19-10:57:00).


STRING
Variable-length character string (e.g., 'Hello').


WSTRING
Wide-character string (Unicode support).


ARRAY
Multi-dimensional array of any data type (e.g., ARRAY[0..9] OF INT).


ENUM
Enumerated type with named values (e.g., STATE : (IDLE, RUNNING)).


STRUCT
User-defined structure combining multiple data types.


ANY
Generic type for polymorphic operations (vendor-specific usage).


Example: Data Types
VAR
    isActive : BOOL := FALSE;
    counter : DINT := 0;
    temperature : REAL := 25.5;
    duration : TIME := T#10s;
    status : STRING := 'Idle';
    states : ARRAY[0..2] OF INT := [0, 1, 2];
    TYPE MachineState : (IDLE, RUNNING, STOPPED); END_TYPE
    machine : MachineState := IDLE;
    TYPE Config : STRUCT
        maxTemp : REAL;
        minTemp : REAL;
    END_TYPE
    config : Config;
END_VAR

config.maxTemp := 100.0;
config.minTemp := 0.0;
IF machine = IDLE THEN
    status := 'Machine Stopped';
END_IF;

1.3 Constants and Operators



Keyword
Description



TRUE
Boolean constant for true.


FALSE
Boolean constant for false.


NULL
Null pointer or undefined value (vendor-specific).


AND
Logical AND operator (e.g., A AND B).


OR
Logical OR operator (e.g., A OR B).


XOR
Logical exclusive OR operator (e.g., A XOR B).


NOT
Logical negation (e.g., NOT A).


MOD
Modulo operator (e.g., 10 MOD 3 = 1).


**
Exponentiation (e.g., 2 ** 3 = 8).


=
Equality comparison (e.g., A = B).


<>
Inequality comparison (e.g., A <> B).


<
Less than (e.g., A < B).


>
Greater than (e.g., A > B).


<=
Less than or equal to (e.g., A <= B).


>=
Greater than or equal to (e.g., A >= B).


Example: Operators
VAR
    a : INT := 5;
    b : INT := 3;
    result : BOOL;
    remainder : INT;
END_VAR

result := (a > b) AND (a MOD 2 = 1); // TRUE
remainder := a MOD b; // 2
IF a ** 2 >= 25 THEN
    result := TRUE; // 5^2 = 25
END_IF;

1.4 Program Organization Keywords



Keyword
Description



VAR
Declares variables (e.g., VAR x : INT; END_VAR).


VAR_INPUT
Declares input variables for FBs or functions.


VAR_OUTPUT
Declares output variables for FBs or functions.


VAR_IN_OUT
Declares variables that are both input and output.


VAR_GLOBAL
Declares global variables accessible across programs.


VAR_EXTERNAL
Declares external variables defined elsewhere.


VAR_TEMP
Declares temporary variables within a scope.


VAR_CONSTANT
Declares constant values (e.g., VAR_CONSTANT PI : REAL := 3.14159;).


FUNCTION
Defines a function that returns a single value (e.g., FUNCTION Add : INT).


FUNCTION_BLOCK
Defines a function block with internal state (e.g., FUNCTION_BLOCK FB).


PROGRAM
Defines a main program unit (e.g., PROGRAM Main).


END_VAR
Closes a variable declaration block.


END_IF
Closes an IF statement.


END_CASE
Closes a CASE statement.


END_FOR
Closes a FOR loop.


END_WHILE
Closes a WHILE loop.


END_REPEAT
Closes a REPEAT loop.


END_FUNCTION
Closes a function definition.


END_FUNCTION_BLOCK
Closes a function block definition.


END_PROGRAM
Closes a program definition.


Example: Program Organization
PROGRAM BatchControl
VAR
    start : BOOL;
    timer : TON;
END_VAR

FUNCTION_BLOCK ProcessStep
VAR_INPUT
    duration : TIME;
END_VAR
VAR_OUTPUT
    complete : BOOL;
END_VAR
VAR
    stepTimer : TON;
END_VAR
    stepTimer(IN := TRUE, PT := duration);
    complete := stepTimer.Q;
END_FUNCTION_BLOCK

2. Standard Function Blocks
The following are commonly used standard function blocks in IEC 61131-3, critical for automation tasks like timing, counting, and triggering.
2.1 TON (Timer On Delay)

Purpose: Delays the output (Q) turning TRUE until the input (IN) has been TRUE for a specified period (PT).
Inputs:
IN (BOOL): Timer enable.
PT (TIME): Preset time duration.


Outputs:
Q (BOOL): TRUE when PT is reached.
ET (TIME): Elapsed time.


Usage: Timing process steps (e.g., valve open for 10 seconds).

Example:
VAR
    timer : TON;
    valveOpen : BOOL;
END_VAR

timer(IN := valveOpen, PT := T#10s);
IF timer.Q THEN
    valveOpen := FALSE; // Close valve after 10s
END_IF;

2.2 TOF (Timer Off Delay)

Purpose: Delays the output (Q) turning FALSE after the input (IN) turns FALSE for a specified period (PT).
Inputs:
IN (BOOL): Timer enable.
PT (TIME): Preset time duration.


Outputs:
Q (BOOL): TRUE until PT elapses after IN goes FALSE.
ET (TIME): Elapsed time.


Usage: Extending actuator operation after a trigger (e.g., fan runs after motor stops).

Example:
VAR
    fanTimer : TOF;
    motorOn : BOOL;
    fanOn : BOOL;
END_VAR

fanTimer(IN := motorOn, PT := T#5s);
fanOn := fanTimer.Q; // Fan stays on 5s after motor stops

2.3 TP (Timer Pulse)

Purpose: Generates a pulse of specified duration (PT) when the input (IN) transitions to TRUE.
Inputs:
IN (BOOL): Trigger pulse.
PT (TIME): Pulse duration.


Outputs:
Q (BOOL): TRUE for PT duration.
ET (TIME): Elapsed time.


Usage: Triggering a one-shot action (e.g., brief alarm signal).

Example:
VAR
    pulseTimer : TP;
    button : BOOL;
    alarm : BOOL;
END_VAR

pulseTimer(IN := button, PT := T#2s);
alarm := pulseTimer.Q; // Alarm sounds for 2s on button press

2.4 CTU (Counter Up)

Purpose: Increments a counter (CV) each time the input (CU) transitions to TRUE, up to a preset value (PV).
Inputs:
CU (BOOL): Count up trigger.
R (BOOL): Reset counter.
PV (INT): Preset value.


Outputs:
Q (BOOL): TRUE when CV >= PV.
CV (INT): Current count.


Usage: Counting parts on a conveyor.

Example:
VAR
    counter : CTU;
    sensor : BOOL;
    reset : BOOL;
    batchDone : BOOL;
END_VAR

counter(CU := sensor, R := reset, PV := 100);
batchDone := counter.Q; // TRUE when 100 parts counted

2.5 R_TRIG (Rising Edge Trigger)

Purpose: Detects a rising edge on the input (CLK), setting output (Q) TRUE for one cycle.
Inputs:
CLK (BOOL): Input signal.


Outputs:
Q (BOOL): TRUE for one cycle on rising edge.


Usage: Detecting button presses or sensor activations.

Example:
VAR
    trigger : R_TRIG;
    button : BOOL;
    event : BOOL;
END_VAR

trigger(CLK := button);
event := trigger.Q; // TRUE for one cycle when button is pressed

3. Standard Functions
The following are key standard functions for mathematical and logical operations, commonly used in ST.
3.1 ABS

Purpose: Returns the absolute value of a number.
Input: IN (ANY_NUM, e.g., INT, REAL).
Output: Absolute value (same type as input).
Usage: Ensuring positive values (e.g., pressure readings).

Example:
VAR
    pressure : REAL := -10.5;
    absPressure : REAL;
END_VAR

absPressure := ABS(pressure); // 10.5

3.2 SIN

Purpose: Computes the sine of an angle (in radians).
Input: IN (REAL, angle in radians).
Output: REAL (-1.0 to 1.0).
Usage: Modeling oscillatory behavior (e.g., robotic arm motion).

Example:
VAR
    angle : REAL := 1.5708; // ~π/2
    sineValue : REAL;
END_VAR

sineValue := SIN(angle); // ~1.0

3.3 CONCAT

Purpose: Concatenates two strings.
Inputs: IN1, IN2 (STRING).
Output: Concatenated STRING.
Usage: Building log messages or HMI displays.

Example:
VAR
    prefix : STRING := 'Temp: ';
    value : STRING := '25.5';
    message : STRING;
END_VAR

message := CONCAT(prefix, value); // 'Temp: 25.5'

4. Practical Code Snippets
4.1 Batch Process Control
Controls a simple batch process with states (Idle, Running, Complete) using a timer.
TYPE BatchState : (IDLE, RUNNING, COMPLETE); END_TYPE

VAR
    state : BatchState := IDLE;
    start : BOOL;
    timer : TON;
    complete : BOOL;
END_VAR

CASE state OF
    IDLE:
        complete := FALSE;
        timer(IN := FALSE);
        IF start THEN
            state := RUNNING;
        END_IF;
    RUNNING:
        timer(IN := TRUE, PT := T#30s);
        IF timer.Q THEN
            state := COMPLETE;
        END_IF;
    COMPLETE:
        complete := TRUE;
        timer(IN := FALSE);
        IF NOT start THEN
            state := IDLE;
        END_IF;
END_CASE;

4.2 PID-Like Temperature Control
Implements a basic proportional control loop for temperature regulation.
VAR
    tempPV : REAL;          // Process variable (°C)
    tempSP : REAL := 75.0;  // Setpoint (°C)
    Kp : REAL := 2.0;       // Proportional gain
    error : REAL;
    output : REAL;          // Control output (0.0 to 100.0%)
END_VAR

error := tempSP - tempPV;
output := Kp * error;

// Clamp output
IF output > 100.0 THEN
    output := 100.0;
ELSIF output < 0.0 THEN
    output := 0.0;
END_IF;

4.3 Conveyor Part Counting
Counts parts on a conveyor and signals when a batch is complete.
VAR
    sensor : BOOL;
    reset : BOOL;
    counter : CTU;
    batchDone : BOOL;
END_VAR

counter(CU := sensor, R := reset, PV := 50);
batchDone := counter.Q; // TRUE when 50 parts counted

5. Usage Notes

Standards Compliance: Use keywords and FBs as defined in IEC 61131-3 to ensure portability across PLC platforms (e.g., CODESYS, TwinCAT, Siemens).
Vendor Extensions: Some PLCs add non-standard functions (e.g., Siemens’ PID_Compact). Verify compatibility before use.
Debugging: Use clear variable names and comments. Test loops and timers in simulation to avoid infinite loops or timing errors.
Performance: Minimize complex loops (e.g., nested FOR) in tight scan cycles (<10 ms) to maintain determinism.
Documentation: Align code with project standards, using VAR sections and comments for clarity.

6. Conclusion
This reference guide consolidates IEC 61131-3 Structured Text’s keywords, data types, function blocks, and functions, with practical examples for automation tasks. It serves as a quick lookup for developers, a training tool for new team members, and a foundation for consistent, standards-compliant code. Exportable as a PDF or hosted on a wiki, it supports development, debugging, and project scalability in industrial automation.
For vendor-specific details or additional examples, consult your PLC’s documentation or contact the automation team.
