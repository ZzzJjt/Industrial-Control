```
+------------------------+
| Boiler (BOILER_001)    |
|                        |
| +----------------------+ |
| | Water Level (WLEV)   | |
| | Pressure (BPRES)     | |
| +----------------------+ |
|                        |
| Feedwater In (FWIN)    |
| Steam Out (STEAMOUT)   |
+------------------------+

+---------------------+
| Feedwater Heater    |
| (FEEDHEAT_001)      |
|                    |
| +------------------+ |
| | Level (FLVL)      | |
| | Temperature (FTMP)| |
| +------------------+ |
|                    |
| Feedwater In (FWIN) |
| Steam In (STEAMIN)  |
| Feedwater Out (FWOUT)|
+---------------------+

+------------------+
| Economizer       |
| (ECONOMIZER_001) |
|                 |
| +---------------+ |
| | Inlet Temp (ITMP)| |
| | Outlet Temp (OTMP)| |
| +---------------+ |
|                 |
| Feedwater In (FWIN)|
| Feedwater Out (FWOUT)|
+------------------+

+------------------+
| Deaerator        |
| (DEAERATOR_001)  |
|                 |
| +---------------+ |
| | Level (DLVL)   | |
| | Pressure (DPRES)| |
| +---------------+ |
|                 |
| Feedwater In (FWIN)|
| Condensate In (CONDIN)|
| Feedwater Out (FWOUT)|
+------------------+

+------------------+
| Condenser        |
| (CONDENSER_001)  |
|                 |
| +---------------+ |
| | Level (CLVL)   | |
| | Pressure (CPRES)| |
| +---------------+ |
|                 |
| Steam In (STEAMIN)|
| Condensate Out (CONDOUT)|
+------------------+

+------------------+
| Feedwater Pump   |
| (FWPUMP_001)     |
|                 |
| +---------------+ |
| | Power (PW)     | |
| +---------------+ |
|                 |
| Feedwater In (FWIN)|
| Feedwater Out (FWOUT)|
+------------------+

Control Loops:

1. Feedwater Flow Control Loop:
   - Feedwater Flow Meter (FWFM_001)
   - Feedwater Control Valve (FWCV_001)
   - Setpoint (FWF_SETPOINT)
   - PID Controller (FW_PID_001)

   [FWFM_001] --> [FW_PID_001] --> [FWCV_001]

2. Boiler Water Level Control Loop:
   - Boiler Water Level Transmitter (WLEV_TRNS_001)
   - Feedwater Control Valve (FWCV_001)
   - Setpoint (WLEV_SETPOINT)
   - PID Controller (WLEV_PID_001)

   [WLEV_TRNS_001] --> [WLEV_PID_001] --> [FWCV_001]

3. Boiler Pressure Control Loop:
   - Boiler Pressure Transmitter (BPRES_TRNS_001)
   - Boiler Pressure Control Valve (BPCV_001)
   - Setpoint (BPRES_SETPOINT)
   - PID Controller (BPRES_PID_001)

   [BPRES_TRNS_001] --> [BPRES_PID_001] --> [BPCV_001]

4. Feedwater Heater Level Control Loop:
   - Feedwater Heater Level Transmitter (FLVL_TRNS_001)
   - Feedwater Heater Control Valve (FLCV_001)
   - Setpoint (FLVL_SETPOINT)
   - PID Controller (FLVL_PID_001)

   [FLVL_TRNS_001] --> [FLVL_PID_001] --> [FLCV_001]

5. Feedwater Heater Temperature Control Loop:
   - Feedwater Heater Temperature Transmitter (FTMP_TRNS_001)
   - Steam Control Valve (SCV_001)
   - Setpoint (FTMP_SETPOINT)
   - PID Controller (FTMP_PID_001)

   [FTMP_TRNS_001] --> [FTMP_PID_001] --> [SCV_001]

6. Deaerator Level Control Loop:
   - Deaerator Level Transmitter (DLVL_TRNS_001)
   - Deaerator Control Valve (DCV_001)
   - Setpoint (DLVL_SETPOINT)
   - PID Controller (DLVL_PID_001)

   [DLVL_TRNS_001] --> [DLVL_PID_001] --> [DCV_001]

7. Condenser Level Control Loop:
   - Condenser Level Transmitter (CLVL_TRNS_001)
   - Condenser Control Valve (CCV_001)
   - Setpoint (CLVL_SETPOINT)
   - PID Controller (CLVL_PID_001)

   [CLVL_TRNS_001] --> [CLVL_PID_001] --> [CCV_001]

8. Economizer Temperature Control Loop:
   - Economizer Inlet Temperature Transmitter (ITMP_TRNS_001)
   - Economizer Outlet Temperature Transmitter (OTMP_TRNS_001)
   - Economizer Control Valve (ECV_001)
   - Setpoint (ITMP_SETPOINT)
   - PID Controller (ITMP_PID_001)

   [ITMP_TRNS_001] --> [ITMP_PID_001] --> [ECV_001]
```
