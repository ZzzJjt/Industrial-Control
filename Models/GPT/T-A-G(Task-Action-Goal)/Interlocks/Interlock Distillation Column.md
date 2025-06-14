**Text-Based P&ID - Distillation Column System**

Feed Tank --> FV-101 --> C-101 (Distillation Column)
                                  |
                                  v
                           LT-101 (Level Transmitter)
                                  |
                                  v
                                E-101 (Reboiler) <--> TT-101 (Temp Transmitter)
                                  |
                                  v
                              Bottoms Outlet

C-101 (Top) --> E-102 (Condenser) --> Product Tank
                            |
                            v
                       PT-101 (Pressure Transmitter)
                            |
                            v
                       PRV-101 (Pressure Relief Valve)


**IEC 61131-3 Structured Text - Interlock Logic**

VAR_INPUT
    PT_101 : REAL; (* Pressure in psi *)
    TT_101 : REAL; (* Temperature in deg C *)
    LT_101 : REAL; (* Level in % *)
END_VAR

VAR_OUTPUT
    PRV_101_Open : BOOL := FALSE;  (* Relief valve activation *)
    FV_101_Open  : BOOL := TRUE;   (* Feed valve status *)
    Heater_E101_On : BOOL := TRUE; (* Reboiler heating *)
END_VAR

(* Interlock Logic *)

(* Overpressure protection *)
IF PT_101 > 120.0 THEN
    PRV_101_Open := TRUE;  (* Open pressure relief valve *)
END_IF;

(* Underpressure protection *)
IF PT_101 < 50.0 THEN
    FV_101_Open := FALSE;  (* Stop feed to prevent vacuum or unstable operation *)
END_IF;

(* Overtemperature protection *)
IF TT_101 > 180.0 THEN
    Heater_E101_On := FALSE;  (* Shut off reboiler heating to prevent degradation *)
END_IF;


**Explanation of Interlocks**

1. **PT-101 > 120 psi triggers PRV-101**: Prevents vessel overpressure, reducing explosion risk and maintaining mechanical integrity.
2. **PT-101 < 50 psi closes FV-101**: Avoids drawing in vapors under vacuum conditions, protecting column stability.
3. **TT-101 > 180°C shuts off E-101**: Prevents thermal degradation of contents and heat exchanger damage, enhancing product quality and safety.

These interlocks ensure the column operates within its safe design envelope, reducing the likelihood of process failure, protecting equipment, and enabling safe shutdown procedures.
