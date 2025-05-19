Cause / Effect                  | Open Door | Close Door | Move Up | Move Down | Clear Request | Set Error | Log Event
--------------------------------|-----------|------------|---------|-----------|---------------|-----------|-----------
AtFloor[i] = TRUE               | X         |            |         |           | X             |           | X
Timer1 Done, No CabinRequest    | X         |            |         |           |               |           | X
Timer2 Done                     |           | X          |         |           |               |           | X
Timer1 Done, CabinRequest       |           | X          |         |           |               |           | X
DoorOpen = TRUE                 |           |            |         |           |               |           | X
Requests Above, GoingUp         |           |            | X       |           |               |           | X
Requests Below, GoingDown       |           |            |         | X         |               |           | X
No Requests in Direction        |           |            |         |           |               |           | X
No Requests                     |           |            |         |           |               |           | X
Multiple Floors Detected        |           |            |         |           |               | X (1)     | X
Invalid Inputs (NaN, >1E6)      |           |            |         |           |               | X (2)     | X

Legend:
- Open Door: Sets DoorOpen := TRUE
- Close Door: Sets DoorOpen := FALSE
- Move Up: Sets MoveUp := TRUE
- Move Down: Sets MoveDown := TRUE
- Clear Request: Clears CabinRequest[i], UpCall[i], DownCall[i]
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
