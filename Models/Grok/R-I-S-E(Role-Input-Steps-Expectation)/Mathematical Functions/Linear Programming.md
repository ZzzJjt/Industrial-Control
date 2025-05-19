Cause / Effect                  | Compute Solution | Set Error | Log Event | Set Status | Set X to 0.0
--------------------------------|------------------|-----------|-----------|------------|--------------
Valid Inputs                    | X                |           | X         | 0 (Optimal)| 
Invalid Input (NaN, >1E6)       |                  | X (4)     | X         | 4          | X
Infeasible Problem              |                  | X (1)     | X         | 1          | X
Unbounded Problem               |                  | X (2)     | X         | 2          | X
Max Iterations Reached          |                  | X (3)     | X         | 3          | X

Legend:
- Compute Solution: Executes Simplex iterations
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set Status: Sets Status (0–4)
- Set X to 0.0: Clears solution vector
