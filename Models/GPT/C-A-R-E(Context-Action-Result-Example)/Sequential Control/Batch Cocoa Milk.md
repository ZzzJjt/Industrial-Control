VAR_INPUT
    StartBatch       : BOOL;
    CurrentTemp      : REAL;
    BatchReady       : BOOL; // External system reset trigger
END_VAR

VAR CONSTANT
    Setpoint_Milk        : REAL := 60.0;   // kg
    Setpoint_Water       : REAL := 20.0;   // kg
    Setpoint_Sugar       : REAL := 15.0;   // kg
    Setpoint_Cocoa       : REAL := 5.0;    // kg
    Setpoint_MixSpeed    : INT := 200;     // RPM
    Setpoint_Temp        : REAL := 70.0;   // °C
    BlendTime            : TIME := T#10m;
END_VAR

CASE PhaseStep OF

0: // Idle / Await Start
    IF StartBatch THEN
        StatusMessage := 'Starting Cocoa Milk Batch';
        PhaseStep := 1;
    END_IF

1: // Add Milk
    IF NOT DosedMilk THEN
        AddIngredient('Milk', Setpoint_Milk); // Assume this is a callable FB
        DosedMilk := TRUE;
    END_IF
    StatusMessage := 'Milk Added';
    PhaseStep := 2;

2: // Add Water
    IF NOT DosedWater THEN
        AddIngredient('Water', Setpoint_Water);
        DosedWater := TRUE;
    END_IF
    StatusMessage := 'Water Added';
    PhaseStep := 3;

3: // Add Sugar
    IF NOT DosedSugar THEN
        AddIngredient('Sugar', Setpoint_Sugar);
        DosedSugar := TRUE;
    END_IF
    StatusMessage := 'Sugar Added';
    PhaseStep := 4;

4: // Add Cocoa
    IF NOT DosedCocoa THEN
        AddIngredient('Cocoa', Setpoint_Cocoa);
        DosedCocoa := TRUE;
    END_IF
    StatusMessage := 'Cocoa Added';
    PhaseStep := 5;

5: // Heating
    StatusMessage := 'Heating to Target Temperature';
    StartHeating(Setpoint_Temp); // external FB
    IF CurrentTemp >= Setpoint_Temp THEN
        HeatingComplete := TRUE;
        PhaseStep := 6;
    END_IF

6: // Mixing
    StatusMessage := 'Mixing at 200 RPM';
    StartMixing(Setpoint_MixSpeed);
    BlendTimer(IN := TRUE, PT := BlendTime);
    IF BlendTimer.Q THEN
        BlendTimer(IN := FALSE);
        PhaseStep := 7;
    END_IF

7: // Complete
    StatusMessage := 'Batch Complete';
    IF BatchReady THEN
        ResetAll(); // Reset booleans and timers
        PhaseStep := 0;
    END_IF

END_CASE

METHOD ResetAll
DosedMilk := FALSE;
DosedWater := FALSE;
DosedSugar := FALSE;
DosedCocoa := FALSE;
HeatingComplete := FALSE;
BlendTimer(IN := FALSE);
StatusMessage := 'System Reset';
END_METHOD
