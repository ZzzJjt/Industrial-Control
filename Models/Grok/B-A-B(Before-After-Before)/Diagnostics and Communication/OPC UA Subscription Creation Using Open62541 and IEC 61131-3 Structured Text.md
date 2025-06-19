(* Function Block: OPC_UA_SUBSCRIBE
   Purpose: Creates an OPC UA subscription using external C-code.
   Features:
   - Initiates subscription creation on rising edge of EXECUTE
   - Sets priority and publishing interval
   - Manages subscription lifecycle with timeout
   - Provides status outputs (Done, Busy, Error, ErrorID, SubscriptionHdl)
   - Interfaces with C-code via external function calls
*)

FUNCTION_BLOCK OPC_UA_SUBSCRIBE
VAR_INPUT
    EXECUTE : BOOL;               (* Rising edge triggers subscription creation *)
    ConnectionHdl : DWORD;        (* OPC UA client session handle *)
    Priority : BYTE;              (* Subscription priority, 0-255 *)
    Timeout : TIME;               (* Timeout for subscription creation *)
    PublishingInterval : TIME;     (* Requested publishing interval, e.g., T#100ms *)
END_VAR

VAR_OUTPUT
    DONE : BOOL;                  (* TRUE when subscription is created *)
    BUSY : BOOL;                  (* TRUE during subscription creation *)
    ERROR : BOOL;                 (* TRUE if creation fails *)
    ERROR_ID : DWORD;             (* Error code: 0=None, 1=Invalid Handle, 2=Invalid Interval, others=OPC UA status *)
    SubscriptionHdl : DWORD;      (* Handle to created subscription *)
    PublishingInterval : TIME;     (* Actual publishing interval returned by server *)
END_VAR

VAR
    LastExecute : BOOL;           (* Tracks previous EXECUTE state for edge detection *)
    SubscriptionTimer : TON;      (* Timer for timeout *)
    C_Result : DWORD;             (* Result from C-code *)
    C_PublishingIntervalMs : DWORD; (* Publishing interval in milliseconds for C-code *)
    C_SubscriptionId : DWORD;     (* Subscription ID from C-code *)
END_VAR

(* Initialize outputs *)
DONE := FALSE;
BUSY := FALSE;
ERROR := FALSE;
ERROR_ID := 0;
SubscriptionHdl := 0;

(* Main logic *)
IF EXECUTE AND NOT LastExecute THEN
    (* Initialize on rising edge *)
    BUSY := TRUE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    SubscriptionHdl := 0;
    C_SubscriptionId := 0;
    SubscriptionTimer(IN := TRUE, PT := Timeout);
    
    (* Validate inputs *)
    IF ConnectionHdl = 0 THEN
        ERROR := TRUE;
        ERROR_ID := 1; (* Invalid Handle *)
        BUSY := FALSE;
        SubscriptionTimer(IN := FALSE);
        RETURN;
    END_IF
    IF PublishingInterval < T#10ms THEN
        ERROR := TRUE;
        ERROR_ID := 2; (* Invalid Interval *)
        BUSY := FALSE;
        SubscriptionTimer(IN := FALSE);
        RETURN;
    END_IF
    
    (* Convert publishing interval to milliseconds *)
    C_PublishingIntervalMs := TIME_TO_MS(PublishingInterval);
    
    (* Call C-code to create subscription *)
    C_Result := OPCUA_CreateSubscription(ConnectionHdl, Priority, ADR(C_PublishingIntervalMs), ADR(C_SubscriptionId));
    IF C_Result = 0 THEN
        (* Subscription created successfully *)
        DONE := TRUE;
        BUSY := FALSE;
        SubscriptionHdl := C_SubscriptionId;
        PublishingInterval := MS_TO_TIME(C_PublishingIntervalMs);
        SubscriptionTimer(IN := FALSE);
    ELSE
        (* Subscription creation failed *)
        ERROR := TRUE;
        ERROR_ID := C_Result;
        BUSY := FALSE;
        SubscriptionTimer(IN := FALSE);
    END_IF
END_IF

(* Handle timeout *)
IF BUSY AND SubscriptionTimer.Q THEN
    ERROR := TRUE;
    ERROR_ID := 3; (* Timeout *)
    BUSY := FALSE;
    SubscriptionTimer(IN := FALSE);
END_IF

(* Reset outputs when EXECUTE is FALSE *)
IF NOT EXECUTE THEN
    BUSY := FALSE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    SubscriptionHdl := 0;
    SubscriptionTimer(IN := FALSE);
END_IF

(* Update edge detection *)
LastExecute := EXECUTE;

(* External C function declaration *)
FUNCTION OPCUA_CreateSubscription : DWORD
VAR_INPUT
    Client : DWORD;               (* Pointer to UA_Client *)
    Priority : BYTE;              (* Subscription priority *)
    PublishingIntervalMs : DWORD; (* Pointer to publishing interval in ms *)
    SubscriptionId : DWORD;       (* Pointer to subscription ID *)
END_VAR
(* External C function, returns 0 on success or error code *)
END_FUNCTION

(* Helper functions for time conversion *)
FUNCTION TIME_TO_MS : DWORD
VAR_INPUT
    t : TIME;
END_VAR
TIME_TO_MS := t / T#1ms;
END_FUNCTION

FUNCTION MS_TO_TIME : TIME
VAR_INPUT
    ms : DWORD;
END_VAR
MS_TO_TIME := ms * T#1ms;
END_FUNCTION
END_FUNCTION_BLOCK
