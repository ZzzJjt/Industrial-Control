FUNCTION_BLOCK FB_ModbusTCPServer
VAR_INPUT
    Enable          : BOOL;
    Request         : ARRAY[1..10] OF BOOL; (* Triggers from 10 clients *)
    FunctionCode    : ARRAY[1..10] OF BYTE;
    Address         : ARRAY[1..10] OF WORD;
    Quantity        : ARRAY[1..10] OF WORD;
    WriteData       : ARRAY[1..10, 0..255] OF BYTE; (* For write ops *)
END_VAR

VAR_OUTPUT
    ResponseData    : ARRAY[1..10, 0..255] OF BYTE;
    Status          : ARRAY[1..10] OF INT; (* 0 = OK, <0 = Error *)
END_VAR

VAR
    CoilData        : ARRAY[0..1023] OF BOOL;
    HoldingReg      : ARRAY[0..255] OF WORD;
    InputReg        : ARRAY[0..255] OF WORD;
    DiscreteInput   : ARRAY[0..1023] OF BOOL;

    i               : INT;
    byteIndex       : INT;
    bitPos          : INT;
    tempByte        : BYTE;
END_VAR

(* Core Logic *)
FOR i := 1 TO 10 DO
    IF Enable AND Request[i] THEN
        CASE FunctionCode[i] OF

            16#01: // Read Coils
                IF Quantity[i] > 200 THEN
                    Status[i] := -1; // Quantity too large
                ELSE
                    byteIndex := 0;
                    bitPos := 0;
                    tempByte := 0;

                    FOR j := 0 TO Quantity[i] - 1 DO
                        IF CoilData[Address[i] + j] THEN
                            tempByte := tempByte OR SHL(1, bitPos);
                        END_IF
                        bitPos := bitPos + 1;
                        IF bitPos = 8 THEN
                            ResponseData[i, byteIndex] := tempByte;
                            byteIndex := byteIndex + 1;
                            bitPos := 0;
                            tempByte := 0;
                        END_IF
                    END_FOR

                    IF bitPos <> 0 THEN
                        ResponseData[i, byteIndex] := tempByte;
                    END_IF

                    Status[i] := 0;
                END_IF

            16#03: // Read Holding Registers
                FOR j := 0 TO Quantity[i] - 1 DO
                    ResponseData[i, j * 2] := WORD_TO_BYTE(SHR(HoldingReg[Address[i] + j], 8));
                    ResponseData[i, j * 2 + 1] := WORD_TO_BYTE(HoldingReg[Address[i] + j]);
                END_FOR
                Status[i] := 0;

            16#05: // Write Single Coil
                CoilData[Address[i]] := WriteData[i, 0] = 16#FF;
                Status[i] := 0;

            16#06: // Write Single Register
                HoldingReg[Address[i]] := SHL(WORD(WriteData[i,0]), 8) OR WORD(WriteData[i,1]);
                Status[i] := 0;

            // Add similar logic for 0x02, 0x04, 0x0F, 0x10, 0x17 as needed

        ELSE
            Status[i] := -99; // Unsupported Function
        END_CASE
    END_IF
END_FOR
