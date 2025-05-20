Ammonium Nitrate Prilling Station Interlock System
Interlock List
The following list details 10 interlocks for the safe operation of an ammonium nitrate prilling station, specifying the hazardous condition, trigger threshold, safety action, priority, and status indicators. These interlocks mitigate risks associated with ammonium nitrate’s reactivity, ensuring operator safety, environmental protection, and regulatory compliance.

Overtemperature in Prill Tower

Condition: Excessive temperature in the prill tower, risking decomposition or explosion.
Trigger: TT-101 (tower temperature transmitter) > 200°C.
Action: Stop melt feed (close FV-101), reduce prill tower fan speed, sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


High Pressure in Melt System

Condition: Overpressure in the melt system, risking pipe rupture or explosion.
Trigger: PT-101 (melt system pressure transmitter) > 10 bar.
Action: Open relief valve (PRV-101), stop melt pump (P-101), sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


Cooling Air Failure

Condition: Loss of cooling air, causing overheating and potential decomposition.
Trigger: FT-101 (cooling air flow transmitter) < 500 m³/h.
Action: Stop melt feed (close FV-101), initiate tower shutdown, sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


Melt Pump or Feeder Failure

Condition: Failure of melt pump or feeder, causing flow disruption or pressure buildup.
Trigger: P-101 motor fault (e.g., no current feedback) or FV-101 feedback mismatch.
Action: Stop melt feed (close FV-101), initiate tower shutdown, sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


High Product Accumulation

Condition: Excessive prill accumulation at tower base, risking blockages or dust explosion.
Trigger: LT-101 (base level transmitter) > 80% capacity.
Action: Stop melt feed (close FV-101), increase conveyor speed (C-101), sound alarm.
Priority: Medium
Status: Alarm (A)


Scrubber Failure or Bypass

Condition: Failure of exhaust scrubber, risking release of ammonium nitrate dust or fumes.
Trigger: Scrubber flow < 200 m³/h or bypass valve open (XV-101 feedback).
Action: Stop melt feed (close FV-101), initiate tower shutdown, sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


Emergency Stop Trigger

Condition: Operator-initiated stop for unforeseen hazards (e.g., fire, personnel safety).
Trigger: Emergency stop button (E-Stop-101) pressed.
Action: Immediate tower shutdown, close all valves (FV-101, PRV-101), sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


Loss of Inerting Gas

Condition: Loss of nitrogen or inert gas, increasing explosion risk in dust-rich areas.
Trigger: FT-102 (inert gas flow transmitter) < 50 m³/h.
Action: Stop melt feed (close FV-101), initiate tower shutdown, sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


Power Failure

Condition: Loss of power, risking uncontrolled process conditions or ventilation failure.
Trigger: Power supply monitor (PS-101) detects loss of main power.
Action: Switch to backup power (UPS-101), close all valves, initiate tower shutdown if unresolved (>10s), sound alarm.
Priority: High
Status: Alarm (A), Manual Reset (R)


Explosion Vent Activation

Condition: Activation of explosion vent, indicating a deflagration or pressure surge.
Trigger: Explosion vent sensor (XS-101) detects vent open.
Action: Immediate tower shutdown, close all valves, sound alarm, lockout until inspected.
Priority: High
Status: Alarm (A), Manual Reset (R)



System Logic: Activation and Reset

Activation:

Interlocks are triggered when sensor readings exceed thresholds (e.g., PT-101 > 10 bar) or fault conditions are detected (e.g., P-101 motor fault).
High-priority interlocks (e.g., overtemperature, high pressure) act immediately, closing valves or initiating shutdown within one PLC cycle (~100ms).
Medium-priority interlocks (e.g., high product accumulation) may include a short delay (e.g., 5s) to confirm persistence, avoiding nuisance trips.
All interlocks trigger alarms to the DCS/SCADA system, logged for traceability.


Reset:

Manual Reset (R): High-priority interlocks (e.g., overtemperature, emergency stop) latch the tripped state, requiring manual reset via HMI or physical button after the condition clears (e.g., TT-101 < 180°C) and operator verification (e.g., maintenance check). This ensures no automatic restart without confirming safety.
Automatic Reset: Medium-priority interlocks (e.g., high product accumulation) may reset automatically if the condition clears (e.g., LT-101 < 70%) and no faults persist, to minimize downtime.
Reset logic checks sensor validity and equipment status (e.g., no valve faults) to prevent unsafe operation.



Structured Text Code Snippet
Below is an example IEC 61131-3 Structured Text snippet for selected interlocks, demonstrating implementation in a PLC.

FUNCTION_BLOCK PRILLING_INTERLOCKS
(*
    Function Block: PRILLING_INTERLOCKS
    Purpose: Implements interlock logic for ammonium nitrate prilling station.
*)

VAR_INPUT    TT_101 : REAL; (* Tower temperature (°C) )    PT_101 : REAL; ( Melt pressure (bar) )    FT_101 : REAL; ( Cooling air flow (m³/h) )    P_101_Fault : BOOL; ( Melt pump fault )    LT_101 : REAL; ( Base level (%) *)    Execute : BOOL;    ManualReset : BOOL;    System_Tick_ms : UDINT;END_VAR
VAR_OUTPUT    FV_101_Close : BOOL; (* Close melt feed valve )    Shutdown : BOOL; ( Initiate tower shutdown )    AlarmActive : BOOL;    AlarmID : UINT; ( 1=Overtemp, 2=High Pressure, 3=Cooling Air Failure, ... *)    AuditLogEntry : STRING[80];END_VAR
VAR    InterlockLatched : BOOL;    LastAlarmID : UINT;    Timer : TON;END_VAR
(* Interlock 1: Overtemperature *)IF TT_101 > 200.0 THEN    FV_101_Close := TRUE;    Shutdown := TRUE;    InterlockLatched := TRUE;    AlarmActive := TRUE;    AlarmID := 1;    LastAlarmID := 1;    AuditLogEntry := 'I-001: Overtemperature, FV-101 closed, shutdown';END_IF;
(* Interlock 2: High Pressure *)IF PT_101 > 10.0 THEN    FV_101_Close := TRUE;    Shutdown := TRUE;    InterlockLatched := TRUE;    AlarmActive := TRUE;    AlarmID := 2;    LastAlarmID := 2;    AuditLogEntry := 'I-002: High pressure, PRV-101 opened, shutdown';END_IF;
(* Interlock 3: Cooling Air Failure *)IF FT_101 < 500.0 THEN    FV_101_Close := TRUE;    Shutdown := TRUE;    InterlockLatched := TRUE;    AlarmActive := TRUE;    AlarmID := 3;    LastAlarmID := 3;    AuditLogEntry := 'I-003: Cooling air failure, FV-101 closed, shutdown';END_IF;
(* Reset Logic *)IF InterlockLatched AND ManualReset AND TT_101 < 180.0 AND PT_101 < 8.0 AND FT_101 > 600.0 THEN    InterlockLatched := FALSE;    Shutdown := FALSE;    AlarmActive := FALSE;    AuditLogEntry := 'Manual reset: Interlocks cleared';END_IF;END_FUNCTION_BLOCK
