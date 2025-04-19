**Heating System Control Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text program (not a function block) to control the temperature of a heating system. The system should use input from three temperature sensors to automatically turn the heating on and off, maintaining a constant temperature range between 20°C and 22°C. The program must ensure smooth temperature regulation and prioritize energy efficiency by minimizing frequent switching. Safety measures should be implemented to handle sensor faults or temperature deviations beyond the specified range.

**R-I-S-E:**

🟥 R (Role) – Your Role

You are a PLC programmer tasked with designing a Structured Text (ST) program that controls a heating system. Your objective is to maintain the temperature within a specified comfort range while prioritizing energy efficiency and system safety.

⸻

🟩 I (Input) – What You’re Given
	•	Temperature Sensors:
	•	TempSensor1, TempSensor2, TempSensor3 — Real values (°C) from three sensors
	•	Setpoint Range:
	•	Target operating range: 20°C to 22°C
	•	Safety Range:
	•	Valid sensor readings: 10°C to 30°C
	•	Output Variable:
	•	HeatingOn : BOOL — Controls the heating system
	•	Fault Flag:
	•	SensorFault : BOOL — Indicates a sensor fault or unsafe condition

⸻

🟧 S (Steps) – What You Need to Do
	1.	Calculate Average Temperature
	•	Compute the average of the three sensors for smoother control.
	2.	Check for Sensor Faults
	•	If any sensor reads outside the 10–30°C range, set SensorFault := TRUE and disable heating.
	3.	Control Heating with Hysteresis
	•	If no fault is detected:
	•	Turn HeatingOn := TRUE if the average temperature is below 20°C
	•	Turn HeatingOn := FALSE if the average temperature is above 22°C
	•	Maintain current heating state if the temperature is within bounds.

⸻

🟦 E (Expectation) – What Success Looks Like

You will deliver a robust, real-time Structured Text program that:
	•	Maintains temperature between 20°C and 22°C with minimal switching
	•	Improves reliability by using average temperature from 3 sensors
	•	Disables heating and raises a fault flag if any sensor reports an unsafe value
	•	Can be extended with timers, manual overrides, or data logging if needed

⸻

✅ Sample Code Snippet:
VAR
    TempSensor1, TempSensor2, TempSensor3 : REAL;
    AvgTemp : REAL;
    HeatingOn : BOOL := FALSE;
    SensorFault : BOOL := FALSE;
END_VAR

AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Fault detection
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE;
ELSE
    SensorFault := FALSE;

    // Hysteresis-based temperature control
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;
END_IF;
