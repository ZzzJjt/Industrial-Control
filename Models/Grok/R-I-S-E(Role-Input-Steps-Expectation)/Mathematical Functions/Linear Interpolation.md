Cause / Effect                  | Compute Y | Set Error | Log Event | Set Y to Fallback
--------------------------------|-----------|-----------|-----------|-------------------
Valid Inputs (X2 ≠ X1)          | X         |           | X         | 
Division by Zero (X2 = X1)      |           | X (1)     | X         | X (Y1)
Invalid Input (NaN, >1E6)       |           | X (2)     | X         | X (0.0)
Numerical Overflow              |           | X (3)     | X         | X (0.0)

Legend:
- Compute Y: Executes interpolation formula
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set Y to Fallback: Sets Y to Y1 or 0.0
