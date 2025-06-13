FUNCTION_BLOCK FB_RealComparator
VAR_INPUT
    Input1 : REAL; // First input value
    Input2 : REAL; // Second input value
    Precision : INT; // Number of decimal places for comparison
    Enable : BOOL; // Flag to enable the comparator
END_VAR

VAR_OUTPUT
    Equal : BOOL := FALSE; // Output flag indicating if inputs are equal
    Greater : BOOL := FALSE; // Output flag indicating if Input1 is greater than Input2
    Less : BOOL := FALSE; // Output flag indicating if Input1 is less than Input2
    Error : BOOL := FALSE; // Output flag indicating an error (invalid precision or disabled execution)
END_VAR

VAR
    Scale : REAL; // Scaling factor based on precision
    ScaledInput1 : INT; // Scaled and rounded Input1
    ScaledInput2 : INT; // Scaled and rounded Input2
END_VAR

// Main method to execute the comparison logic
METHOD Execute(this : REFERENCE TO FB_RealComparator) : BOOL
BEGIN
    // Reset output flags
    this.Equal := FALSE;
    this.Greater := FALSE;
    this.Less := FALSE;
    this.Error := FALSE;

    // Check if the comparator is enabled
    IF NOT this.Enable THEN
        this.Error := TRUE;
        RETURN TRUE;
    END_IF;

    // Validate precision
    IF this.Precision < 0 THEN
        this.Error := TRUE;
        RETURN TRUE;
    END_IF;

    // Calculate scaling factor
    Scale := POWER(10, this.Precision);

    // Scale and round the inputs
    ScaledInput1 := ROUND(this.Input1 * Scale);
    ScaledInput2 := ROUND(this.Input2 * Scale);

    // Compare the scaled and rounded inputs
    IF ScaledInput1 = ScaledInput2 THEN
        this.Equal := TRUE;
    ELSIF ScaledInput1 > ScaledInput2 THEN
        this.Greater := TRUE;
    ELSE
        this.Less := TRUE;
    END_IF;

    RETURN TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_RealComparator) : BOOL
BEGIN
    RETURN this.Execute();
END_METHOD

PROGRAM MainProgram
VAR
    ComparatorFB : FB_RealComparator;
    Input1 : REAL := 123.456;
    Input2 : REAL := 123.457;
    Precision : INT := 3;
    Enable : BOOL := TRUE;
    Equal : BOOL;
    Greater : BOOL;
    Less : BOOL;
    Error : BOOL;
END_VAR

// Set the inputs for the comparator
ComparatorFB.Input1 := Input1;
ComparatorFB.Input2 := Input2;
ComparatorFB.Precision := Precision;
ComparatorFB.Enable := Enable;

// Call the comparator function block
ComparatorFB();

// Get the results
Equal := ComparatorFB.Equal;
Greater := ComparatorFB.Greater;
Less := ComparatorFB.Less;
Error := ComparatorFB.Error;

// Output the results
// For demonstration purposes, assume there's a way to display or log the results
// Display(Equal); // Hypothetical display function
// Display(Greater); // Hypothetical display function
// Display(Less); // Hypothetical display function
// Display(Error); // Hypothetical display function
