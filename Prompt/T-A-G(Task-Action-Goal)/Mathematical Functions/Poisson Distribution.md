**Poisson Distribution:**

Design a self-contained function block in IEC 61131-3 Structured Text to compute the Poisson distribution for a given mean (λ) and input value. Ensure that the function block is structured for clarity, with comments explaining the mathematical basis of the Poisson distribution and the computational steps involved. Address potential limitations, such as handling large values of λ, and discuss the practical applications of the Poisson distribution in industrial systems, particularly in the context of event-driven processes.

**T-A-G:**

🟥 T (Task) – What You Need to Do

Design a self-contained IEC 61131-3 Structured Text function block that calculates the Poisson probability for a given number of discrete events based on a specified average rate (λ).

⸻

🟩 A (Action) – How to Do It
	1.	Create a function block called PoissonProbability with the following structure:
	•	Inputs:
	•	Lambda : REAL — the expected mean number of events (λ)
	•	K : INT — the observed number of events
	•	Output:
	•	Probability : REAL — the result of the Poisson probability mass function
	2.	Implement the Poisson formula:
P(k; \lambda) = \frac{e^{-\lambda} \cdot \lambda^k}{k!}
	3.	Write Structured Text code that:
	•	Uses EXP(-Lambda) and POW(Lambda, K)
	•	Computes K! using a loop (since factorial is not built-in)
	•	Handles edge cases (e.g., return 0.0 if K < 0 or Lambda <= 0)
	•	Includes comments explaining:
	•	Each part of the formula
	•	Why and how factorial is computed
	•	Numerical stability and PLC execution limits
	4.	Example snippet:

 IF (Lambda > 0.0) AND (K >= 0) THEN
    Factorial := 1.0;
    FOR i := 1 TO K DO
        Factorial := Factorial * i;
    END_FOR;
    Probability := EXP(-Lambda) * POW(Lambda, K) / Factorial;
ELSE
    Probability := 0.0;
END_IF;

🟦 G (Goal) – What You Want to Achieve

Deliver a reliable, efficient, and reusable function block that:
	•	Computes Poisson probabilities for use in real-time control systems
	•	Enables PLCs to evaluate event likelihoods directly on the controller
	•	Can be applied in scenarios like:
	•	Predictive maintenance (e.g., faults per hour)
	•	Sensor event modeling (e.g., number of triggers per cycle)
	•	Queueing and flow rate prediction in production lines

This block helps integrate probabilistic reasoning into deterministic control logic while maintaining scan-time performance and accuracy.
