Cause / Effect                  | Set M20 | Reset M20 | Set M30 | Reset M30 | Red Lights On | Green Lights On | Set Error | Log Event
--------------------------------|---------|-----------|---------|-----------|---------------|-----------------|-----------|-----------
M1 Pulse (Ground Entry at X1)   | X       |           |         |           | X             |                 |           | X
M4 Pulse (Ground Exit at X2)    | X       | X         |         |           | X             |                 |           | X
M2 Pulse (Basement Exit at X1)  |         |           | X       | X         | X             |                 |           | X
M3 Pulse (Basement Entry at X2) |         |           | X       |           | X             |                 |           | X
M20 OR M30 = TRUE               |         |           |         |           | X             |                 |           | X
M20 AND M30 = FALSE             |         |           |         |           |               | X               |           | X
Conflicting Pulses (e.g., M1 AND M2) |    |           |         |           | X             |                 | X (1)     | X
Sensor Fault (Extendable)       |         |           |         |           | X             |                 | X (2)     | X

Legend:
- Set M20: Sets M20 := TRUE
- Reset M20: Sets M20 := FALSE
- Set M30: Sets M30 := TRUE
- Reset M30: Sets M30 := FALSE
- Red Lights On: Sets Y1 := TRUE, Y2 := FALSE
- Green Lights On: Sets Y1 := FALSE, Y2 := TRUE
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
