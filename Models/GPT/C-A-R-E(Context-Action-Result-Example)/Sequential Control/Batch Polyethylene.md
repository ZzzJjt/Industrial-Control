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


CASE Phase OF

0: // Idle phase – wait for start trigger
    IF StartProcess THEN
        Phase := 1;
    END_IF

1: // Raw Material Preparation
    HeatReactor(TempSetPoint);
    IF IsHeated THEN
        DoseMonomers();
        IF IsDosed THEN
            Phase := 2;
        END_IF
    END_IF

2: // Polymerization Phase
    StartAgitation(Speed := AgitationSpeed);
    ControlPolymerization();
    IF IsPolymerized THEN
        Phase := 3;
    END_IF

3: // Quenching Phase
    Timer(IN := TRUE, PT := QuenchDuration);
    IF NOT IsQuenched THEN
        StartQuenching();
        IF Timer.Q THEN
            StopQuenching();
            IsQuenched := TRUE;
            Timer(IN := FALSE);
            Phase := 4;
        END_IF
    END_IF

4: // Drying Phase
    Timer(IN := TRUE, PT := DryDuration);
    IF NOT IsDried THEN
        StartDrying();
        IF Timer.Q THEN
            StopDrying();
            IsDried := TRUE;
            Timer(IN := FALSE);
            Phase := 5;
        END_IF
    END_IF

5: // Pelletizing
    PelletizeMaterial();
    IF IsPelletized THEN
        Phase := 6;
    END_IF

6: // Quality Control
    PerformInspection();
    IF IsInspected THEN
        Phase := 7;
    END_IF

7: // Packaging
    PackageProduct();
    IF IsPackaged THEN
        Phase := 8;
    END_IF

8: // Batch Complete
    IF ResetProcess THEN
        ResetAll();
        Phase := 0;
    END_IF

END_CASE

METHOD HeatReactor
VAR_INPUT Temp : REAL;
StatusMessage := CONCAT('Heating reactor to ', REAL_TO_STRING(Temp), ' °C');
// Simulate heating complete
IF CurrentTemp >= Temp THEN
    IsHeated := TRUE;
END_IF
END_METHOD

METHOD DoseMonomers
// Simulate dosing monomers
StatusMessage := 'Dosing monomers into reactor';
IsDosed := TRUE;
END_METHOD

METHOD StartAgitation
VAR_INPUT Speed : INT;
StatusMessage := CONCAT('Starting agitator at ', INT_TO_STRING(Speed), ' RPM');
END_METHOD

METHOD ControlPolymerization
// Simulate process stabilization
StatusMessage := 'Polymerization running';
IF CurrentPressure <= 1.5 THEN
    IsPolymerized := TRUE;
END_IF
END_METHOD

METHOD StartQuenching
StatusMessage := 'Coolant flow ON';
END_METHOD

METHOD StopQuenching
StatusMessage := 'Coolant flow OFF';
END_METHOD

METHOD StartDrying
StatusMessage := 'Drying chamber active';
END_METHOD

METHOD StopDrying
StatusMessage := 'Drying complete';
END_METHOD

METHOD PelletizeMaterial
StatusMessage := 'Pelletizing melted polymer';
IsPelletized := TRUE;
END_METHOD

METHOD PerformInspection
StatusMessage := 'Inspecting product quality';
IsInspected := TRUE;
END_METHOD

METHOD PackageProduct
StatusMessage := 'Packing finished polyethylene pellets';
IsPackaged := TRUE;
END_METHOD

METHOD ResetAll
IsHeated := FALSE;
IsDosed := FALSE;
IsPolymerized := FALSE;
IsQuenched := FALSE;
IsDried := FALSE;
IsPelletized := FALSE;
IsInspected := FALSE;
IsPackaged := FALSE;
Timer(IN := FALSE);
StatusMessage := 'System reset and ready';
END_METHOD
