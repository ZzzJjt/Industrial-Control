Cause / Effect                  | Set ManualMode | Set AutoMode | Execute Clip | Execute Transfer | Execute Release | Reset Timer/State | Set Error | Log Event
--------------------------------|----------------|--------------|--------------|------------------|-----------------|-------------------|-----------|-----------
BtnManual Pressed               | X              |              |              |                  |                 |                   |           | X
BtnAuto Pressed                 |                | X            |              |                  |                 |                   |           | X
Both BtnManual and BtnAuto      |                |              |              |                  |                 |                   | X (1)     | X
ManualMode, CmdClip             |                |              | X            |                  |                 |                   |           | X
ManualMode, CmdTransfer         |                |              |              | X                |                 |                   |           | X
ManualMode, CmdRelease          |                |              |              |                  | X               |                   |           | X
AutoMode, BtnAuto Rising Edge   |                |              | X (State 0)  |                  |                 |                   |           | X
AutoMode, State 1, Timer Running|                |              |              | X                |                 |                   |           | X
AutoMode, State 1, Timer Done   |                |              |              |                  | X (State 2)     |                   |           | X
AutoMode, State 2 Complete      |                |              |              |                  |                 | X                 |           | X
Sensor Fault (Extendable)       |                |              |              |                  |                 |                   | X (2)     | X

Legend:
- Set ManualMode: Sets ManualMode := TRUE, AutoMode := FALSE
- Set AutoMode: Sets AutoMode := TRUE, ManualMode := FALSE
- Execute Clip: Sets Clip := TRUE
- Execute Transfer: Sets Transfer := TRUE
- Execute Release: Sets Release := TRUE
- Reset Timer/State: Resets AutoTimer, State, AutoTrigger
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
