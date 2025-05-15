FUNCTION_BLOCK FFT_Block
VAR_INPUT
    InputSignal : ARRAY[1..64] OF REAL; // Input time-domain signal (must be power of 2)
    SampleRate : REAL; // Sampling rate in Hz
END_VAR

VAR_OUTPUT
    Amplitude : ARRAY[1..32] OF REAL; // Output amplitude spectrum (first N/2 bins)
END_VAR

VAR
    RealPart : ARRAY[1..64] OF REAL; // Real part of complex numbers
    ImagPart : ARRAY[1..64] OF REAL; // Imaginary part of complex numbers
    TwiddleFactorReal : ARRAY[1..64] OF REAL; // Real part of twiddle factors
    TwiddleFactorImag : ARRAY[1..64] OF REAL; // Imaginary part of twiddle factors
    NumPoints : INT := 64; // Number of points in the FFT
    NumStages : INT := 6; // Log2(NumPoints)
    Stage : INT; // Current stage of the FFT
    GroupSize : INT; // Size of groups within each stage
    k : INT; // Index within each group
    m : INT; // Distance between butterflies
    t : INT; // Bit-reversed index
    ButterflyIndex : INT; // Index for butterfly operation
    TempReal : REAL; // Temporary storage for real part
    TempImag : REAL; // Temporary storage for imaginary part
    AngleStep : REAL; // Step size for angle calculation
    Angle : REAL; // Angle for twiddle factor calculation
    i, j : INT; // Loop indices
END_VAR

METHOD BitReverseIndex : INT
VAR_INPUT
    Value : INT;
    Bits : INT;
END_VAR
VAR
    Reversed : INT := 0;
    Bit : INT;
BEGIN
    FOR Bit := 0 TO Bits - 1 DO
        Reversed := Reversed * 2 + ((Value SHR Bit) AND 1);
    END_FOR;
    BitReverseIndex := Reversed;
END_METHOD

// Main execution logic
FOR i := 1 TO NumPoints DO
    RealPart[i] := InputSignal[i];
    ImagPart[i] := 0.0;
END_FOR;

// Bit-reversal permutation
FOR i := 1 TO NumPoints - 1 DO
    t := BitReverseIndex(i - 1, NumStages) + 1;
    IF i < t THEN
        TempReal := RealPart[i];
        RealPart[i] := RealPart[t];
        RealPart[t] := TempReal;
        
        TempImag := ImagPart[i];
        ImagPart[i] := ImagPart[t];
        ImagPart[t] := TempImag;
    END_IF;
END_FOR;

// Calculate twiddle factors
AngleStep := 2.0 * PI / NumPoints;
FOR i := 1 TO NumPoints DO
    Angle := (i - 1) * AngleStep;
    TwiddleFactorReal[i] := COS(Angle);
    TwiddleFactorImag[i] := SIN(-Angle);
END_FOR;

// Perform FFT stages
FOR Stage := 1 TO NumStages DO
    GroupSize := 2 ** (Stage - 1);
    m := 2 * GroupSize;
    FOR k := 1 TO GroupSize DO
        ButterflyIndex := (k - 1) * (NumPoints / m);
        FOR j := k TO NumPoints BY m DO
            TempReal := RealPart[j + GroupSize] * TwiddleFactorReal[ButterflyIndex + 1] 
                      - ImagPart[j + GroupSize] * TwiddleFactorImag[ButterflyIndex + 1];
            TempImag := RealPart[j + GroupSize] * TwiddleFactorImag[ButterflyIndex + 1] 
                      + ImagPart[j + GroupSize] * TwiddleFactorReal[ButterflyIndex + 1];
            
            RealPart[j + GroupSize] := RealPart[j] - TempReal;
            ImagPart[j + GroupSize] := ImagPart[j] - TempImag;
            
            RealPart[j] := RealPart[j] + TempReal;
            ImagPart[j] := ImagPart[j] + TempImag;
        END_FOR;
    END_FOR;
END_FOR;

// Calculate amplitude spectrum
FOR i := 1 TO NumPoints / 2 DO
    Amplitude[i] := SQRT((RealPart[i] * RealPart[i]) + (ImagPart[i] * ImagPart[i]));
END_FOR;

// Scale the amplitude by the number of points
FOR i := 1 TO NumPoints / 2 DO
    Amplitude[i] := Amplitude[i] / NumPoints;
END_FOR;



