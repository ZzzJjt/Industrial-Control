TEXT-BASED P&ID - DISTILLATION COLUMN (C-101)

  ┌──────────────────────────────────────┐
  │              C-101                  │
  │          (Distillation Column)      │
  │    ┌────────────────────────────┐   │
  │    │                            │   │
  │    │                            │   │
  │    │                            │   │
  │    │                            │   │
  │    │  LT-101        TT-101      │   │
  │    │  (Level Tx)    (Temp Tx)   │   │
  │    └─────┬────────────┬────────┘   │
  │          │            │            │
  │        FV-101        E-101         │
  │     (Feed Valve)   (Reboiler)      │
  └──────────┼────────────┼────────────┘
             │            │
         PT-101        PRV-101
      (Pressure Tx)   (Relief Valve)
             │
         E-102 (Condenser)


       (* Distillation Column Interlock Logic *)

IF PT_101 > 120.0 THEN
    PRV_101 := TRUE;        // Open relief valve to relieve overpressure
ELSE
    PRV_101 := FALSE;
END_IF;

IF PT_101 < 50.0 THEN
    FV_101 := FALSE;        // Close feed valve to avoid vacuum
END_IF;

IF TT_101 > 180.0 THEN
    E_101_Heating := FALSE; // Shut off reboiler heating
END_IF;
