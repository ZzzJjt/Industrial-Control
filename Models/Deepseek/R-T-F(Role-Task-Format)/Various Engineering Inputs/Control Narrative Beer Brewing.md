Section 4 – Lautering

Equipment:
- Lauter Tun: LT-401 (level transmitter)
- Rake Motor: RM-401
- Flow Meter: FT-402 (wort outlet)
- Sparge Pump: SP-402
- Sparge Water Temperature Sensor: TT-403
- Turbidity Meter: TU-404 (wort outlet)

Instrumentation:
- LT-401: Lauter tun level transmitter
- FT-402: Wort outlet flow transmitter
- TT-403: Sparge water temperature sensor
- TU-404: Turbidity meter at wort outlet

Control Sequence:

1. Initiate Recirculation:
   - Action: Start the rake motor to stir the grain bed.
   - Control Logic: Set RecirculateStart = TRUE.
   - Instrumentation: Use TU-404 to monitor turbidity.

2. Monitor Turbidity Until It Falls Below 200 NTU:
   - Action: Continue recirculating until the turbidity drops below 200 NTU.
   - Control Logic: If TU-404.Value < 200 NTU, set RecirculateStop = TRUE.
   - Interlock: Ensure the rake motor continues running until the turbidity threshold is met.

3. Begin Wort Transfer to Kettle While Initiating Sparge Water at 76°C:
   - Action: Start transferring wort to the kettle while initiating sparging with 76°C water.
   - Control Logic: Set LauterStart = TRUE. Open wort transfer valve and start sparge pump.
   - Instrumentation: Use TT-403 to ensure sparge water temperature is within ±1°C of 76°C.

4. Maintain Level in Lauter Tun (60–80%):
   - Action: Adjust sparge rate to maintain the liquid level between 60% and 80% of the lauter tun capacity.
   - Control Logic: If LT-401.Value < 60%, increase sparge rate. If LT-401.Value > 80%, decrease sparge rate.
   - Interlock: Ensure the sparge rate does not exceed safe limits to prevent overflow or underflow.

5. Stop Lautering When Flow < 0.5 L/min or Target Volume Reached:
   - Action: Stop lautering when the flow rate falls below 0.5 L/min or the target volume is reached.
   - Control Logic: If FT-402.Value < 0.5 L/min OR VolumeCollected >= TargetVolume, set LauterStop = TRUE.
   - Interlock: Close wort transfer valve and stop sparge pump.

Outputs:
- LauterStart: Boolean flag to initiate lautering.
- LauterStop: Boolean flag to stop lautering.
- DivertCloudyWort: Boolean flag to divert cloudy wort if turbidity exceeds a certain threshold during recirculation.

Comments and Logic Interlocks:
- Recirculation Monitoring: Ensure the rake motor runs continuously until the turbidity threshold is met to achieve clear wort.
- Level Control: Maintain the lauter tun level between 60% and 80% to optimize wort extraction and prevent overflow.
- Flow Rate Control: Monitor and adjust the flow rate to ensure consistent wort collection and prevent premature stopping.
- Safety Interlocks: Implement safety interlocks to prevent damage to equipment due to excessive flow rates or incorrect temperatures.
