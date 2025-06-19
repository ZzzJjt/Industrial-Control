
### ISA-88 Compliant Batch Control Recipe for Adhesive Production - Reaction Step

#### Overview
The Reaction step consists of three main phases:
1. **Heating**: Start heating the reactor.
2. **Mixing**: Start mixing the contents.
3. **Holding**: Hold the reaction at a specific temperature for a set duration.

Each phase has its own process parameters and timers. The program includes modular methods (`StartHeating`, `StartMixing`, `HoldReaction`) and uses conditional checks and timer values to control transitions between these phases.

#### Structured Text (ST) Program



### Explanation of the Code

1. **Inputs and Outputs**:
   - `StartButton`: Initiates the start of the Reaction step.
   - `StopButton`: Stops the Reaction step prematurely.
   - `EmergencyStop`: Stops the machine immediately and safely.
   - `HeaterActive`: Output to control the heater.
   - `MixerActive`: Output to control the mixer.

2. **Internal Variables**:
   - `CurrentPhase`: Tracks the current phase of the Reaction step (0 = Idle, 1 = Heating, 2 = Mixing, 3 = Holding).
   - `Running`: Indicates whether the Reaction step is running.
   - `EmergencyStopped`: Indicates whether an emergency stop has occurred.
   - `TargetTemperature`: Desired temperature for the heating phase.
   - `MixingDuration`: Duration for the mixing phase.
   - `HoldingDuration`: Duration for the holding phase.
   - `TimerHeating`, `TimerMixing`, `TimerHolding`: Timers for each phase.
   - `CurrentTemperature`: Simulated temperature reading from a sensor.

3. **Initialization**:
   - All outputs are initialized to their default states (e.g., heater and mixer off).

4. **Main Control Logic**:
   - If `EmergencyStop` is active, the machine enters an emergency stop state, deactivating all components.
   - If `StartButton` is pressed and the Reaction step is not already running or in an emergency stop state, the Heating phase begins.
   - If `StopButton` is pressed and the Reaction step is running, the process stops.

5. **Phase Control Logic**:
   - **Heating Phase**: Activates the heater and simulates temperature increase. Transitions to the Mixing phase when the target temperature is reached.
   - **Mixing Phase**: Activates the mixer and starts the Mixing timer. Transitions to the Holding phase when the timer expires.
   - **Holding Phase**: Starts the Holding timer. Ends the Reaction step when the timer expires.

6. **Emergency Stop Handling**:
   - Ensures all components are deactivated if an emergency stop occurs.
   - Resets the emergency stop flag after handling.

7. **Modular Methods**:
   - `StartHeating()`: Activates the heater and simulates temperature increase.
   - `StartMixing()`: Activates the mixer and starts the Mixing timer.
   - `HoldReaction()`: Starts the Holding timer.

This structured text program ensures precise execution of the Reaction step in adhesive production, adheres to ISA-88 principles, and supports maintainability and reusability across similar batch processes.
