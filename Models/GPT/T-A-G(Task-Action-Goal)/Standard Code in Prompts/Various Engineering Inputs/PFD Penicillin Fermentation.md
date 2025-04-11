```
+-----------------------+
|   Penicillin Fermentor |
|   (FERMENTOR_001)     |
|                       |
| +--------------------+ |
| | Air Filter (AF)    | |
| | pH Probe (PH_PROBE)| |
| | Temperature (TEMP) | |
| | Pressure (PRES)    | |
| | Agitator (AGITATOR)| |
| +--------------------+ |
|                       |
| Air In (AIRIN)        |
| Sterile Air Out (SAIROUT)|
| Penicillin Suspension Out (SUSPENSIONOUT)|
+-----------------------+

+---------------------+
| Seed Tank           |
| (SEEDTANK_001)      |
|                     |
| +------------------+ |
| | Level (LEVEL)     | |
| | Temperature (TEMP)| |
| +------------------+ |
|                     |
| Seed Medium In (SEEDIN)|
| Seed Medium Out (SEEDOUT)|
+---------------------+

+---------------------+
| Sterilizer          |
| (STERILIZER_001)   |
|                     |
| +------------------+ |
| | Temperature (TEMP)| |
| | Pressure (PRES)  | |
| +------------------+ |
|                     |
| Medium In (MEDIUMIN)|
| Medium Out (MEDIUMOUT)|
+---------------------+

+---------------------+
| Harvest Tank        |
| (HARVESTTANK_001)  |
|                     |
| +------------------+ |
| | Level (LEVEL)     | |
| | Temperature (TEMP)| |
| +------------------+ |
|                     |
| Suspension In (SUSPENSIONIN)|
| Suspension Out (SUSPENSIONOUT)|
+---------------------+

+---------------------+
| Storage Tank        |
| (STORAGE_001)      |
|                     |
| +------------------+ |
| | Level (LEVEL)     | |
| | Temperature (TEMP)| |
| +------------------+ |
|                     |
| Medium In (MEDIUMIN)|
| Medium Out (MEDIUMOUT)|
+---------------------+

+---------------------+
| Coolant System      |
| (COOLANTSYSTEM_001)|
|                     |
| +------------------+ |
| | Temperature (TEMP)| |
| +------------------+ |
|                     |
| Coolant In (COOLANTIN)|
| Coolant Out (COOLANTOUT)|
+---------------------+

Control Philosophy:

1. Temperature Control Loop:
   - Temperature Sensor (TEMP)
   - PID Controller (TEMP_PID)
   - Coolant Valve (COOLANTVALVE)
   - Setpoint (TEMP_SETPOINT)

   [TEMP] --> [TEMP_PID] --> [COOLANTVALVE]

2. Pressure Control Loop:
   - Pressure Sensor (PRES)
   - PID Controller (PRES_PID)
   - Pressure Relief Valve (PRV)
   - Setpoint (PRES_SETPOINT)

   [PRES] --> [PRES_PID] --> [PRV]

3. pH Control Loop:
   - pH Probe (PH_PROBE)
   - PID Controller (PH_PID)
   - pH Adjustment Valve (PH_ADJUSTVALVE)
   - Setpoint (PH_SETPOINT)

   [PH_PROBE] --> [PH_PID] --> [PH_ADJUSTVALVE]

4. Agitator Speed Control:
   - Agitator (AGITATOR)
   - Speed Control (AGIT_SPEED_CTRL)
   - Setpoint (AGIT_SPEED_SETPOINT)

   [AGIT_SPEED_CTRL] --> [AGITATOR]

5. Sterilization Control Loop:
   - Sterilizer Temperature (ST_TEMP)
   - PID Controller (ST_TEMP_PID)
   - Steam Valve (STEAMVALVE)
   - Setpoint (ST_TEMP_SETPOINT)

   [ST_TEMP] --> [ST_TEMP_PID] --> [STEAMVALVE]

6. Seed Medium Transfer Control:
   - Seed Tank Level (SEED_LEVEL)
   - PID Controller (SEED_LVL_PID)
   - Transfer Pump (TRANSFERPUMP)
   - Setpoint (SEED_LVL_SETPOINT)

   [SEED_LEVEL] --> [SEED_LVL_PID] --> [TRANSFERPUMP]

7. Harvest Tank Level Control:
   - Harvest Tank Level (HARVEST_LVL)
   - PID Controller (HARVEST_LVL_PID)
   - Harvest Pump (HARVESTPUMP)
   - Setpoint (HARVEST_LVL_SETPOINT)

   [HARVEST_LVL] --> [HARVEST_LVL_PID] --> [HARVESTPUMP]
```
