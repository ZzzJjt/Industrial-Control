Cause / Effect                  | Start Fill | Run Mixer | Dispense | Stop All | Trigger Alarm | Log Event
--------------------------------|------------|-----------|----------|----------|---------------|-----------
Start Pressed, Valid Mode, Safe | X          |           |          |          |               | X
MixerLevelFull                  |            | X         |          |          |               | X
Mixer Timer Done                |            |           | X        |          |               | X
EmergencyStop                   |            |           |          | X        | X             | X
Invalid Mode (None/Both Selected)|           |           |          |          | X             | X
MixerFault/ValveError (Extendable)|          |           |          | X        | X             | X

Legend:
- Start Fill: Opens CoffeeValve and/or MilkValve
- Run Mixer: Sets Mixer := TRUE
- Dispense: Opens OutputValve
- Stop All: Sets all actuators FALSE, resets to IDLE
- Trigger Alarm: Sets AlarmActive := TRUE (extendable)
- Log Event: Records event to AuditMessage
