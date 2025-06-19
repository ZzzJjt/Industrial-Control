✅ Structure and Function of the OSCAT PID Function Block

⸻

🟡 1. Inputs

Input
Type
Description
SP
REAL
Setpoint – the desired target value of the process.
PV
REAL
Process Variable – the current measured value.
KP
REAL
Proportional gain.
KI
REAL
Integral gain (per second).
KD
REAL
Derivative gain (per second).
MAN
BOOL
Manual mode switch – when TRUE, the controller does not act automatically.
MAN_IN
REAL
Manual control input – used when MAN = TRUE.
LIMIT_HI
REAL
Upper limit for output.
LIMIT_LO
REAL
Lower limit for output.
CYCLE
TIME
Task cycle time (e.g., T#100ms).
EN
BOOL
Enable – controller runs when TRUE.
RST
BOOL
Reset – resets integral and derivative memory.
