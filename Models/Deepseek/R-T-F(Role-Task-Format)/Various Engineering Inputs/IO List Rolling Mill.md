| Signal Name               | Input/Output | Signal Tagnumber | Analog/Digital | Engineering Unit | Range                | Setpoint              | P&ID Reference |
|---------------------------|--------------|------------------|----------------|------------------|----------------------|-----------------------|----------------|
| Motor_Speed_Feedback      | Input        | AI_101           | Analog         | RPM              | 0 - 5000             | N/A                   | PID_M1         |
| Roll_Gap_Position         | Input        | AI_102           | Analog         | mm               | 0 - 50               | N/A                   | PID_RG         |
| Strip_Tension             | Input        | AI_103           | Analog         | kN               | 0 - 100              | N/A                   | PID_ST         |
| Hydraulic_Pressure_Main   | Input        | AI_104           | Analog         | bar              | 0 - 300              | N/A                   | PID_HP         |
| Cooling_Valve_Opening     | Input        | AI_105           | Analog         | %                | 0 - 100              | N/A                   | PID_CV         |
| Oil_Level_MainTank        | Input        | AI_106           | Analog         | cm               | 0 - 150              | N/A                   | PID_OIL        |
| Motor_Current             | Input        | AI_107           | Analog         | A                | 0 - 1000             | N/A                   | PID_MC         |
| Bearing_Temperature       | Input        | AI_108           | Analog         | °C               | 0 - 100              | N/A                   | PID_BT         |
| Gearbox_Temperature       | Input        | AI_109           | Analog         | °C               | 0 - 100              | N/A                   | PID_GT         |
| Roll_Surface_Roughness    | Input        | AI_110           | Analog         | μm               | 0 - 50               | N/A                   | PID_RS         |
| Strip_Width               | Input        | AI_111           | Analog         | mm               | 0 - 500              | N/A                   | PID_SW         |
| Mill_Load                 | Input        | AI_112           | Analog         | kW               | 0 - 5000             | N/A                   | PID_ML         |
| Lubricant_Flow            | Input        | AI_113           | Analog         | L/min            | 0 - 100              | N/A                   | PID_LF         |
| Air_Compressor_Pressure   | Input        | AI_114           | Analog         | bar              | 0 - 15               | N/A                   | PID_ACP        |
| Motor_Voltage             | Input        | AI_115           | Analog         | V                | 0 - 690              | N/A                   | PID_MV         |
| Main_Hydraulic_Flow       | Input        | AI_116           | Analog         | L/min            | 0 - 500              | N/A                   | PID_HF         |
| Coolant_Temperature       | Input        | AI_117           | Analog         | °C               | 0 - 50               | N/A                   | PID_CT         |
| Drive_Brake_Status        | Input        | DI_101           | Digital        | Bool             | Off/On               | N/A                   | PID_DB         |
| Emergency_Stop_Button     | Input        | DI_102           | Digital        | Bool             | Off/On               | N/A                   | PID_ESB        |
| Power_Supply_Status       | Input        | DI_103           | Digital        | Bool             | Off/On               | N/A                   | PID_PSS        |
| Roll_Alignment_Sensor     | Input        | DI_104           | Digital        | Bool             | Misaligned/Aligned   | N/A                   | PID_RSA        |
| Motor_Start_Command       | Output       | DO_101           | Digital        | Bool             | Off/On               | N/A                   | PID_MSC        |
| Motor_Stop_Command        | Output       | DO_102           | Digital        | Bool             | Off/On               | N/A                   | PID_MST        |
| Cooling_Valve_Control     | Output       | AO_101           | Analog         | %                | 0 - 100              | 50                    | PID_CVC        |
| Hydraulic_Pump_Command    | Output       | DO_103           | Digital        | Bool             | Off/On               | N/A                   | PID_HPC        |
| Lubrication_System_Enable | Output       | DO_104           | Digital        | Bool             | Off/On               | N/A                   | PID_LSE        |
| Air_Compressor_Command    | Output       | DO_105           | Digital        | Bool             | Off/On               | N/A                   | PID_ACC        |
| Roll_Reversal_Command     | Output       | DO_106           | Digital        | Bool             | Off/On               | N/A                   | PID_RRC        |
| Maintenance_Mode_Active   | Output       | DO_107           | Digital        | Bool             | Off/On               | N/A                   | PID_MMA        |
