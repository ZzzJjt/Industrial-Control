Cause / Effect                  | Run Conveyor | Extend Cylinder | Retract Cylinder | Reset Timer | Log Event
--------------------------------|--------------|-----------------|------------------|-------------|-----------
BottlePresent AND EmptyBottle   | X            | X               |                  |             | X
BottlePresent AND NOT EmptyBottle | X          |                 | X                | X           | X
No BottlePresent                | X            |                 | X                | X           | X
EjectTimer Done                 | X            |                 | X                | X           | X
Sensor Fault (Extendable)       | X            |                 | X                | X           | X

Legend:
- Run Conveyor: Sets ConveyorMotor := TRUE
- Extend Cylinder: Sets EjectCylinder := TRUE
- Retract Cylinder: Sets EjectCylinder := FALSE
- Reset Timer: Sets EjectTimer.IN := FALSE
- Log Event: Records event to AuditMessage
