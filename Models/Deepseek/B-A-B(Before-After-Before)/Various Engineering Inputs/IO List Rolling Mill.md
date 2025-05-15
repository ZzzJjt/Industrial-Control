Below is a structured I/O list table tailored for a rolling mill system in a steel production plant. The table includes 30 entries with key details such as Signal Name, Input/Output type, Signal Tagnumber, Analog/Digital classification, Engineering Unit, Range, Setpoint, and P&ID Reference. This table can be easily adapted into Excel, PDF datasheets, or used directly within PLC documentation.

### Rolling Mill System I/O List

| Signal Name        | Input/Output | Signal Tagnumber | Analog/Digital | Engineering Unit | Range           | Setpoint       | P&ID Reference |
|--------------------|--------------|------------------|----------------|------------------|-----------------|----------------|----------------|
| Entry Tension      | Input        | TEN-01           | Analog         | kN               | 0 - 50          | 25             | P&ID-01A       |
| Exit Tension       | Input        | TEN-02           | Analog         | kN               | 0 - 50          | 25             | P&ID-01B       |
| Roll Force         | Input        | FOR-01           | Analog         | kN               | 0 - 2000        | 1000           | P&ID-02A       |
| Roll Speed Feedback| Input        | SPD-01           | Analog         | m/min            | 0 - 500         | 250            | P&ID-03A       |
| Motor Current      | Input        | CUR-01           | Analog         | A                | 0 - 500         | 250            | P&ID-04A       |
| Hydraulic Pressure | Input        | HYP-01           | Analog         | bar              | 0 - 300         | 150            | P&ID-05A       |
| Cooling Water Temp | Input        | CWT-01           | Analog         | °C               | 0 - 100         | 40             | P&ID-06A       |
| Oil Temperature    | Input        | OIT-01           | Analog         | °C               | 0 - 100         | 50             | P&ID-07A       |
| Emergency Stop     | Input        | EMO-01           | Digital        | N/A              | N/A             | N/A            | P&ID-08A       |
| Start Command      | Output       | STC-01           | Digital        | N/A              | N/A             | N/A            | P&ID-09A       |
| Stop Command       | Output       | SPC-01           | Digital        | N/A              | N/A             | N/A            | P&ID-10A       |
| Roll Gap Adjust    | Output       | RGA-01           | Analog         | mm               | 0 - 50          | 25             | P&ID-11A       |
| Cooling Valve Cmd  | Output       | CLV-01           | Digital        | N/A              | N/A             | N/A            | P&ID-12A       |
| Heating Element Cmd| Output       | HEC-01           | Digital        | N/A              | N/A             | N/A            | P&ID-13A       |
| Lubrication Pump   | Output       | LUB-01           | Digital        | N/A              | N/A             | N/A            | P&ID-14A       |
| Guide Position     | Input        | GUP-01           | Analog         | mm               | 0 - 1000        | 500            | P&ID-15A       |
| Strip Width        | Input        | SPW-01           | Analog         | mm               | 500 - 2000      | 1000           | P&ID-16A       |
| Edge Trim Cmd      | Output       | ETC-01           | Digital        | N/A              | N/A             | N/A            | P&ID-17A       |
| Thickness Gauge    | Input        | THG-01           | Analog         | mm               | 0 - 10          | 5              | P&ID-18A       |
| Surface Quality    | Input        | SUQ-01           | Digital        | N/A              | N/A             | N/A            | P&ID-19A       |
| Material Type      | Input        | MAT-01           | Digital        | N/A              | N/A             | N/A            | P&ID-20A       |
| Coil Weight        | Input        | CWG-01           | Analog         | kg               | 0 - 25000       | 12500          | P&ID-21A       |
| Coil Diameter      | Input        | COD-01           | Analog         | mm               | 500 - 2000      | 1000           | P&ID-22A       |
| Coil Length        | Input        | COL-01           | Analog         | m                | 0 - 5000        | 2500           | P&ID-23A       |
| Coil Tension       | Input        | COT-01           | Analog         | kN               | 0 - 50          | 25             | P&ID-24A       |
| Coil Alignment     | Input        | COA-01           | Analog         | mm               | -100 - 100      | 0              | P&ID-25A       |
| Scrap Conveyor Cmd | Output       | SCC-01           | Digital        | N/A              | N/A             | N/A            | P&ID-26A       |
| Waste Bin Level    | Input        | WBL-01           | Analog         | %                | 0 - 100         | 50             | P&ID-27A       |
| Dust Extraction Cmd| Output       | DEC-01           | Digital        | N/A              | N/A             | N/A            | P&ID-28A       |
| Alarm Reset        | Output       | ALR-01           | Digital        | N/A              | N/A             | N/A            | P&ID-29A       |

This table provides a comprehensive overview of the signals involved in controlling a rolling mill, ensuring that all aspects from instrumentation to PLC programming are aligned and consistent. It serves as a crucial document for engineers working on the project, facilitating smooth integration and operation of the control systems.
