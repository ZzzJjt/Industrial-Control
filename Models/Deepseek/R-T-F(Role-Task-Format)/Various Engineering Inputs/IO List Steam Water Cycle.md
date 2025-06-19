| Signal Name                     | Signal Tag Number | Analog/Digital | Engineering Unit | Range              | Setpoint           | P&ID Reference |
|---------------------------------|-------------------|----------------|------------------|--------------------|--------------------|----------------|
| Feedwater_Flow_Transmitter      | AI_101            | Analog         | t/h              | 0 - 200            | 150                | PID_FWFT         |
| Drum_Level_Sensor               | AI_102            | Analog         | %                | 0 - 100            | 50                 | PID_DLS          |
| Boiler_Feed_Pump_Speed          | AI_103            | Analog         | RPM              | 0 - 3000           | N/A                | PID_BFPS         |
| Deaerator_Level_Sensor          | AI_104            | Analog         | %                | 0 - 100            | 60                 | PID_DEAL         |
| Condenser_Outlet_Temperature    | AI_105            | Analog         | °C               | 0 - 100            | N/A                | PID_COT          |
| Makeup_Water_Valve_Command      | AO_101            | Analog         | %                | 0 - 100            | 40                 | PID_MWVC         |
| Feedwater_Control_Valve_Command | AO_102            | Analog         | %                | 0 - 100            | 50                 | PID_FWCVC        |
| Boiler_Feed_Pump_Start_Command  | DO_101            | Digital        | Bool             | Off/On             | N/A                | PID_BFPSC        |
| Boiler_Feed_Pump_Stop_Command   | DO_102            | Digital        | Bool             | Off/On             | N/A                | PID_BFPSTC       |
| Emergency_Shutdown_Button       | DI_101            | Digital        | Bool             | Off/On             | N/A                | PID_ESB          |
| High_Level_Alarm                | DI_102            | Digital        | Bool             | Off/On             | N/A                | PID_HLA          |
| Low_Level_Alarm                 | DI_103            | Digital        | Bool             | Off/On             | N/A                | PID_LLA          |
| High_Temperature_Alarm          | DI_104            | Digital        | Bool             | Off/On             | N/A                | PID_HTSA         |
| Low_Temperature_Alarm           | DI_105            | Digital        | Bool             | Off/On             | N/A                | PID_LTSA         |
| Pump_Seal_Integrity_Check       | DI_106            | Digital        | Bool             | Intact/Faulty      | N/A                | PID_PSIC         |
| Pump_Discharge_Pressure         | AI_106            | Analog         | bar              | 0 - 200            | N/A                | PID_PDPR         |
| Pump_Inlet_Pressure             | AI_107            | Analog         | bar              | 0 - 100            | N/A                | PID_PIPI         |
| Pump_Motor_Current              | AI_108            | Analog         | A                | 0 - 1000           | N/A                | PID_PMCI         |
| Pump_Motor_Voltage              | AI_109            | Analog         | V                | 0 - 690            | N/A                | PID_PMVI         |
| Pump_Bearing_Temperature        | AI_110            | Analog         | °C               | 0 - 100            | N/A                | PID_PBTE         |
| Pump_Gearbox_Temperature        | AI_111            | Analog         | °C               | 0 - 100            | N/A                | PID_PGTE         |
| Pump_Oil_Level                  | AI_112            | Analog         | cm               | 0 - 150            | N/A                | PID_POIL         |
| Pump_Seal_Water_Flow            | AI_113            | Analog         | L/min            | 0 - 100            | N/A                | PID_PSWF         |
| Feedwater_Regulator_Status      | DI_107            | Digital        | Bool             | Active/Inactive    | N/A                | PID_FRSTS        |
| Feedwater_Pump_Interlock        | DI_108            | Digital        | Bool             | Off/On             | N/A                | PID_FPINT        |
| Feedwater_Valve_Interlock       | DI_109            | Digital        | Bool             | Off/On             | N/A                | PID_FVINT        |
| Feedwater_System_Mode_Selector  | DI_110            | Digital        | Manual/Auto      | N/A                | Auto               | PID_FSMS         |
