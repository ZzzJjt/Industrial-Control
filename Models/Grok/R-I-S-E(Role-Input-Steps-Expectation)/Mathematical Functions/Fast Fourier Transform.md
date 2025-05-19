Cause / Effect                  | Compute FFT | Set Error | Log Event | Set MagnitudeSpectrum
--------------------------------|------------|-----------|-----------|----------------------
Valid Input Signal              | X          |           | X         | X
Invalid Input (NaN, >1E6)       |            | X (1)     | X         | Set to 0.0
Numerical Instability (Overflow)|            | X (2)     | X         | Set to 0.0

Legend:
- Compute FFT: Executes FFT algorithm
- Set Error: Sets Error := TRUE, ErrorID
- Log Event: Records event to AuditMessage
- Set MagnitudeSpectrum: Outputs computed magnitudes or 0.0
