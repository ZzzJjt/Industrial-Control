
### Overall Process Structure

The cocoa milk production process involves the following stages:
1. **Ingredient Addition**: Adding milk, water, liquid sugar, and cocoa.
2. **Heating**: Raising the temperature to the desired level.
3. **Blending**: Stirring the mixture at a specified speed for a set duration.

#### Required Ingredients and Quantities
- **Milk**: 50 kg
- **Water**: 40 kg
- **Liquid Sugar**: 5 kg
- **Cocoa**: 5 kg

#### Processing Parameters
- **Heating Temperature**: 70°C
- **Stirring Speed**: 200 RPM
- **Mixing Duration**: 10 minutes

### Structured Text (ST) Program



### Explanation of the Code

1. **Inputs and Outputs**:
   - `StartButton`: Initiates the start of the mixing process.
   - `StopButton`: Stops the mixing process prematurely.
   - `EmergencyStop`: Stops the machine immediately and safely.
   - `IngredientValves`: Array of boolean values controlling the valves for milk, water, liquid sugar, and cocoa.
   - `HeaterActive`: Output to control the heater.
   - `MixerSpeed`: Output to control the mixer speed.

2. **Internal Variables**:
   - `CurrentPhase`: Tracks the current phase of the mixing process (0 = Idle, 1 = Ingredient Addition, 2 = Heating, 3 = Blending).
   - `Running`: Indicates whether the mixing process is running.
   - `EmergencyStopped`: Indicates whether an emergency stop has occurred.
   - `TargetTemperature`: Desired temperature for the heating phase.
   - `TargetSpeed`: Desired stirring speed for the blending phase.
   - `MixingDuration`: Duration for the blending phase.
   - `TimerAddition`, `TimerHeating`, `TimerBlending`: Timers for each phase.
   - `CurrentTemperature`: Simulated temperature reading from a sensor.

3. **Initialization**:
   - All outputs are initialized to their default states (e.g., ingredient valves closed, heater off, mixer speed zero).

4. **Main Control Loop**:
   - Handles emergency stops by deactivating all components and resetting flags.
   - Manages transitions between phases based on inputs and conditions.

5. **Phase Control Logic**:
   - **Ingredient Addition Phase**: Opens the valves for milk, water, liquid sugar, and cocoa. Closes the valves after a 5-minute timer expires.
   - **Heating Phase**: Activates the heater and simulates temperature increase. Transitions to the Blending phase when the target temperature is reached.
   - **Blending Phase**: Sets the mixer speed to the target value and starts the Blending timer. Ends the mixing process when the timer expires.

6. **Modular Methods**:
   - `AddIngredients()`: Controls the opening and closing of ingredient valves and manages the addition time.
   - `StartHeating()`: Activates the heater and simulates temperature increase.
   - `StartBlending()`: Sets the mixer speed and starts the Blending timer.

This structured text program ensures precise execution of the mixing process in cocoa milk production, adheres to ISA-88 principles, and supports maintainability and scalability. It can be easily adjusted for future changes in batch size or formula while maintaining compliance with ISA-88 standards.
