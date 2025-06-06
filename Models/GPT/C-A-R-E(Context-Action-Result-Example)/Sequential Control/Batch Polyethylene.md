VAR
    Phase : INT := 0;
    Timer : TON;

    // Status flags
    IsHeated       : BOOL := FALSE;
    IsDosed        : BOOL := FALSE;
    IsPolymerized  : BOOL := FALSE;
    IsQuenched     : BOOL := FALSE;
    IsDried        : BOOL := FALSE;
    IsPelletized   : BOOL := FALSE;
    IsInspected    : BOOL := FALSE;
    IsPackaged     : BOOL := FALSE;

    // Parameters
    TempSetPoint     : REAL := 80.0;
    AgitationSpeed   : INT := 120;
    QuenchDuration   : TIME := T#5m;
    DryDuration      : TIME := T#10m;

    StartProcess     : BOOL := FALSE;
    ResetProcess     : BOOL := FALSE;
END_VAR
