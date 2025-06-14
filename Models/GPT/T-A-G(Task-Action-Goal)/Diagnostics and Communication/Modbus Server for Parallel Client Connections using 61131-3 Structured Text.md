FUNCTION_BLOCK FB_ModbusTCP_Server
VAR_INPUT
    Enable       : BOOL;
    ClientData   : ARRAY[1..10] OF ModbusRequest; // Custom request type per client
END_VAR

VAR_OUTPUT
    ResponseData : ARRAY[1..10] OF ModbusResponse;
    Status       : ARRAY[1..10] OF INT;
END_VAR

VAR
    CoilBuffer         : ARRAY[0..511] OF BOOL;
    HoldingRegisters   : ARRAY[0..255] OF WORD;
    InputRegisters     : ARRAY[0..255] OF WORD;
    DiscreteInputs     : ARRAY[0..255] OF BOOL;
    i, j               : INT;
    byteVal            : BYTE;
    tempWord           : WORD;
    fc                 : BYTE;
    addr, quantity     : INT;
END_VAR

FOR i := 1 TO 10 DO
    IF Enable AND ClientData[i].Connected THEN
        fc := ClientData[i].FunctionCode;
        addr := ClientData[i].Address;
        quantity := ClientData[i].Quantity;

        CASE fc OF

            1: // Read Coils
                ResponseData[i].ByteCount := (quantity + 7) / 8;
                FOR j := 0 TO quantity - 1 DO
                    IF CoilBuffer[addr + j] THEN
                        ResponseData[i].Data[j / 8] := ResponseData[i].Data[j / 8] OR SHL(1, j MOD 8);
                    END_IF
                END_FOR
                Status[i] := 0;

            2: // Read Discrete Inputs
                ResponseData[i].ByteCount := (quantity + 7) / 8;
                FOR j := 0 TO quantity - 1 DO
                    IF DiscreteInputs[addr + j] THEN
                        ResponseData[i].Data[j / 8] := ResponseData[i].Data[j / 8] OR SHL(1, j MOD 8);
                    END_IF
                END_FOR
                Status[i] := 0;

            3: // Read Holding Registers
                FOR j := 0 TO quantity - 1 DO
                    ResponseData[i].Data[j] := HoldingRegisters[addr + j];
                END_FOR
                Status[i] := 0;

            4: // Read Input Registers
                FOR j := 0 TO quantity - 1 DO
                    ResponseData[i].Data[j] := InputRegisters[addr + j];
                END_FOR
                Status[i] := 0;

            5: // Write Single Coil
                CoilBuffer[addr] := ClientData[i].WriteValue = 16#FF00;
                Status[i] := 0;

            6: // Write Single Register
                HoldingRegisters[addr] := ClientData[i].WriteValue;
                Status[i] := 0;

            15: // Write Multiple Coils
                FOR j := 0 TO quantity - 1 DO
                    byteVal := ClientData[i].Data[j / 8];
                    CoilBuffer[addr + j] := (byteVal AND SHL(1, j MOD 8)) <> 0;
                END_FOR
                Status[i] := 0;

            16: // Write Multiple Registers
                FOR j := 0 TO quantity - 1 DO
                    HoldingRegisters[addr + j] := ClientData[i].Data[j];
                END_FOR
                Status[i] := 0;

            23: // Read/Write Multiple Registers
                // First write phase
                FOR j := 0 TO ClientData[i].WriteQuantity - 1 DO
                    HoldingRegisters[ClientData[i].WriteAddress + j] := ClientData[i].WriteData[j];
                END_FOR
                // Then read response
                FOR j := 0 TO quantity - 1 DO
                    ResponseData[i].Data[j] := HoldingRegisters[addr + j];
                END_FOR
                Status[i] := 0;

            ELSE
                Status[i] := -1; // Unsupported function code
        END_CASE

    ELSE
        Status[i] := -2; // Client not connected or server not enabled
    END_IF
END_FOR
