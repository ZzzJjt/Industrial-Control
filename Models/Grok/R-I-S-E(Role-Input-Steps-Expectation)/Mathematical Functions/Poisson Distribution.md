Cause / Effect                  | Compute Probability | Set Error | Log Event | Set Probability to 0.0
--------------------------------|--------------------|-----------|-----------|------------------------
Valid Inputs (λ > 0, k ≥ 0)     | X                  |           | X         | 
Invalid Input (λ ≤ 0, k < 0)    |                    | X (1)     | X         | X
Overflow (e⁻λ, λᵏ, result)      |                    | X (2)     | X         | X
Factorial Overflow (k > 20)     |                    | X (3)     | X         | X

Legend:
- Compute Probability: Executes Poisson formula
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set Probability to 0.0: Sets Probability to 0.0
