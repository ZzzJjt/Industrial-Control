VAR
    x: INT := 5;
    y: INT := 10;
    result: BOOL;
END_VAR

IF x < y THEN
    result := TRUE;
ELSE
    result := FALSE;
END_IF;

VAR
    t1 : TON;
    start : BOOL;
    elapsed : BOOL;
END_VAR

t1(IN := start, PT := T#10s);
elapsed := t1.Q;
