Control Narrative for Ammonium Nitrate Reactor

1. Process Overview

Purpose:
Neutralize nitric acid with ammonia to produce ammonium nitrate.

Reactor Type:
Continuous Stirred Tank Reactor (CSTR)

Normal Flow Conditions and Operational Limits:
	•	Continuous operation with consistent mixing
	•	Flow controlled to maintain ammonia:nitric acid molar ratio close to 1.01:1
	•	Operational conditions maintained to ensure optimal temperature, pressure, and pH conditions for reaction stability and safety

2. Key Setpoints and Ranges

Parameter	Tag	Setpoint	Acceptable Range
Temperature	TIC-101	175 °C	± 2 °C
Pressure	PIC-102	4.8 bar	± 0.2 bar
Ammonia:Acid Flow Ratio	FRC-103	1.01:1	± 0.01
pH	AIC-105	6.2	5.8–6.5

3. Instrumentation and Equipment

Control Instruments:
	•	Temperature transmitter and PID Controller (TIC-101)
	•	Pressure transmitter and PID Controller (PIC-102)
	•	Flow ratio controller with transmitters (FIC-103, FIC-104)
	•	pH analyzer and PID Controller (AIC-105)

Safety Instruments and Interlocks:
	•	High-pressure safety switch
	•	High-temperature alarm
	•	Emergency shutdown valve (ESD-106), automatically activated by interlock conditions

4. Sequence of Operation

Startup Procedure:
	1.	Initialize all controllers (TIC-101, PIC-102, FRC-103, AIC-105).
	2.	Gradually ramp up ammonia and nitric acid feed flows.
	3.	Preheat reactor contents to operating temperature (175 °C).
	4.	Activate ammonia:nitric acid flow ratio control loop.
	5.	Activate pH control loop to maintain desired acidity.

Steady-State Operation:
	1.	Maintain PID control loops for temperature, pressure, flow ratio, and pH.
	2.	Continuously monitor operational parameters for deviations.
	3.	Log all critical parameters regularly.
	4.	Respond promptly to deviation alarms.

Shutdown Procedure:
	1.	Initiate controlled isolation of ammonia and nitric acid feeds.
	2.	Depressurize reactor gradually to atmospheric pressure.
	3.	Deactivate heating elements.
	4.	Drain reactor contents to designated holding tanks or treatment units.

5. Interlocks and Alarms

Condition	Action
Temperature exceeds 185 °C	Emergency shutdown initiated (activate ESD-106)
Pressure exceeds 5.2 bar	Automatically close ammonia and nitric acid feed valves
pH drops below 5.5	Trigger alarm and reduce ammonia flow immediately

These interlock conditions are integrated to ensure immediate response to deviations, maintaining safe operational limits and protecting personnel and equipment.
