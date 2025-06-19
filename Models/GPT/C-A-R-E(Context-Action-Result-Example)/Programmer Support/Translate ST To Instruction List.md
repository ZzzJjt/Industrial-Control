// --- Mode Selection ---
LD ManualButton
JMPC SetManual

LD AutoButton
JMPC SetAuto
JMP ModeCheck

SetManual:
LD 0
ST Mode
JMP ModeCheck

SetAuto:
LD 1
ST Mode

ModeCheck:
// --- If Mode = 1, proceed with auto sequence ---
LD Mode
EQ 1
JMPC AutoStart
JMP EndCycle

AutoStart:
// CASE Step OF

LD Step
EQ 0
JMPC Step0

LD Step
EQ 1
JMPC Step1

LD Step
EQ 2
JMPC Step2

LD Step
EQ 3
JMPC Step3

Step0:
LD TRUE
ST ConveyorRun
LD SensorAtPick
JMPC ToStep1
JMP EndCycle

ToStep1:
LD 1
ST Step
JMP EndCycle

Step1:
LD FALSE
ST ConveyorRun
LD TRUE
ST ArmPick
LD ArmPickDone
JMPC ToStep2
JMP EndCycle

ToStep2:
LD FALSE
ST ArmPick
LD 2
ST Step
JMP EndCycle

Step2:
LD TRUE
ST ArmPlace
LD ArmPlaceDone
JMPC ToStep3
JMP EndCycle

ToStep3:
LD FALSE
ST ArmPlace
LD 3
ST Step
JMP EndCycle

Step3:
LD FALSE
ST ArmPlace
LD 0
ST Step

EndCycle:
NOP
