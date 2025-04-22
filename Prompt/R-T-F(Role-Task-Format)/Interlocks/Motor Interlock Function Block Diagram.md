**Motor Interlock Function Block Diagram:**

Design a motor interlock as a function block diagram that prevents the motor from starting while other associated equipment is still running. The interlock should monitor the operational status of surrounding equipment and block the motor start command if any equipment is still active. Include inputs from sensors or status indicators and outputs that control the motor start circuit.

Provide the implementation of the MotorInterlock function block in IEC 61131-3 Structured Text. This function block should check the statuses of relevant equipment (e.g., EquipmentRunning), and if all equipment is stopped, it should allow the motor to start by setting the output to TRUE. If any equipment is still running, the output should remain FALSE, preventing the motor from starting.

Discuss the role of motor interlocks in industrial safety and how this logic prevents premature or unsafe motor operation.

**R-T-F:**

🟥 R (Role) – Your Role

You are an industrial automation engineer responsible for implementing safe motor start logic using IEC 61131-3 Structured Text. Your task is to ensure that a motor can only start when all associated equipment is safely shut down.

⸻

🟩 T (Task) – What You Need to Do

Create a function block in Structured Text named MotorInterlock that:
	•	Accepts Boolean inputs representing the running status of nearby equipment (e.g., Equipment1Running, Equipment2Running)
	•	Outputs a single Boolean value AllowStart that is TRUE only if all inputs are FALSE
	•	Prevents motor startup when any associated equipment is still active
	•	Can be reused in multiple control situations requiring coordinated startup logic

Additionally, briefly describe the role of motor interlocks in process safety and equipment coordination.

⸻

🟧 F (Format) – Expected Output

Your deliverable should include:
	•	A complete IEC 61131-3 Structured Text implementation of the MotorInterlock function block:

 FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL;
    Equipment2Running : BOOL;
    Equipment3Running : BOOL;
END_VAR
VAR_OUTPUT
    AllowStart : BOOL;
END_VAR

AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;

	•	A brief explanation highlighting how this interlock:
	•	Prevents unsafe motor activation
	•	Ensures correct sequencing of machine operations
	•	Contributes to system safety and regulatory compliance
