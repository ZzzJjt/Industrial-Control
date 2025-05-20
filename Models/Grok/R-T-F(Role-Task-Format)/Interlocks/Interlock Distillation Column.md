# P&ID: Distillation Column System (C-101)

## Equipment
- **C-101**: Distillation Column
  - Processes feed into distillate and bottoms products
- **E-101**: Reboiler
  - Provides heat to C-101 via steam or hot oil
- **E-102**: Condenser
  - Cools and condenses vapor from C-101 top
- **T-101**: Reflux Drum
  - Collects condensed distillate from E-102
- **P-101**: Feed Pump
  - Delivers feed to C-101

## Instruments and Actuators
- **PT-101**: Pressure Transmitter
  - Location: Top of C-101
  - Measures: Column pressure (psi)
  - Range: 0–150 psi
- **TT-101**: Temperature Transmitter
  - Location: Bottom of C-101 (near E-101)
  - Measures: Reboiler temperature (°C)
  - Range: 50–200°C
- **LT-101**: Level Transmitter
  - Location: T-101 (Reflux Drum)
  - Measures: Liquid level (%)
  - Range: 0–100%
- **FV-101**: Feed Valve
  - Location: Feed line to C-101 (downstream of P-101)
  - Type: Control valve, fail-closed
  - Controls: Feed flow to C-101
- **PRV-101**: Pressure Relief Valve
  - Location: Top of C-101 (vapor line to flare/vent)
  - Type: Spring-loaded, self-actuating
  - Setpoint: Opens at 120 psi
- **HV-101**: Reboiler Heating Valve
  - Location: Steam/hot oil supply to E-101
  - Type: Control valve, fail-closed
  - Controls: Heat input to reboiler

## Control Loops
- **LIC-101**: Level Control Loop
  - Input: LT-101 (Reflux Drum level)
  - Output: Manipulates LV-101 (Reflux Drum outlet valve, not shown)
  - Setpoint: Maintains 50% level in T-101
- **TIC-101**: Temperature Control Loop
  - Input: TT-101 (Reboiler temperature)
  - Output: Manipulates HV-101 (Reboiler heating valve)
  - Setpoint: Maintains 150°C in reboiler
- **FIC-101**: Feed Flow Control Loop
  - Input: FT-101 (Feed flow transmitter, not shown)
  - Output: Manipulates FV-101 (Feed valve)
  - Setpoint: Maintains desired feed rate

## Safety Interlocks
- **Interlock I-101**: High Pressure Protection
  - Condition: PT-101 > 120 psi
  - Action: Open PRV-101 to vent excess pressure
  - Priority: High
  - Status: Alarm, requires operator reset
- **Interlock I-102**: Low Pressure Protection
  - Condition: PT-101 < 50 psi
  - Action: Close FV-101 to stop feed
  - Priority: Medium
  - Status: Alarm
- **Interlock I-103**: High Temperature Protection
  - Condition: TT-101 > 180°C
  - Action: Close HV-101 to stop heating
  - Priority: High
  - Status: Alarm, requires operator reset

## Notes
- All interlocks trigger alarms to the control room via DCS/SCADA.
- PRV-101 is self-actuating but monitored by PT-101 for logging.
- FV-101 and HV-101 are fail-closed to ensure safe state on power loss.
- Interlocks are implemented in PLC using IEC 61131-3 Structured Text.
