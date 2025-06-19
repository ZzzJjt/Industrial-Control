Cause / Effect                  | Compute LnX | Set Error | Log Event | Set LnX to 0.0
--------------------------------|-------------|-----------|-----------|----------------
Valid Input (X > 0)             | X           |           | X         | 
Non-Positive Input (X ≤ 0)      |             | X (1)     | X         | X
Invalid Input (NaN, >1E6)       |             | X (2)     | X         | X

Legend:
- Compute LnX: Executes LN(X)
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set LnX to 0.0: Sets LnX to 0.0
