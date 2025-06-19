Cause / Effect                  | Compute Mean/StdDev | Set Error | Log Event | Set Outputs to 0.0
--------------------------------|--------------------|-----------|-----------|--------------------
Valid Input Array               | X                  |           | X         | 
Invalid Input (All Zeros, >1E6) |                    | X (1)     | X         | X
Overflow in Squared Differences |                    | X (2)     | X         | X (StdDev only)
Zero Variance                   | X (Mean only)      | X (3)     | X         | X (StdDev only)

Legend:
- Compute Mean/StdDev: Executes mean and standard deviation calculations
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set Outputs to 0.0: Sets Mean and/or StdDev to 0.0
