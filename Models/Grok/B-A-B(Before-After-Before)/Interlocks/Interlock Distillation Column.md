(* P&I Diagram: Distillation Column C-101 *)
(* Description: Text-based representation of a distillation column with control loops and safety interlocks *)
(* Components and Tagnames: *)
(* - C-101: Distillation Column *)
(* - E-101: Reboiler *)
(* - E-102: Condenser *)
(* - LT-101: Level Transmitter (Bottoms Level) *)
(* - PT-101: Pressure Transmitter (Column Top) *)
(* - TT-101: Temperature Transmitter (Reboiler Outlet) *)
(* - FV-101: Feed Valve (Inlet to C-101) *)
(* - PRV-101: Pressure Relief Valve (Top of C-101) *)
(* - RV-101: Reboiler Supply Valve (Steam to E-101) *)

(* P&I Diagram (Text-Based) *)
(*
  [Feed Stream] ----> [FV-101] ----> [C-101: Distillation Column]
                                     |
                                     | (Vapor) ----> [E-102: Condenser]
                                     |                  |
                                     |                  | [Distillate Out]
                                     | (Pressure) <---- [PT-101]
                                     |                  |
                                     |                  | ----> [PRV-101] ----> [Relief Line]
                                     |
                                     | (Liquid) ----> [E-101: Reboiler]
                                     |                  |
                                     |                  | [Bottoms Out]
                                     | (Level) <------- [LT-101]
                                     |                  |
                                     | (Temperature) <-- [TT-101]
                                     |                  |
                                     |                  | <---- [RV-101] <---- [Steam Supply]
*)

(* Control Loops *)
(* - LC-101: LT-101 controls bottoms level by adjusting bottoms outlet valve (not shown). *)
(* - PC-101: PT-101 modulates PRV-101 for pressure control (interlock override). *)
(* - TC-101: TT-101 regulates RV-101 for reboiler temperature control (interlock override). *)
(* - FC-101: Feed flow controlled by FV-101 (interlock override). *)

(* Safety Interlocks *)
(* - Close FV-101 if PT-101 < 50 psi (prevent vacuum conditions). *)
(* - Open PRV-101 if PT-101 > 120 psi (prevent overpressure). *)
(* - Close RV-101 if TT-101 > 180°C (prevent overheating). *)
(* - Trigger high-level alarm if LT-101 > 90% (prevent flooding). *)

(* IEC 61131-3 Structured Text Program *)
PROGRAM DISTILLATION_COLUMN_INTERLOCKS
VAR
    (* Inputs *)
    PT_101 : REAL;          (* Pressure in psi, from PT-101 *)
    TT_101 : REAL;          (* Temperature in °C, from TT-101 *)
    LT_101 : REAL;          (* Level in %, from LT-101 *)
    
    (* Outputs *)
    FV_101 : BOOL;          (* Feed Valve: TRUE = Open, FALSE = Closed *)
    PRV_101 : BOOL;         (* Pressure Relief Valve: TRUE = Open, FALSE = Closed *)
    RV_101 : BOOL;          (* Reboiler Supply Valve: TRUE = Open, FALSE = Closed *)
    HIGH_LEVEL_ALARM : BOOL; (* High Level Alarm: TRUE = Active *)
    
    (* Internal Variables *)
    PRESSURE_LOW_LIMIT : REAL := 50.0;    (* Low pressure threshold, psi *)
    PRESSURE_HIGH_LIMIT : REAL := 120.0;  (* High pressure threshold, psi *)
    TEMP_LIMIT : REAL := 180.0;           (* High temperature threshold, °C *)
    LEVEL_ALARM_LIMIT : REAL := 90.0;     (* High level alarm threshold, % *)
END_VAR

(* Initialize outputs *)
FV_101 := TRUE;      (* Feed valve initially open *)
PRV_101 := FALSE;    (* Relief valve initially closed *)
RV_101 := TRUE;      (* Reboiler valve initially open *)
HIGH_LEVEL_ALARM := FALSE;

(* Interlock Logic *)
(* Interlock 1: Close FV-101 if PT-101 < 50 psi *)
IF PT_101 < PRESSURE_LOW_LIMIT THEN
    FV_101 := FALSE;
END_IF

(* Interlock 2: Open PRV-101 if PT-101 > 120 psi *)
IF PT_101 > PRESSURE_HIGH_LIMIT THEN
    PRV_101 := TRUE;
ELSE
    PRV_101 := FALSE;
END_IF

(* Interlock 3: Close RV-101 if TT-101 > 180°C *)
IF TT_101 > TEMP_LIMIT THEN
    RV_101 := FALSE;
END_IF

(* Interlock 4: Trigger high-level alarm if LT-101 > 90% *)
IF LT_101 > LEVEL_ALARM_LIMIT THEN
    HIGH_LEVEL_ALARM := TRUE;
ELSE
    HIGH_LEVEL_ALARM := FALSE;
END_IF

END_PROGRAM
