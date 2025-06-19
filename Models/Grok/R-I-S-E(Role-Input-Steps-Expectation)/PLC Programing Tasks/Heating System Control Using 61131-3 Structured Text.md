Cause / Effect                  | Compute AvgTemp | Set SensorFault | Turn Heating On | Turn Heating Off | Set Error | Log Event
--------------------------------|-----------------|-----------------|-----------------|------------------|-----------|-----------
Valid Sensor Readings (10–30°C)  | X               |                 | X (if AvgTemp < 20) | X (if AvgTemp > 22) |           | X
Sensor Out of Range (<10 or >30) | X               | X               |                 | X                |           | X
Invalid Input (NaN, >1E6)       |                 | X               |                 | X                | X (1)     | X
AvgTemp in Range (20–22°C)      | X               |                 |                 |                  |           | X
Sensor Fault (Extendable)       |                 | X               |                 | X                | X (2)     | X

Legend:
- Compute AvgTemp: Calculates average of TempSensor1, TempSensor2, TempSensor3
- Set SensorFault: Sets SensorFault := TRUE
- Turn Heating On: Sets HeatingOn := TRUE
- Turn Heating Off: Sets HeatingOn := FALSE
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
