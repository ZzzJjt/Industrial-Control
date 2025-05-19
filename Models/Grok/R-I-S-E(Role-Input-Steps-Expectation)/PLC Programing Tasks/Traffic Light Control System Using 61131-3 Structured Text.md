Cause / Effect                  | Set RedLight | Set GreenLight | Set YellowLight | Extend Red Phase | Clear PedestrianRequest | Reset Timer | Set Error | Log Event
--------------------------------|--------------|----------------|-----------------|------------------|-------------------------|-------------|-----------|-----------
TrafficState = 0 (Red)          | X            |                |                 |                  |                         |             |           | X
TrafficState = 1 (Green)        |              | X              |                 |                  |                         |             |           | X
TrafficState = 2 (Yellow)       |              |                | X               |                  | X                       | X           |           | X
PedestrianRequest, Red State    | X            |                |                 | X                |                         |             |           | X
StateTimer Done, Red, No Request|              | X              |                 |                  |                         | X           |           | X
StateTimer Done, Green          |              |                | X               |                  |                         | X           |           | X
StateTimer Done, Yellow         | X            |                |                 |                  | X                       | X           |           | X
EmergencyDetected               |              | X              |                 |                  | X                       | X           |           | X
Emergency Cleared               | X            |                |                 |                  | X                       | X           |           | X
Invalid State (Extendable)      | X            |                |                 |                  |                         | X           | X         | X

Legend:
- Set RedLight: Sets RedLight := TRUE, others FALSE
- Set GreenLight: Sets GreenLight := TRUE, others FALSE
- Set YellowLight: Sets YellowLight := TRUE, others FALSE
- Extend Red Phase: Sets PedestrianActive := TRUE, extends Red duration
- Clear PedestrianRequest: Sets PedestrianRequest := FALSE
- Reset Timer: Resets StateTimer.IN := FALSE
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
