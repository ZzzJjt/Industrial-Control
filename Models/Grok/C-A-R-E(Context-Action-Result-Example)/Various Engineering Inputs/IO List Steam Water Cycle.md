Signal Name                    | Signal Tag Number | A/D | Engineering Unit | Range          | Setpoint     | P&ID Reference
------------------------------|-------------------|-----|------------------|----------------|--------------|---------------
DrumLevelTransmitter          | FWLT1001          | A   | %                | 0–100          | 50.0         | P&ID-FW-01
DrumLevelTransmitterRedundant | FWLT1002          | A   | %                | 0–100          | 50.0         | P&ID-FW-01
FeedwaterFlowTransmitter      | FWFT1003          | A   | t/h              | 0–120          | 80.0         | P&ID-FW-02
FeedwaterControlValvePosition | FWCV1004          | A   | %                | 0–100          | Auto         | P&ID-FW-02
FeedwaterControlValveCommand  | FWCV1005          | A   | %                | 0–100          | Auto         | P&ID-FW-02
FeedwaterPressureTransmitter  | FWPT1006          | A   | bar              | 0–200          | 150.0        | P&ID-FW-02
FeedwaterTemperatureTransmitter | FWTT1007        | A   | °C               | 0–200          | 120.0        | P&ID-FW-02
BoilerDrumPressureTransmitter | FWPT1008          | A   | bar              | 0–250          | 180.0        | P&ID-FW-01
FeedwaterPump1SpeedFeedback   | FWPS1009          | A   | RPM              | 0–3600         | 3000         | P&ID-FW-03
FeedwaterPump1SpeedSetpoint   | FWPS1010          | A   | RPM              | 0–3600         | 3000         | P&ID-FW-03
FeedwaterPump1StartFeedback   | FWPS1011          | D   | —                | 0 or 1         | 1            | P&ID-FW-03
FeedwaterPump1RunCommand      | FWPS1012          | D   | —                | 0 or 1         | 1            | P&ID-FW-03
FeedwaterPump2SpeedFeedback   | FWPS1013          | A   | RPM              | 0–3600         | 3000         | P&ID-FW-03
FeedwaterPump2SpeedSetpoint   | FWPS1014          | A   | RPM              | 0–3600         | 3000         | P&ID-FW-03
FeedwaterPump2StartFeedback   | FWPS1015          | D   | —                | 0 or 1         | 1            | P&ID-FW-03
FeedwaterPump2RunCommand      | FWPS1016          | D   | —                | 0 or 1         | 1            | P&ID-FW-03
HighDrumLevelAlarm            | FWLA1017          | D   | —                | 0 or 1         | 0            | P&ID-FW-01
LowDrumLevelAlarm             | FWLA1018          | D   | —                | 0 or 1         | 0            | P&ID-FW-01
FeedwaterPumpFault            | FWPS1019          | D   | —                | 0 or 1         | 0            | P&ID-FW-03
EmergencyShutdownCommand      | FWES1020          | D   | —                | 0 or 1         | 0            | P&ID-FW-04
