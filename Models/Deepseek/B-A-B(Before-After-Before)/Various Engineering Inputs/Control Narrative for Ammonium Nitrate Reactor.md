Here is a **comprehensive control narrative** for the **Ammonium Nitrate Reactor**, specifically designed to support automation, process safety, and compliance in industrial chemical production. This narrative includes clearly defined setpoints, instrumentation references, control logic, and interlocks—organized for clarity and implementation by engineers or PLC programmers.

---

# 🧪 Ammonium Nitrate Reactor Control Narrative

## 📋 Process Overview

The ammonium nitrate (AN) reactor operates as a **Continuous Stirred-Tank Reactor (CSTR)** where gaseous ammonia (NH₃) reacts with concentrated nitric acid (HNO₃) under controlled conditions:

$$
\text{NH}_3 + \text{HNO}_3 → \text{NH}_4\text{NO}_3
$$

This exothermic neutralization reaction requires precise control of:
- Temperature  
- Pressure  
- Feed ratio (NH₃:HNO₃)  
- pH level  

Failure to maintain these parameters can result in:
- Incomplete reaction  
- Formation of hazardous byproducts (e.g., NOₓ gases)  
- Overpressure events  
- Product quality deviation  

---

## 🔧 Key Equipment and Instrumentation

| Tag | Description | Type |
|-----|-------------|------|
| R-101 | Ammonium Nitrate Reactor | CSTR with agitator |
| TIC-101 | Temperature Indicator & Controller | PT100 sensor + PID loop |
| PIC-102 | Pressure Indicator & Controller | Pressure transmitter |
| FIC-103 | Ammonia feed flow controller | Mass flow meter + control valve |
| FIC-104 | Nitric acid feed flow controller | Coriolis flow meter + control valve |
| pH-105 | pH analyzer | Online pH probe |
| LIC-106 | Level indicator/controller | Guided-wave radar level |
| HHS-107 | High-high level switch | Mechanical float or capacitance type |
| ESD-108 | Emergency shutdown system | PLC-based interlock |

---

## 🎯 Key Setpoints and Operating Ranges

| Parameter | Setpoint | Acceptable Range | Tolerance |
|----------|----------|------------------|-----------|
| Reaction Temperature | 175 °C | 170–180 °C | ±2 °C |
| Reactor Pressure | 4.8 bar | 4.5–5.0 bar | ±0.2 bar |
| NH₃/HNO₃ Molar Ratio | 1.01:1 | 1.00–1.02:1 | ±0.01 |
| pH Value | 6.25 | 6.0–6.5 | ±0.1 |
| Reactor Level | 50% | 40–60% | ±5% |

---

## 🔁 Control Loops and Logic Descriptions

### 🟢 Temperature Control Loop – TIC-101
- **Objective:** Maintain stable reaction temperature.
- **Action:** Adjust cooling water flow via control valve.
- **Logic:**
```pascal
IF Temp > 177°C THEN
    Open(CoolingValve);
ELSIF Temp < 173°C THEN
    Close(CoolingValve);
END_IF;
```

### 🟡 Pressure Control Loop – PIC-102
- **Objective:** Regulate vapor pressure inside the reactor.
- **Action:** Modulate vent valve position.
- **Logic:**
```pascal
IF Pressure > 4.9 bar THEN
    Open(VentValve);
ELSIF Pressure < 4.7 bar THEN
    Close(VentValve);
END_IF;
```

### 🔵 Flow Ratio Control – FIC-103 / FIC-104
- **Objective:** Maintain stoichiometric balance between reactants.
- **Master Variable:** Nitric acid flow rate (FIC-104).
- **Slave Variable:** Ammonia flow rate (FIC-103) adjusted to match at 1.01:1 ratio.
- **Logic:**
```pascal
Setpoint_FIC103 := Measured_FIC104 * 1.01;

IF FIC103.Actual < Setpoint_FIC103 THEN
    Increase(AmmoniaValve);
ELSE
    Decrease(AmmoniaValve);
END_IF;
```

### 🔴 pH Control Loop – pH-105
- **Objective:** Ensure complete neutralization and prevent acidic product.
- **Action:** Fine-tune ammonia feed based on pH reading.
- **Logic:**
```pascal
IF pH < 6.0 THEN
    Increase(FIC103); // More NH3
ELSIF pH > 6.5 THEN
    Decrease(FIC103); // Less NH3
END_IF;
```

---

## ⚙️ Startup Procedure

### ✅ Phase 1: Pre-Startup Checks
- Confirm no alarms active
- Verify all valves closed
- Check minimum cooling water availability
- Ensure emergency stop is not engaged

### ✅ Phase 2: System Priming
- Start circulation pump
- Open nitric acid inlet valve slowly
- Ramp up to nominal flow rate over 60 seconds

### ✅ Phase 3: Introduce Ammonia
- Slowly open ammonia valve when nitric acid flow is stable
- Monitor pH and temperature response
- Engage ratio control once steady-state reached

### ✅ Phase 4: Stabilize Reaction
- Allow system to reach target temperature and pressure
- Enable full control loops (TIC-101, PIC-102, pH-105)
- Confirm product sample meets quality specs

---

## ⚠️ Shutdown Logic

### 🟡 Normal Shutdown
- Stop ammonia feed
- Stop nitric acid feed
- Drain reactor contents if required
- Disable all controllers
- Reset alarm flags

### 🔴 Emergency Shutdown Conditions (ESD-108)
Trigger emergency shutdown if any of the following occur:
- Temperature > 185 °C
- Pressure > 5.2 bar
- Level > 80% (LIC-106)
- High-high level detected (HHS-107)
- pH < 5.5 or > 7.0
- Cooling water failure
- Operator presses E-stop

```pascal
IF Temp > 185 OR Pressure > 5.2 OR Level > 80 OR pH < 5.5 OR pH > 7.0 THEN
    TriggerEmergencyShutdown();
END_IF;
```

During emergency shutdown:
- Close all feed valves
- Open emergency vent
- Activate alarm
- Log event in historian

---

## 📌 Summary Table: Critical Parameters and Actions

| Parameter | Action Taken | Output Device |
|----------|--------------|---------------|
| Temp > 185 °C | Initiate ESD | Cooling valve closed, ESD activated |
| Pressure > 5.2 bar | Initiate ESD | Vent valve opened, feeds stopped |
| Level > 80% | Alarm then ESD | Pump stopped, feed valves closed |
| pH < 5.5 or > 7.0 | Alarm then ESD | Ammonia valve adjusted/stopped |
| Cooling Failure | Initiate ESD | Emergency vent opened |

---

## 📄 Final Notes

This control narrative provides a detailed foundation for:
- **PLC programming** using IEC 61131-3 languages (ST, LD, FBD)
- **SCADA configuration** with tags and alarms
- **Safety system integration** (SIS/ESD)
- **Operator training** and SOP development
- **Regulatory documentation**

Would you like this exported into a formal Word/PDF format? Or would you like a similar control narrative developed for downstream processes such as crystallization, prilling, or granulation?
