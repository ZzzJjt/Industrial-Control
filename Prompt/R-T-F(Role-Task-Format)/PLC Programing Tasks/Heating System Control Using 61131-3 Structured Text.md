**Heating System Control Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text program (not a function block) to control the temperature of a heating system. The system should use input from three temperature sensors to automatically turn the heating on and off, maintaining a constant temperature range between 20°C and 22°C. The program must ensure smooth temperature regulation and prioritize energy efficiency by minimizing frequent switching. Safety measures should be implemented to handle sensor faults or temperature deviations beyond the specified range.

**R-T-F:**

🟥 R (Role) – Your Role

You are a PLC software developer responsible for programming a heating system control application using IEC 61131-3 Structured Text (ST). Your job is to create a robust and efficient temperature regulation program that ensures comfort, safety, and energy efficiency.

⸻

🟩 T (Task) – What You Need to Do

Develop a self-contained Structured Text program (not a function block) that:
	1.	Monitors three temperature sensors (TempSensor1, TempSensor2, TempSensor3)
	2.	Calculates the average temperature to make decisions less sensitive to individual sensor noise
	3.	Turns on the heating when average temperature drops below 20°C
	4.	Turns off the heating when average temperature exceeds 22°C
	5.	Uses hysteresis to avoid frequent switching between on and off
	6.	Implements safety logic:
	•	If any sensor reading is below 10°C or above 30°C, mark it as a sensor fault and disable the heater

⸻

🟧 F (Format) – Expected Output

The final output should be a clean, readable Structured Text program like the following:
VAR
    TempSensor1, TempSensor2, TempSensor3 : REAL;
    AvgTemp : REAL;
    HeatingOn : BOOL := FALSE;
    SensorFault : BOOL := FALSE;
END_VAR

// Compute average temperature
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Check for any sensor out of safe bounds
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE; // Force off if fault detected
ELSE
    SensorFault := FALSE;

    // Hysteresis control for energy-efficient switching
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;
END_IF;
This ensures:
	•	Smooth temperature regulation
	•	Minimized wear on heating components
	•	Safe shutdown in case of sensor failure or out-of-bound values
