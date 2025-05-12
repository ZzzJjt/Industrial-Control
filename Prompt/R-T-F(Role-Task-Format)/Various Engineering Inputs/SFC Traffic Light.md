**SFC Traffic Light:**

Create a 61131-3 sequential function chart in ASCII art for a traffic light. Include proper delays for switching the lights.

**R-T-F:**

🟥 R (Role) – Define Your Role

Act as a control systems engineer responsible for designing and documenting a Sequential Function Chart (SFC) for a traffic light system in compliance with IEC 61131-3.

⸻

🟩 T (Task) – Define the Objective

Create an ASCII-based SFC diagram for a traffic light that includes:
	•	Three sequential steps: Green, Yellow, and Red
	•	A TON (on-delay) timer at each step to define its duration:
	•	Green for 5 seconds
	•	Yellow for 2 seconds
	•	Red for 5 seconds
	•	Transitions between steps based on each timer’s .Q output
	•	A clear loop that cycles continuously from Red back to Green

⸻

🟧 F (Format) – Specify the Output Format

Present the SFC in plain ASCII art, using:
	•	Boxes or labels to represent steps
	•	Arrows (-->) or downward flow to indicate transitions
	•	Timer function blocks and conditions next to each step
	•	Optional: clearly label the SFC loop to show full cycle logic

Example:

[Step: Green Light]
   TON tGreen (PT := T#5s)
        |
        V (tGreen.Q)
[Step: Yellow Light]
   TON tYellow (PT := T#2s)
        |
        V (tYellow.Q)
[Step: Red Light]
   TON tRed (PT := T#5s)
        |
        V (tRed.Q)
--> Loop to Green
