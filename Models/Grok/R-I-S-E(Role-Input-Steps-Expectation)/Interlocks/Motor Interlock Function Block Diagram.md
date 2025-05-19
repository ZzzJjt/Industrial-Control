Cause / Effect                  | Prevent Motor Start | Trigger Alarm | Log Event
--------------------------------|--------------------|---------------|-----------
Equipment 1 Running             | X                  | X             | X
Equipment 2 Running             | X                  | X             | X
Equipment 3 Running             | X                  | X             | X
Sensor Fault                    | X                  | X             | X
Normal (All Equipment Stopped)  |                    |               | X

Legend:
- Prevent Motor Start: Sets AllowStart := FALSE
- Trigger Alarm: Activates operator alarm
- Log Event: Records event to HMI/log
