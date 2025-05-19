Cause / Effect                  | Open FlowValve | Close FlowValve | Open ReliefValve | Set FlowError | Set PressureError | Set Error | Log Event
--------------------------------|----------------|-----------------|------------------|---------------|------------------|-----------|-----------
FlowInput < 50 SLPM             | X              |                 |                  |               |                  |           | X
FlowInput ≥ 50 SLPM             |                | X               |                  |               |                  |           | X
PressureInput < 5.5 or > 6.0    |                |                 | X                |               | X                |           | X
PressureInput in 5.5–6.0        |                |                 |                  |               |                  |           | X
ABS(FlowInput - 50) > 5.0       |                |                 |                  | X             |                  |           | X
ABS(FlowInput - 50) ≤ 5.0       |                |                 |                  |               |                  |           | X
Invalid Input (NaN, >1E6)       |                | X               |                  |               | X                | X (1)     | X
Sensor Fault (Extendable)       |                | X               | X                |               | X                | X (2)     | X

Legend:
- Open FlowValve: Sets FlowValveOutput := TRUE
- Close FlowValve: Sets FlowValveOutput := FALSE
- Open ReliefValve: Sets PressureReliefValve := TRUE
- Set FlowError: Sets FlowError := TRUE
- Set PressureError: Sets PressureError := TRUE
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
