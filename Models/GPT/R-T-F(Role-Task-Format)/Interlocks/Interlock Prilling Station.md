VAR
    TT_101 : REAL;
    OverTempSetpoint : REAL := 120.0;
    ESD_OverTemp : BOOL := FALSE;
    MELT_FEED_STOP : BOOL := FALSE;
END_VAR

(* Overtemperature Interlock *)
IF TT_101 > OverTempSetpoint THEN
    ESD_OverTemp := TRUE;
END_IF;

IF ESD_OverTemp THEN
    MELT_FEED_STOP := TRUE;
END_IF;
