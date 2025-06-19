VAR
    TypeID  : BYTE;
    Triggered : BOOL := FALSE;
END_VAR

IF Execute AND NOT Triggered THEN
    Triggered := TRUE;

    IF DiagValid THEN
        TypeID := DiagData[7]; // Assume 8th byte defines diagnostic type (per DPV1 spec)

        FaultFlag := TRUE;
        ErrorTypeID := TypeID;

        CASE TypeID OF
            1: // Communication Error
                CommErrorCode := SHL(WORD(DiagData[8]), 8) OR WORD(DiagData[9]);
                LogText := 'Comm Error detected';
            
            2: // Device Status
                DeviceStatus := DiagData[8];
                LogText := 'Device status updated';

            3: // Parameter Fault
                ParamFaultCode := SHL(WORD(DiagData[8]), 8) OR WORD(DiagData[9]);
                LogText := 'Parameter fault occurred';

            4: // Configuration Mismatch
                ConfigMismatch := TRUE;
                LogText := 'Configuration mismatch detected';

            5: // Voltage Supply Issue
                VoltageLow := (DiagData[8] AND 16#01) <> 0;
                LogText := 'Voltage supply issue';

            6: // Temperature Warning
                TempWarning := (DiagData[8] AND 16#01) <> 0;
                LogText := 'Temperature warning detected';

            7: // Hardware Fault
                HardwareFault := TRUE;
                LogText := 'Hardware fault reported';

            8: // Bus Interruption
                BusInterrupted := TRUE;
                LogText := 'Bus interruption error';

            9: // Watchdog Timeout
                WatchdogTimeout := TRUE;
                LogText := 'Watchdog timeout occurred';

            10: // Manufacturer-specific diagnostic
                VendorCode := SHL(WORD(DiagData[8]), 8) OR WORD(DiagData[9]);
                LogText := 'Vendor-specific diagnostic';

            ELSE
                FaultFlag := TRUE;
                LogText := 'Unknown diagnostic type';
        END_CASE

    ELSE
        FaultFlag := FALSE;
        LogText := 'No valid diagnostic data';
    END_IF

ELSIF NOT Execute THEN
    Triggered := FALSE; // Reset trigger
END_IF
