VAR
    AutoMode: BOOL;
    ManualMode: BOOL;
    StartCycle: BOOL;
    ArmReady: BOOL;
    ConveyorReady: BOOL;
    ArmCommand: BOOL;
    ConveyorCommand: BOOL;
END_VAR

IF AutoMode AND StartCycle THEN
    IF ArmReady THEN
        ArmCommand := TRUE;
    END_IF;
    IF ConveyorReady THEN
        ConveyorCommand := TRUE;
    END_IF;
ELSIF ManualMode THEN
    ArmCommand := FALSE;
    ConveyorCommand := FALSE;
END_IF;

      LD    AutoMode           // If AutoMode AND StartCycle
      AND   StartCycle
      JMPC  auto_section       // Jump to auto_section if condition is TRUE
      LD    ManualMode         // Else if ManualMode
      JMPC  manual_section     // Jump to manual_section

      JMP   end                // If neither, skip to end

auto_section:
      LD    ArmReady
      JMPC  arm_ok
      JMP   conveyor_check     // Skip ArmCommand if not ready

arm_ok:
      LD    TRUE
      ST    ArmCommand

conveyor_check:
      LD    ConveyorReady
      JMPC  conveyor_ok
      JMP   end

conveyor_ok:
      LD    TRUE
      ST    ConveyorCommand
      JMP   end

manual_section:
      LD    FALSE
      ST    ArmCommand
      ST    ConveyorCommand

end:
      NOP
