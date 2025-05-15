Here is a **IEC 61131-3-compliant Function Block Diagram (FBD)** representation using **ASCII art**, as requested. It includes:

- An **Analog Input** block  
- A **TON (On-Delay Timer)** block  
- A **PID** control block  
- An **Analog Output** block  

Signal flow and IEC 61131-3 standard signal names (e.g., `PV`, `SP`, `CV`, `IN`, `OUT`, `ET`, etc.) are clearly labeled for readability in plain-text environments such as code editors or documentation tools.

---

### 🧠 FBD Layout (Text-Based Representation)

```
+-------------+
| Analog Input |
|             |
| OUT ----> AI_Value ------+
+-------------+            |
                            v
         +--------------------------+      +------------+
         |          TON Timer       |      |   PID       |
         |                          |      | Block       |
         | IN <------------------+  |      | PV <------+ |
         | Q ------------------> +-+--+    | SP --> setpoint |
         | ET -----------------> | Time |  | CV --> analog_out |
         +----------------------+------|  +------------+
                                    |
                                    v
                           +------------------+
                           | Analog Output     |
                           | IN <---- CV        |
                           | OUT ---> ao_value  |
                           +------------------+
```

---

### 🔍 Signal Descriptions

| Block / Signal | Description |
|----------------|-------------|
| **Analog Input.OUT** (`AI_Value`) | Raw process variable from field instrument |
| **TON.IN** | Starts the timer when enabled |
| **TON.Q** | Timer output becomes TRUE after time elapses |
| **TON.ET** | Elapsed time value (in milliseconds or seconds) |
| **PID.PV** | Process Variable input from AI_Value |
| **PID.SP** | Setpoint defined by operator or higher-level logic |
| **PID.CV** | Controller output sent to analog output |
| **Analog Output.IN** (`CV`) | Control value from PID block |
| **Analog Output.OUT** (`ao_value`) | Output signal to final control element (e.g., valve, VFD) |

---

### ✅ Summary of Connections

- `AnalogInput.OUT` → `TON.IN`  
- `AnalogInput.OUT` → `PID.PV`  
- `PID.CV` → `AnalogOutput.IN`  

This layout ensures:
- The analog input feeds both the **timer** and **PID** block.
- Only the **PID output** drives the **analog output**.
- Signals follow IEC 61131-3 naming conventions.
- The diagram remains compact and legible in plain text.

---

Would you like this exported in a formatted Markdown file or extended with additional function blocks such as alarms, interlocks, or scaling functions?
