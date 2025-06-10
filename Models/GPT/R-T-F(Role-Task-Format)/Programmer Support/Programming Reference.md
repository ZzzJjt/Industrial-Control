IF MotorRunning THEN
    Speed := 100;
ELSE
    Speed := 0;
END_IF;

CASE Mode OF
    0: Status := 'Idle';
    1: Status := 'Run';
    2: Status := 'Error';
ELSE
    Status := 'Unknown';
END_CASE;

FOR i := 1 TO 10 DO
    Sum := Sum + i;
END_FOR;

WHILE Temp < 100.0 DO
    Temp := Temp + 1.0;
END_WHILE;

Timer1(IN := StartButton, PT := T#10s);
IF Timer1.Q THEN
    Output := TRUE;
END_IF;

TYPE TankStatus :
STRUCT
    Level   : REAL;
    Temp    : REAL;
    Alarm   : BOOL;
END_STRUCT
END_TYPE;
