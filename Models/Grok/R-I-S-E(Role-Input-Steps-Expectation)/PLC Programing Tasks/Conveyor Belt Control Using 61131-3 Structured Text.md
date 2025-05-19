Cause / Effect                  | Run Conveyor | Stop Conveyor | Set Error | Log Event
--------------------------------|--------------|---------------|-----------|-----------
Any Station Stop (1, 2, or 3)   |              | X             |           | X
AutoMode, All Sensors TRUE      | X            |               |           | X
AutoMode, Any Sensor FALSE      |              | X             |           | X
ManualMode, No Station Stop     | X            |               |           | X
ManualMode, Any Station Stop    |              | X             |           | X
Invalid Mode (Both/Neither)     |              | X             | X (1)     | X
Sensor Fault (Extendable)       |              | X             | X (2)     | X

Legend:
- Run Conveyor: Sets ConveyorRunning := TRUE
- Stop Conveyor: Sets ConveyorRunning := FALSE
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
