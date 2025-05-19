Cause / Effect                  | Compute InterpolatedY | Set Error | Log Event | Set InterpolatedY to 0.0
--------------------------------|----------------------|-----------|-----------|-------------------------
Valid Inputs (N ≥ 3, X monotonic)| X                    |           | X         | 
Invalid Input (NaN, >1E6)       |                      | X (1)     | X         | X
Insufficient Points (N < 3)     |                      | X (2)     | X         | X
Non-Monotonic X                 |                      | X (1)     | X         | X
Numerical Overflow              |                      | X (3)     | X         | X
TargetX Out of Bounds           | X (clamped)          |           | X         | 

Legend:
- Compute InterpolatedY: Executes spline interpolation
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set InterpolatedY to 0.0: Sets InterpolatedY to 0.0
