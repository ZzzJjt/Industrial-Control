Cause / Effect                  | Compute MatrixC | Set Error | Log Event | Set MatrixC to 0.0
--------------------------------|----------------|-----------|-----------|--------------------
Valid Inputs                    | X              |           | X         | 
Invalid Input (NaN, >1E6)       |                | X (1)     | X         | X
Numerical Overflow              |                | X (2)     | X         | X

Legend:
- Compute MatrixC: Executes matrix multiplication
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set MatrixC to 0.0: Clears output matrix
