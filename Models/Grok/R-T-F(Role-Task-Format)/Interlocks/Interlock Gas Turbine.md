Gas Turbine Interlock Protection System
Interlock List
The following table lists the 10 interlocks for the gas turbine, specifying the monitored parameter, threshold condition, protective action, priority, and status indicators. Each interlock is designed to prevent hazardous conditions, protect equipment, and ensure personnel safety.



Interlock ID
Monitored Parameter
Threshold Condition
Protective Action
Priority
Status



I-001
Exhaust Temperature
> 650°C
Reduce fuel flow; if persistent (>5s), initiate emergency shutdown
High
A, R


I-002
Turbine Speed
> 110% of rated speed (e.g., > 3960 rpm for 3600 rpm turbine)
Immediate emergency shutdown
High
A, R


I-003
Compressor Discharge Pressure
> 25 bar
Open bleed valve; if persistent (>10s), initiate emergency shutdown
High
A, R


I-004
Lubrication Oil Pressure
< 1.5 bar
Initiate auxiliary oil pump; if unresolved (>10s), initiate emergency shutdown
High
A, R


I-005
Vibration (Bearing)
> 7 mm/s RMS
Reduce load; if persistent (>15s), initiate emergency shutdown
High
A, R


I-006
Flame Failure
No flame detected (UV sensor signal < threshold)
Close fuel valve; initiate purge sequence; lockout until manual reset
High
A, R


I-007
Fuel Gas Pressure
< 5 bar
Switch to backup fuel (if available); if unresolved (>5s), initiate emergency shutdown
Medium
A


I-008
Cooling Water Flow
< 50% of nominal flow (e.g., < 100 L/min)
Reduce load; if unresolved (>10s), initiate emergency shutdown
Medium
A


I-009
Compressor Surge
Surge detected (pressure/flow oscillations)
Open anti-surge valve; if persistent (>5s), reduce compressor load
High
A, R


I-010
Manual Emergency Stop
Button pressed
Immediate emergency shutdown; lockout until manual reset
High
A, R


Annotations

Priority:
High: Immediate or rapid action required to prevent catastrophic failure (e.g., turbine damage, fire).
Medium: Action needed to maintain safe operation, less urgent but critical.


Status:
A (Alarm): Triggers an operator alarm via DCS/SCADA for awareness and logging.
R (Requires Reset): Requires manual acknowledgment/reset after condition clears to ensure operator verification.



Structured Text Code Snippets
Below are example IEC 61131-3 Structured Text snippets for selected interlocks, demonstrating implementation in a PLC or DCS environment.

(* Interlock I-001: Overtemperature Protection *)
IF Exhaust_Temp > 650.0 THEN
    Fuel_Flow_Setpoint := Fuel_Flow_Setpoint * 0.8; (* Reduce fuel flow *)
    Alarm_Active := TRUE;
    Alarm_ID := 1; (* Overtemperature *)
    Overtemp_Timer(IN := TRUE, PT := T#5000ms);
    Overtemp_Timer();
    IF Overtemp_Timer.Q THEN
        Emergency_Shutdown := TRUE;
        Lockout_Active := TRUE;
        LogAuditEntry('I-001: Overtemperature shutdown');
    END_IF;
ELSE
    Overtemp_Timer(IN := FALSE);
END_IF;

(* Interlock I-002: Overspeed Protection )IF Turbine_Speed > 3960.0 THEN    Emergency_Shutdown := TRUE;    Lockout_Active := TRUE;    Alarm_Active := TRUE;    Alarm_ID := 2; ( Overspeed *)    LogAuditEntry('I-002: Overspeed shutdown');END_IF;
(* Interlock I-006: Flame Failure Protection )IF Flame_Signal < FLAME_THRESHOLD THEN    Fuel_Valve_Close := TRUE;    Purge_Sequence_Start := TRUE;    Lockout_Active := TRUE;    Alarm_Active := TRUE;    Alarm_ID := 6; ( Flame Failure *)    LogAuditEntry('I-006: Flame failure, fuel valve closed');END_IF;
(* Interlock I-010: Manual Emergency Stop )IF Manual_EStop_Pressed THEN    Emergency_Shutdown := TRUE;    Lockout_Active := TRUE;    Alarm_Active := TRUE;    Alarm_ID := 10; ( Manual E-Stop *)    LogAuditEntry('I-010: Manual emergency stop activated');END_IF;
