**FBD PID:**

Create a 61131-3 function block diagram in ASCII art. It shall contain an analog input, a timer block, and a PID block as well as an analog output. The analog input feeds both the timer and the PID block. Only the PID block is connected to the analog output. Provide typical input and output signals for each function block and show them inside each block.


**R-T-F:**

🟥 R (Role) – Define Your Role

Act as a PLC systems designer tasked with documenting control logic using IEC 61131-3 Function Block Diagram (FBD) principles in ASCII format for use in design documentation and peer review.

⸻

🟩 T (Task) – Define the Objective

Create a readable ASCII function block diagram that includes the following components:
	•	An Analog Input block
	•	A TON (On-delay Timer) block
	•	A PID control block
	•	An Analog Output block

Ensure the diagram follows this flow:
	•	The Analog Input feeds both the Timer and PID block
	•	Only the PID output (Control Variable, CV) connects to the Analog Output
	•	Display typical signal names (e.g., IN, OUT, PV, SP, CV, ET) inside each block
	•	Arrange and label the blocks and connections clearly using ASCII characters (e.g., +---+, -->)

⸻

🟧 F (Format) – Specify the Output Format

Deliver:
	•	A plain-text ASCII diagram with:
	•	Each block represented as a boxed unit (+------+)
	•	Clearly labeled signal paths between blocks
	•	Inputs and outputs shown with common control variable names
	•	A layout that shows how the signal flows from input to output, including branches to multiple blocks
	•	All blocks and signal paths aligned for easy readability in code editors or printed docs
