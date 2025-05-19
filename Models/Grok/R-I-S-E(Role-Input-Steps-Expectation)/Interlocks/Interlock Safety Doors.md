Cause / Effect                  | Prevent Startup | Emergency Shutdown | Trigger Alarm | Log Event
--------------------------------|-----------------|-------------------|---------------|-----------
Any Door Open (Startup)         | X               |                   | X             | X
Door 1 Opens (Running)          |                 | X                 | X             | X
Door 2 Opens (Running)          |                 | X                 | X             | X
Door 3 Opens (Running)          |                 | X                 | X             | X
Sensor Fault                    | X               | X                 | X             | X
Emergency Stop Activated        | X               | X                 | X             | X

Legend:
- Prevent Startup: Blocks ReactorRunning := TRUE
- Emergency Shutdown: Sets EMERGENCY_SHUTDOWN := TRUE, ReactorRunning := FALSE
- Trigger Alarm: Activates operator alarm
- Log Event: Records event to HMI/log
