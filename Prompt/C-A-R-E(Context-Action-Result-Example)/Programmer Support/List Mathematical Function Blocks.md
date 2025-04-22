**List Mathematical Function Blocks:**
Compile a detailed list of all mathematical function blocks available in the OSCAT library for PLC programming, including a brief description of their functionality and typical use cases.

**C-A-R-E:**

🟥 C (Context) – Background

You are developing control logic using a PLC programming environment and leveraging the OSCAT library, a widely used open-source library for automation. The OSCAT library offers a large number of mathematical function blocks, but they are spread across modules and lack a centralized, user-friendly summary.

⸻

🟩 A (Action) – What You Need to Do

Compile a detailed and organized list of all mathematical function blocks provided in the OSCAT library. For each function block, include:
	•	The name of the block
	•	A brief description of its function
	•	Typical use cases in industrial automation (e.g., PID control, signal processing, data normalization)

⸻

🟨 R (Result) – The Value You Provide

The result will be a consolidated reference document that:
	•	Helps developers quickly locate the right function block for a task
	•	Saves time during development and troubleshooting
	•	Increases code clarity and maintainability
	•	Supports onboarding of new engineers by offering a readable overview of mathematical capabilities

⸻

🟦 E (Example) – Suggested Format

## Trigonometric Function Blocks
- **SIN(x)** – Calculates the sine of angle x (in radians).  
  *Use case*: Applied in motion control or waveform generation.

- **COS(x)** – Returns the cosine of x.  
  *Use case*: Used in rotating system calculations and phase shifting.

## Algebraic Functions
- **SQRT(x)** – Returns the square root of x.  
  *Use case*: Useful in geometric calculations and physics-based models.

## Statistical Functions
- **MEAN_ARRAY(array)** – Calculates the average of elements in an array.  
  *Use case*: Real-time data smoothing in sensor fusion.

- **STDDEV_ARRAY(array)** – Returns standard deviation of an array.  
  *Use case*: Used in quality control systems or anomaly detection.
