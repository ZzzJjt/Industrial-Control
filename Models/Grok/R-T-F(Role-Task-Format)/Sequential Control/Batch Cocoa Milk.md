(* IEC 61131-3 Structured Text program for ISA-88-compliant cocoa milk production *)
(* Controls ingredient addition, heating, and blending for 100 kg batch *)

PROGRAM CocoaMilkBatchControl
VAR
    (* Batch control variables *)
    BatchState : INT := 0; (* 0=Idle, 1=AddIngredients, 2=Heating, 3=Blending, 4=Complete *)
    IngredientPhase : INT := 0; (* Ingredient sub-states: 0=Idle, 1=Milk, 2=Water, 3=Sugar, 4=Cocoa *)
    StartBatch : BOOL := FALSE; (* Command to start batch *)
    BatchComplete : BOOL := FALSE; (* Flag for batch completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for process issues *)
    
    (* Process parameters *)
    TargetMilkWeight : REAL := 70.0; (* Milk weight in kg *)
    TargetWaterWeight : REAL := 20.0; (* Water weight in kg *)
    TargetSugarWeight : REAL := 8.0; (* Liquid sugar weight in kg *)
    TargetCocoaWeight : REAL := 2.0; (* Cocoa weight in kg *)
    WeightTolerance : REAL := 0.5; (* Acceptable weight deviation in kg *)
    
    TargetHeatingTemp : REAL := 70.0; (* Target heating temperature in Celsius *)
    TempTolerance : REAL := 2.0; (* Acceptable temperature deviation *)
    HeatingStirSpeed : REAL := 100.0; (* Stirring speed during heating in RPM *)
    BlendingStirSpeed : REAL := 200.0; (* Stirring speed during blending in RPM *)
    BlendingTime : TIME := T#10m; (* Blending duration *)
    
    (* Input variables *)
    CurrentWeight : REAL; (* Current vessel weight from load cells *)
    CurrentTemp : REAL; (* Current vessel temperature *)
    StirrerFeedback : BOOL; (* Stirrer operational status *)
    ValveFeedbackMilk : BOOL; (* Milk valve status *)
    ValveFeedbackWater : BOOL; (* Water valve status *)
    ValveFeedbackSugar : BOOL; (* Sugar valve status *)
    ValveFeedbackCocoa : BOOL; (* Cocoa valve status *)
    
    (* Output variables *)
    MilkValveCommand : BOOL; (* Command to milk inlet valve *)
    WaterValveCommand : BOOL; (* Command to water inlet valve *)
    SugarValveCommand : BOOL; (* Command to sugar inlet valve *)
    CocoaValveCommand : BOOL; (* Command to cocoa inlet valve *)
    HeaterCommand : BOOL; (* Command to heater *)
    StirrerCommand : BOOL; (* Command to stirrer *)
    
    (* Timers *)
    IngredientAddTimer : TON; (* Timer for ingredient addition *)
    HeatingTimer : TON; (* Timer for heating phase *)
    BlendingTimer : TON; (* Timer for blending phase *)
    
    (* Timer presets *)
    INGREDIENT_TIMEOUT : TIME := T#2m; (* Max time to add each ingredient *)
    HEATING_TIMEOUT : TIME := T#10m; (* Max time to reach target temperature *)
END_VAR

(* Modular function block declarations - assumed defined elsewhere *)
FUNCTION_BLOCK AddIngredient
    VAR_INPUT
        TargetWeight : REAL;
        CurrentWeight : REAL;
        ValveFeedback : BOOL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        ValveOn : BOOL;
        AdditionDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Logic to control ingredient valve and verify weight *)
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartHeating
    VAR_INPUT
        TargetTemp : REAL;
        CurrentTemp : REAL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        HeaterOn : BOOL;
        HeatingDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Logic to control heater and verify temperature *)
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartMixing
    VAR_INPUT
        TargetSpeed : REAL;
        Feedback : BOOL;
    END_VAR
    VAR_OUTPUT
        MixerOn : BOOL;
        MixingStable : BOOL;
        Fault : BOOL;
    END_VAR
    (* Logic to control stirrer and verify operation *)
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    MilkAddFB : AddIngredient;
    WaterAddFB : AddIngredient;
    SugarAddFB : AddIngredient;
    CocoaAddFB : AddIngredient;
    HeatingFB : StartHeating;
    MixingFB : StartMixing;
END_VAR

(* Main batch control logic *)
CASE BatchState OF
    0: (* Idle - Waiting for start command *)
        IF StartBatch THEN
            BatchState := 1; (* Start ingredient addition phase *)
            IngredientPhase := 0; (* Reset ingredient sub-state *)
            StartBatch := FALSE;
            (* Reset all timers *)
            IngredientAddTimer(IN := FALSE);
            HeatingTimer(IN := FALSE);
            BlendingTimer(IN := FALSE);
        END_IF;
        
    1: (* Ingredient Addition phase *)
        CASE IngredientPhase OF
            0: (* Ingredient Idle - Start milk addition *)
                IngredientPhase := 1;
                
            1: (* Add Milk *)
                MilkAddFB(TargetWeight := TargetMilkWeight, 
                         CurrentWeight := CurrentWeight, 
                         ValveFeedback := ValveFeedbackMilk, 
                         Tolerance := WeightTolerance);
                MilkValveCommand := MilkAddFB.ValveOn;
                IngredientAddTimer(IN := TRUE, PT := INGREDIENT_TIMEOUT);
                
                IF MilkAddFB.Fault OR IngredientAddTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF MilkAddFB.AdditionDone THEN
                    IngredientAddTimer(IN := FALSE);
                    IngredientPhase := 2; (* Transition to water *)
                END_IF;
                
            2: (* Add Water *)
                WaterAddFB(TargetWeight := TargetWaterWeight, 
                          CurrentWeight := CurrentWeight, 
                          ValveFeedback := ValveFeedbackWater, 
                          Tolerance := WeightTolerance);
                WaterValveCommand := WaterAddFB.ValveOn;
                IngredientAddTimer(IN := TRUE, PT := INGREDIENT_TIMEOUT);
                
                IF WaterAddFB.Fault OR IngredientAddTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF WaterAddFB.AdditionDone THEN
                    IngredientAddTimer(IN := FALSE);
                    IngredientPhase := 3; (* Transition to sugar *)
                END_IF;
                
            3: (* Add Liquid Sugar *)
                SugarAddFB(TargetWeight := TargetSugarWeight, 
                          CurrentWeight := CurrentWeight, 
                          ValveFeedback := ValveFeedbackSugar, 
                          Tolerance := WeightTolerance);
                SugarValveCommand := SugarAddFB.ValveOn;
                IngredientAddTimer(IN := TRUE, PT := INGREDIENT_TIMEOUT);
                
                IF SugarAddFB.Fault OR IngredientAddTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF SugarAddFB.AdditionDone THEN
                    IngredientAddTimer(IN := FALSE);
                    IngredientPhase := 4; (* Transition to cocoa *)
                END_IF;
                
            4: (* Add Cocoa *)
                CocoaAddFB(TargetWeight := TargetCocoaWeight, 
                          CurrentWeight := CurrentWeight, 
                          ValveFeedback := ValveFeedbackCocoa, 
                          Tolerance := WeightTolerance);
                CocoaValveCommand := CocoaAddFB.ValveOn;
                IngredientAddTimer(IN := TRUE, PT := INGREDIENT_TIMEOUT);
                
                IF CocoaAddFB.Fault OR IngredientAddTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF CocoaAddFB.AdditionDone THEN
                    IngredientAddTimer(IN := FALSE);
                    BatchState := 2; (* Transition to heating *)
                END_IF;
        END_CASE;
        
    2: (* Heating phase *)
        (* Start heating and low-speed mixing *)
        HeatingFB(TargetTemp := TargetHeatingTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        MixingFB(TargetSpeed := HeatingStirSpeed, 
                Feedback := StirrerFeedback);
        HeaterCommand := HeatingFB.HeaterOn;
        StirrerCommand := MixingFB.MixerOn;
        HeatingTimer(IN := TRUE, PT := HEATING_TIMEOUT);
        
        (* Interlock: Mixing must be stable before heating continues *)
        IF HeatingFB.Fault OR MixingFB.Fault OR HeatingTimer.Q THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF HeatingFB.HeatingDone AND MixingFB.MixingStable THEN
            HeatingTimer(IN := FALSE);
            BatchState := 3; (* Transition to blending *)
        END_IF;
        
    3: (* Blending phase *)
        (* Maintain temperature and increase mixing speed *)
        HeatingFB(TargetTemp := TargetHeatingTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        MixingFB(TargetSpeed := BlendingStirSpeed, 
                Feedback := StirrerFeedback);
        HeaterCommand := HeatingFB.HeaterOn;
        StirrerCommand := MixingFB.MixerOn;
        BlendingTimer(IN := TRUE, PT := BlendingTime);
        
        IF HeatingFB.Fault OR MixingFB.Fault THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF BlendingTimer.Q THEN
            BlendingTimer(IN := FALSE);
            BatchComplete := TRUE;
            BatchState := 4; (* Transition to complete *)
        END_IF;
        
    4: (* Complete - Batch finished *)
        HeaterCommand := FALSE;
        StirrerCommand := FALSE;
        MilkValveCommand := FALSE;
        WaterValveCommand := FALSE;
        SugarValveCommand := FALSE;
        CocoaValveCommand := FALSE;
        (* Await reset or transfer to next process *)
END_CASE;

(* Fault handling *)
IF FaultDetected THEN
    HeaterCommand := FALSE;
    StirrerCommand := FALSE;
    MilkValveCommand := FALSE;
    WaterValveCommand := FALSE;
    SugarValveCommand := FALSE;
    CocoaValveCommand := FALSE;
    (* Fault must be acknowledged by higher-level control *)
END_IF;

END_PROGRAM
