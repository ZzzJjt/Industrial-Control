Here is a **comprehensive control narrative** for the beer brewing process, with a particular focus on **Section 4 – The Lautering Process**, as requested. This narrative includes clear setpoints, instrumentation, and automation logic to support consistent design, commissioning, and operation of brewery automation systems.

---

# 🍺 Brewery Automation Control Narrative

## 🧾 Overview of Brewing Stages and Control Points

The brewing process consists of several key stages, each with defined control parameters and instrumentation:

### 🔹 Stage 1: Mashing
- **Temperature Setpoint:** 65–68 °C  
- **Tolerance:** ±0.5 °C  
- **Control Logic:** PID loop using temperature sensor (PT100) and heating element or steam valve  
- **Instrumentation:** Temperature transmitter, level switch in mash tun

### 🔹 Stage 2: Sparge Water Heating
- **Setpoint:** 75–78 °C  
- **Control Logic:** On/off heater control with high-limit safety thermostat  
- **Instrumentation:** Temperature sensor, flow switch on sparge water line

### 🔹 Stage 3: Wort Boiling
- **Boil Duration:** 60–90 minutes  
- **Temperature Monitoring:** Continuous with alarm at <95 °C  
- **Instrumentation:** High-temperature probe, timer, level sensors in kettle

---

# 🟨 Section 4: Lautering Process

Lautering is the process of separating the liquid wort from spent grain following mashing. It involves precise coordination of mechanical components, flow rates, and quality monitoring (e.g., turbidity).

## 🛠 Equipment and Instrumentation

| Component | Description |
|----------|-------------|
| **Lauter Tun** | Vessel containing mash bed for filtration |
| **Rake Arm System** | Mechanical device that gently agitates the grain bed to prevent compaction |
| **Underlet System** | Perforated pipes beneath the false bottom to evenly distribute sparge water |
| **Flowmeters** | Measure wort flow rate from lauter tun (setpoint: 1.5–2.0 L/min/kg grain) |
| **Level Transmitters** | Monitor wort level in holding tank (setpoint: 50–80%) |
| **Temperature Sensors** | Monitor wort and sparge water temperatures |
| **Turbidity Sensor** | Detects clarity of wort; used to determine when lautering should end |
| **Motor Drives** | Control rake arm movement and pump speeds |

---

## 📋 Step-by-Step Lautering Procedure with Automation Logic

### ✅ Phase 1: Preparation
- **Start Conditions:**
  - Mash complete and transfer confirmed
  - Lauter tun empty and drain valve closed
  - Holding tank level < 80%

- **Actions:**
  - Open mash transfer valve
  - Start mash pump until level in lauter tun reaches low-level threshold (LL)
  - Close transfer valve once LL is reached

```pascal
IF MashTransferComplete AND NOT LauterTunFull THEN
    Open(MashValve);
    Start(PumpMash);
END_IF;
```

---

### ✅ Phase 2: Initial Drainage
- **Objective:** Collect first runnings while avoiding grain bed disturbance
- **Setpoint:** Flow rate = 1.5 L/min/kg grain
- **Logic:**
  - Ramp up wort pump slowly over 30 seconds
  - Monitor turbidity; divert to waste if > threshold

```pascal
IF FirstRunnings THEN
    RampUpPumpOver(30s);
    IF Turbidity > Threshold THEN
        DivertTo(WasteTank);
    END_IF;
END_IF;
```

---

### ✅ Phase 3: Sparge Water Addition
- **Start Condition:** First runnings completed or turbidity acceptable
- **Setpoint:** Sparge water temp = 75–78 °C
- **Logic:**
  - Activate sparge water pump
  - Maintain constant sparge rate based on grain mass
  - Stop sparge if holding tank level < minimum

```pascal
IF FirstRunningsDone THEN
    Start(SpargePump);
    IF TankLevel < MinLevel THEN
        Stop(SpargePump);
    END_IF;
END_IF;
```

---

### ✅ Phase 4: Rake Arm Operation
- **Purpose:** Prevent compaction and improve wort extraction
- **Sequence:**
  - Rake lowers to set depth (based on grain bed height)
  - Rotates slowly for X revolutions
  - Retracts after cycle completion
- **Safety Logic:**
  - Stop rake if torque exceeds limit
  - Raise rake if emergency stop triggered

```pascal
IF LauteringActive THEN
    Lower(RakeArm, TargetDepth);
    Rotate(RakeArm, Revolutions := 3);
    Raise(RakeArm);
END_IF;

IF TorqueSensor > MaxTorque THEN
    EmergencyStop();
    Raise(RakeArm);
END_IF;
```

---

### ✅ Phase 5: End of Lautering
- **Termination Conditions:**
  - Holding tank full (level > 80%)
  - Turbidity > max allowable value
  - Total volume collected >= batch target

- **Action:**
  - Stop all pumps
  - Close valves
  - Signal "Lautering Complete"

```pascal
IF TankLevel > 80% OR Turbidity > Max OR BatchVolumeReached THEN
    StopAllPumps();
    CloseValves();
    LauteringComplete := TRUE;
END_IF;
```

---

## 🎛 Optional Advanced Features (Recommended)

| Feature | Purpose |
|--------|---------|
| **Adaptive Flow Rate Control** | Adjust flow based on real-time turbidity feedback |
| **Rake Depth Profiling** | Use encoder feedback to vary rake depth during cycles |
| **Batch Logging** | Record start/stop times, flow totals, and turbidity values per batch |
| **Alarm Management** | Generate alarms for high turbidity, low sparge flow, or motor overload |

---

## 📌 Summary of Key Setpoints and Operating Ranges

| Parameter | Setpoint Range | Tolerance |
|----------|----------------|-----------|
| Mash Temp | 65–68 °C | ±0.5 °C |
| Sparge Water Temp | 75–78 °C | ±1 °C |
| Wort Flow Rate | 1.5–2.0 L/min/kg grain | ±0.2 L/min |
| Holding Tank Level | 50–80% | ±5% |
| Turbidity Limit | < 5 NTU | ±1 NTU |
| Rake Cycle Time | 5–10 min/cycle | ±1 min |

---

## ✅ Final Notes

This control narrative provides a structured, modular, and instrumented approach to lautering automation. It can be directly used by PLC programmers to implement IEC 61131-3 logic, and by commissioning engineers to verify system behavior. By defining clear setpoints, logical conditions, and instrumentation requirements, this document ensures consistency across design, implementation, and operation phases of brewery automation projects.

Would you like this narrative exported into a formal Word or PDF document? Or would you like the next section (e.g., boiling or fermentation) developed similarly?
