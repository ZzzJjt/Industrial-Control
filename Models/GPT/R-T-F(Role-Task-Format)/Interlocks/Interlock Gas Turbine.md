IF TurbineTemp > 650.0 THEN
    OverTemp_Trip := TRUE;
    TurbineShutdown := TRUE;
END_IF;

IF EStop_Button = TRUE THEN
    Emergency_Shutdown := TRUE;
    All_Outputs_Disable := TRUE;
END_IF;
