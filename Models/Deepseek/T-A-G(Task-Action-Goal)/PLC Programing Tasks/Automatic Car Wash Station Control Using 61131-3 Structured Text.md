PROGRAM CarWashControl
VAR_INPUT
    CarPresentSensor : BOOL; // Detects a car in the wash bay
    HumanDetectedSensor : BOOL; // Detects a person in the wash zone
END_VAR

VAR_OUTPUT
    WashActive : BOOL; // Starts/stops the wash process
    AlarmActive : BOOL; // Activates when a person is detected during wash
END_VAR

VAR
    SafeToRun : BOOL := TRUE; // Internal flag that governs when a wash can safely begin
END_VAR

// Main execution loop
IF HumanDetectedSensor THEN
    // Human detected, stop the wash, activate alarm, and set system to unsafe
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE;
ELSIF CarPresentSensor AND NOT HumanDetectedSensor AND SafeToRun THEN
    // Car detected, no human present, and system is safe, start the wash
    WashActive := TRUE;
    AlarmActive := FALSE;
ELSE
    // No car or human detected, or system is not safe to run
    WashActive := FALSE;
END_IF;

// Reset the system if the area is clear and wash is stopped
IF NOT HumanDetectedSensor AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF;

// Additional comments for clarity
// - If HumanDetectedSensor is TRUE, the system immediately stops the wash process,
//   activates the alarm, and sets SafeToRun to FALSE to prevent further washing.
// - If CarPresentSensor is TRUE and HumanDetectedSensor is FALSE and SafeToRun is TRUE,
//   the system starts the wash process.
// - When neither a car nor a human is detected and the wash is not active, SafeToRun is set to TRUE,
//   allowing the system to be ready for a new wash cycle.
// - This logic ensures that the wash process only starts when it is safe to do so and stops immediately
//   if a human is detected, preventing any potential hazards.



