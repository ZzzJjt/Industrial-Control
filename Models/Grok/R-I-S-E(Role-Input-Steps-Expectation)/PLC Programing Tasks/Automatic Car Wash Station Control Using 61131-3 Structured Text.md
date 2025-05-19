Cause / Effect                  | Start Wash | Stop Wash | Activate Alarm | Lock SafeToRun | Reset SafeToRun | Log Event
--------------------------------|------------|-----------|----------------|----------------|-----------------|-----------
Car Present, No Human, SafeToRun| X          |           |                |                |                 | X
Human Detected                  |            | X         | X              | X              |                 | X
No Car, No Human, Wash Inactive |            |           |                |                | X               | X
Car Present, Human Detected     |            | X         | X              | X              |                 | X
Sensor Fault (Extendable)       |            | X         | X              | X              |                 | X

Legend:
- Start Wash: Sets WashActive := TRUE
- Stop Wash: Sets WashActive := FALSE
- Activate Alarm: Sets AlarmActive := TRUE
- Lock SafeToRun: Sets SafeToRun := FALSE
- Reset SafeToRun: Sets SafeToRun := TRUE
- Log Event: Records event to AuditMessage
