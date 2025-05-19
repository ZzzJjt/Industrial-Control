Cause / Effect                  | Set AllowStart | Trigger Alarm | Log Event | Set Error | Set Converged
--------------------------------|---------------|---------------|-----------|-----------|---------------
Convergence Achieved            | X             |               | X         |           | X (TRUE)
Non-Convergence (Max Iterations)|               | X             | X         | X (1)     | X (FALSE)
Numerical Instability (Norm > 1E6)|              | X             | X         | X (2)     | X (FALSE)
Invalid Matrix Input            |               | X             | X         | X (2)     | X (FALSE)

Legend:
- Set AllowStart: Sets EigenValues and Done := TRUE
- Trigger Alarm: Activates operator alarm
- Log Event: Records event to AuditMessage
- Set Error: Sets Error := TRUE, ErrorID
- Set Converged: Sets Converged := TRUE/FALSE
