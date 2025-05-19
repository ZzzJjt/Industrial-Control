I'll generate detailed developer documentation for the `UreaReactionControl` IEC 61131-3 Structured Text (ST) program, assuming a typical implementation for a urea synthesis process based on the provided context, action, result, and example. The documentation will cover the program’s purpose, variable descriptions, step-by-step logic flow, valve control conditions, reaction timing, and the use of `CURRENT_TIME` for duration tracking. Since the original code isn’t provided, I’ll infer a reasonable structure consistent with the described functionality, ensuring clarity for onboarding, maintenance, and compliance.

---

**Developer Documentation: UreaReactionControl**

### Purpose
The `UreaReactionControl` program automates the two-step urea synthesis process in a chemical reactor, controlling the intake of raw materials (ammonia and CO₂), monitoring reaction conditions (temperature and pressure), and ensuring the process completes safely and efficiently. It:
- Manages valve actuation to load ammonia and CO₂.
- Verifies that temperature and pressure remain within specified tolerances during the reaction.
- Tracks reaction duration to ensure completion after a defined period.
- Safely shuts down the process and signals completion.
The program is designed for cyclic execution in a PLC environment, ensuring real-time control, safety, and compliance with industrial chemical process standards.

### Variable Overview
The program uses inputs, outputs, internal flags, and parameter constants to manage the urea synthesis process. Below is a detailed breakdown of all variables.

#### Inputs
| Name                   | Type  | Description                                      |
|------------------------|-------|--------------------------------------------------|
| `stAmmoniaValve`       | BOOL  | Status of the ammonia valve (TRUE = open, FALSE = closed, read-only from field sensor). |
| `stCO2Valve`           | BOOL  | Status of the CO₂ valve (TRUE = open, FALSE = closed, read-only from field sensor). |
| `rCurrentPressure`     | REAL  | Current reactor pressure, measured in bar.       |
| `rCurrentTemperature`  | REAL  | Current reactor temperature, measured in °C.     |

#### Outputs
| Name                      | Type  | Description                                      |
|---------------------------|-------|--------------------------------------------------|
| `stAmmoniaValveControl`   | BOOL  | Control signal for ammonia valve (TRUE = open, FALSE = close). |
| `stCO2ValveControl`       | BOOL  | Control signal for CO₂ valve (TRUE = open, FALSE = close). |
| `stReactionFinished`      | BOOL  | TRUE when the reaction process is complete and shutdown is executed. |

#### Internal Flags and Variables
| Name                      | Type  | Description                                      |
|---------------------------|-------|--------------------------------------------------|
| `Step`                    | INT   | Current process step (0 = Idle, 1 = Material Loading, 2 = Reaction Control, 3 = Shutdown). |
| `ReactionStartTime`       | TIME  | Timestamp when reaction conditions are met, used to track duration. |
| `ConditionsMet`           | BOOL  | TRUE when temperature and pressure are within tolerance. |
| `LastConditionsMet`       | BOOL  | Previous state of `ConditionsMet` for edge detection. |
| `ReactionTimerActive`     | BOOL  | TRUE when reaction duration timer is active. |
| `Last_AmmoniaValve`       | BOOL  | Previous state of `stAmmoniaValve` for change detection. |
| `Last_CO2Valve`           | BOOL  | Previous state of `stCO2Valve` for change detection. |
| `Last_ReactionFinished`   | BOOL  | Previous state of `stReactionFinished` for change detection. |
| `DiagLog`                 | ARRAY[1..50] OF STRING[80] | Diagnostic logs for state changes and errors. |
| `LogCount`                | INT   | Number of log entries.                           |
| `LogBufferFull`           | BOOL  | TRUE if log buffer is full (50 entries).         |

#### Parameter Constants
| Parameter                 | Value      | Description                                      |
|---------------------------|------------|--------------------------------------------------|
| `rTargetPressure`         | 175.0      | Target reactor pressure, in bar.                 |
| `rPressureTolerance`      | 5.0        | Allowed ± deviation from target pressure, in bar. |
| `rTargetTemperature`      | 185.0      | Target reactor temperature, in °C.               |
| `rTemperatureTolerance`   | 2.0        | Allowed ± deviation from target temperature, in °C. |
| `tReactionTime`           | T#30m      | Required reaction duration (30 minutes).         |
| `rMinPressure`            | 0.0        | Minimum valid pressure, in bar (for input validation). |
| `rMaxPressure`            | 300.0      | Maximum valid pressure, in bar (for input validation). |
| `rMinTemperature`         | 0.0        | Minimum valid temperature, in °C (for input validation). |
| `rMaxTemperature`         | 300.0      | Maximum valid temperature, in °C (for input validation). |

### Program Logic
The `UreaReactionControl` program operates as a state machine with three main steps: Material Loading, Reaction Control, and Shutdown. It uses a cyclic execution model, running every PLC scan (assumed 100 ms cycle time), and leverages `CURRENT_TIME` to track reaction duration. Below is a step-by-step explanation of the logic flow, including valve control conditions and timing.

#### Step 0: Idle
- **Description**: The initial state, where the reactor is inactive, awaiting a start command (not explicitly modeled but assumed via external trigger to enter Step 1).
- **Actions**:
  - All outputs are off: `stAmmoniaValveControl := FALSE`, `stCO2ValveControl := FALSE`, `stReactionFinished := FALSE`.
  - Timers and internal flags are reset: `ReactionStartTime := T#0s`, `ReactionTimerActive := FALSE`, `ConditionsMet := FALSE`.
  - Logs initialization if transitioning from another state: e.g., "2025-05-19 12:45:00 Process Initialized: Idle State".
- **Valve Control**: Both valves closed (`stAmmoniaValveControl := FALSE`, `stCO2ValveControl := FALSE`).
- **Timing**: No timing active in this state.
- **Transition**: Moves to Step 1 (Material Loading) on an external start command (assumed manual or supervisory control system trigger).

#### Step 1: Material Loading
- **Description**: Opens valves to load ammonia and CO₂ into the reactor, ensuring both are fully open before proceeding.
- **Actions**:
  - Sets `stAmmoniaValveControl := TRUE` and `stCO2ValveControl := TRUE` to open both valves.
  - Monitors `stAmmoniaValve` and `stCO2Valve` (input feedback) to confirm both are open (TRUE).
  - If both valves are confirmed open, transitions to Step 2 (Reaction Control).
  - Logs valve state changes and step transition: e.g., "2025-05-19 12:45:00 Ammonia Valve Opened", "2025-05-19 12:45:00 Step 1: Material Loading Complete, Starting Reaction Control".
- **Valve Control**:
  - `stAmmoniaValveControl := TRUE` until `stAmmoniaValve = TRUE`.
  - `stCO2ValveControl := TRUE` until `stCO2Valve = TRUE`.
  - If either valve fails to open (e.g., `stAmmoniaValve` remains FALSE after a timeout, not implemented here but recommended), log an error and halt (future enhancement).
- **Timing**: No explicit timer in this step; relies on valve feedback. A timeout (e.g., 30s) could be added to detect valve failures.
- **Transition**: 
  - To Step 2 (Reaction Control) when `stAmmoniaValve = TRUE` AND `stCO2Valve = TRUE`.
  - Stays in Step 1 if valves are not fully open.

#### Step 2: Reaction Control
- **Description**: Monitors reactor temperature and pressure, ensuring they remain within tolerance ranges, and tracks reaction duration using `CURRENT_TIME`.
- **Actions**:
  - Validates inputs: `rCurrentPressure` (0.0–300.0 bar) and `rCurrentTemperature` (0.0–300.0°C) must be finite and within range, else sets `ControlError := TRUE`, closes valves, and logs error.
  - Checks conditions: `ConditionsMet := (ABS(rCurrentPressure - rTargetPressure) <= rPressureTolerance) AND (ABS(rCurrentTemperature - rTargetTemperature) <= rTemperatureTolerance)`.
    - Pressure range: 170.0–180.0 bar (`175.0 ± 5.0`).
    - Temperature range: 183.0–187.0°C (`185.0 ± 2.0`).
  - Manages reaction timing:
    - On rising edge of `ConditionsMet` (i.e., `ConditionsMet = TRUE` AND `LastConditionsMet = FALSE`), captures `ReactionStartTime := CURRENT_TIME` and sets `ReactionTimerActive := TRUE`.
    - If `ConditionsMet = FALSE`, resets `ReactionTimerActive := FALSE` and clears `ReactionStartTime`.
    - If `ReactionTimerActive = TRUE`, checks if `(CURRENT_TIME - ReactionStartTime) >= tReactionTime` (30 minutes).
  - Logs condition changes and timer events: e.g., "2025-05-19 12:45:00 Reaction Conditions Met: Starting Timer", "2025-05-19 12:45:00 Conditions Lost: Timer Reset".
  - Transitions to Step 3 (Shutdown) when reaction duration reaches 30 minutes.
- **Valve Control**:
  - Maintains `stAmmoniaValveControl := TRUE` and `stCO2ValveControl := TRUE` to keep valves open during reaction.
  - If `ControlError = TRUE` (invalid inputs), sets both to `FALSE` to close valves.
- **Timing**:
  - Uses `CURRENT_TIME` (PLC system clock, providing current timestamp) to track duration.
  - `ReactionStartTime` captures the start when conditions are first met.
  - Duration check: `(CURRENT_TIME - ReactionStartTime) >= T#30m` determines if 30 minutes have elapsed.
  - Resets `ReactionStartTime` and `ReactionTimerActive` if conditions are lost, restarting the timer when conditions are regained.
- **Transition**:
  - To Step 3 (Shutdown) when `ReactionTimerActive = TRUE` AND `(CURRENT_TIME - ReactionStartTime) >= T#30m`.
  - Stays in Step 2 if conditions are not met or duration is insufficient.
  - To Step 3 with error if `ControlError = TRUE` (invalid inputs).

#### Step 3: Shutdown
- **Description**: Closes all valves, marks the process as complete, and prepares the reactor for idle state.
- **Actions**:
  - Sets `stAmmoniaValveControl := FALSE` and `stCO2ValveControl := FALSE` to close valves.
  - Sets `stReactionFinished := TRUE` to signal process completion.
  - Resets internal flags: `ReactionTimerActive := FALSE`, `ReactionStartTime := T#0s`, `ConditionsMet := FALSE`.
  - Logs shutdown: e.g., "2025-05-19 12:45:00 Step 3: Shutdown Complete, Reaction Finished".
  - Transitions back to Step 0 (Idle) after completion (assumed reset by external system or operator).
- **Valve Control**:
  - `stAmmoniaValveControl := FALSE`, `stCO2ValveControl := FALSE` to ensure valves are closed.
- **Timing**: No timing active; `CURRENT_TIME` is not used in this step.
- **Transition**:
  - Remains in Step 3 until externally reset to Step 0 (e.g., by operator or supervisory system).
  - If `ControlError = TRUE` from Step 2, enters Step 3 to safely shut down.

### Additional Notes
- **Valve Control Conditions**:
  - Valves are opened (`TRUE`) in Step 1 and Step 2 to load and maintain reactants.
  - Valves are closed (`FALSE`) in Step 0 (Idle), Step 3 (Shutdown), or on `ControlError` to ensure safety.
  - Valve feedback (`stAmmoniaValve`, `stCO2Valve`) confirms physical state, critical for Step 1 transition.
- **Reaction Timing with CURRENT_TIME**:
  - `CURRENT_TIME` provides a high-resolution timestamp (e.g., milliseconds since PLC epoch), updated every scan.
  - Captured in `ReactionStartTime` when `ConditionsMet` transitions to `TRUE`.
  - Duration is calculated as `CURRENT_TIME - ReactionStartTime`, compared against `T#30m` (30 minutes, or 1800000 ms).
  - Timer resets if conditions are lost (`ConditionsMet = FALSE`), ensuring the reaction only completes after 30 continuous minutes of stable conditions.
- **Error Handling**:
  - Invalid inputs (non-finite or out-of-range `rCurrentPressure` or `rCurrentTemperature`) trigger `ControlError`, closing valves and logging errors (e.g., "2025-05-19 12:45:00 Error: Invalid Pressure=350.0 bar").
  - Valve feedback failures (not implemented) could be added as a timeout check in Step 1.
- **Diagnostics**:
  - Logs all state transitions, valve changes, condition changes, timer events, and errors with timestamps.
  - Example: "2025-05-19 12:45:00 Reaction Conditions Met: Pressure=175.2 bar, Temperature=184.8°C".
  - `LogBufferFull` flags when `LogCount >= 50`, indicating need for log retrieval or buffer expansion.

### Assumptions
- The PLC provides `CURRENT_TIME` as a `TIME` type, updated every scan, with millisecond resolution.
- Inputs (`stAmmoniaValve`, `stCO2Valve`, `rCurrentPressure`, `rCurrentTemperature`) are reliable from PLC I/O; faults manifest as out-of-range values or invalid readings.
- Pressure range 0.0–300.0 bar and temperature range 0.0–300.0°C are practical for validation; extreme values indicate sensor faults.
- Valves respond to `stAmmoniaValveControl` and `stCO2ValveControl` with feedback via `stAmmoniaValve` and `stCO2Valve`; actuator delays are handled externally.
- Reaction requires 30 continuous minutes of stable conditions (pressure 170.0–180.0 bar, temperature 183.0–187.0°C); interruptions reset the timer.
- PLC cycle time is approximately 100 ms, suitable for real-time control.
- PLC supports `STRING[80]`, `REAL`, `BOOL`, `INT`, `TIME`, and arrays; larger logs may require adjustments.
- Timestamp is set to example date/time ("2025-05-19 12:45:00"); in practice, use `CURRENT_TIME` or `GET_SYSTEM_TIME`.
- Logging is optional; can be disabled for memory-constrained PLCs.
- External start/reset commands (e.g., to enter Step 1 or return to Step 0) are managed by a supervisory system or operator, not modeled here.
- `ControlError` halts the process by entering Shutdown; recovery requires external reset.

### Example Logic Flow
- **Initial State (Step 0: Idle)**:
  - All outputs off, waiting for start command.
  - Log: "2025-05-19 12:45:00 Process Initialized: Idle State".
- **Start Command**:
  - Transitions to Step 1, sets `Step := 1`.
- **Step 1: Material Loading**:
  - Opens valves: `stAmmoniaValveControl := TRUE`, `stCO2ValveControl := TRUE`.
  - Waits for `stAmmoniaValve = TRUE` and `stCO2Valve = TRUE`.
  - Log: "2025-05-19 12:45:00 Ammonia Valve Opened".
  - Transitions to Step 2 when both valves confirm open.
- **Step 2: Reaction Control**:
  - Inputs: `rCurrentPressure = 175.2` bar, `rCurrentTemperature = 184.8` °C.
  - Validates: Both within range, no `ControlError`.
  - Checks: `ABS(175.2 - 175.0) = 0.2 ≤ 5.0`, `ABS(184.8 - 185.0) = 0.2 ≤ 2.0`, sets `ConditionsMet := TRUE`.
  - On rising edge: Sets `ReactionStartTime := CURRENT_TIME` (e.g., T#2025-05-19T12:45:00Z), `ReactionTimerActive := TRUE`.
  - Log: "2025-05-19 12:45:00 Reaction Conditions Met: Starting Timer".
  - After 30 minutes: `CURRENT_TIME - ReactionStartTime ≥ T#30m`, transitions to Step 3.
  - If conditions lost (e.g., `rCurrentPressure = 165.0` bar), `ConditionsMet := FALSE`, resets timer, logs: "2025-05-19 12:45:00 Conditions Lost: Timer Reset".
- **Step 3: Shutdown**:
  - Closes valves: `stAmmoniaValveControl := FALSE`, `stCO2ValveControl := FALSE`.
  - Sets `stReactionFinished := TRUE`.
  - Log: "2025-05-19 12:45:00 Step 3: Shutdown Complete, Reaction Finished".
  - Awaits external reset to Step 0.
- **Error Case**:
  - `rCurrentPressure := 350.0` bar, sets `ControlError := TRUE`, transitions to Step 3, closes valves, logs: "2025-05-19 12:45:00 Error: Invalid Pressure=350.0 bar".

### Practical Considerations
- **Onboarding**: Clear variable descriptions and step-by-step logic reduce learning curves for new developers, enabling quick understanding of the urea synthesis process.
- **Maintenance**: Structured documentation with state transitions, valve conditions, and timing logic supports debugging (e.g., checking why timer resets) and upgrades (e.g., adding valve timeout).
- **Compliance**: Detailed logs and explicit condition checks (e.g., pressure/temperature tolerances) provide traceability for regulatory audits, ensuring safety and quality standards.
- **Collaboration**: Standardized format (purpose, variables, logic) facilitates communication among control engineers, process engineers, and maintenance teams.
- **Scalability**: Logic flow supports extensions (e.g., adding concentration monitoring, adjustable tolerances) by modifying parameters or adding steps.
- **Diagnostics**: `DiagLog` captures critical events (e.g., valve state changes, condition losses), aiding real-time monitoring and post-process analysis via SCADA/HMI.
- **Timing Precision**: `CURRENT_TIME` ensures accurate duration tracking, but PLC clock resolution (e.g., milliseconds) should be verified; overflows are unlikely for 30-minute durations.

This documentation provides a comprehensive technical reference for the `UreaReactionControl` program, enabling developers and engineers to understand, maintain, and extend the urea synthesis control logic. It supports onboarding, debugging, compliance, and team collaboration, aligning with the specified requirements. If you have the actual code or need specific details (e.g., additional variables, error handling, or code excerpts), please provide them, and I can tailor the documentation further!
