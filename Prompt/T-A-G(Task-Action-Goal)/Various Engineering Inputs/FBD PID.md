**FBD PID:**

Create a 61131-3 function block diagram in ASCII art. It shall contain an analog input, a timer block, and a PID block as well as an analog output. The analog input feeds both the timer and the PID block. Only the PID block is connected to the analog output. Provide typical input and output signals for each function block and show them inside each block.

**T-A-G:**

🟥 T (Task)

Design a 61131-3 Function Block Diagram (FBD) using ASCII art that visually represents a basic control structure involving an analog input, a timer, a PID controller, and an analog output.

⸻

🟩 A (Action)
	•	Create a clear ASCII layout with the following blocks:
	•	Analog Input block (OUT = AI)
	•	TON timer block (IN = AI, ET, Q)
	•	PID block (PV = AI, SP = 50.0, CV)
	•	Analog Output block (IN = CV)
	•	Ensure the Analog Input feeds both the TON and the PID block.
	•	Show that only the PID’s output (CV) connects to the Analog Output.
	•	Use ASCII symbols (+-----+, -->, |) to create block shapes and signal lines.
	•	Label all signals clearly inside or beside the blocks.

⸻

🟦 G (Goal)

Produce a plain-text diagram that visually communicates the data flow and block interaction within a control loop—useful for documentation, PLC training, version-controlled codebases, or text-only environments where graphical editors are unavailable.
