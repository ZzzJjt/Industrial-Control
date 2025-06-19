# Define the list of potential causes (sensor triggers)
causes = [
    "High Pressure",
    "Low Pressure",
    "High Temperature",
    "Low Temperature",
    "High Liquid Level",
    "Low Liquid Level",
    "Flow Sensor Failure",
    "High Vibration"
]

# Define the list of possible effects (safety actions)
effects = [
    "Close Feed Valve",
    "Open Vent Valve",
    "Stop Recirculation Pump",
    "Activate Emergency Cooling",
    "Trigger High-Priority Alarm",
    "Shutdown Reactor"
]

# Build the cause-effect interlock matrix
interlock_matrix = {
    "High Pressure": ["Open Vent Valve", "Close Feed Valve", "Trigger High-Priority Alarm"],
    "Low Pressure": ["Shutdown Reactor", "Trigger High-Priority Alarm"],
    "High Temperature": ["Activate Emergency Cooling", "Close Feed Valve", "Trigger High-Priority Alarm"],
    "Low Temperature": ["Trigger High-Priority Alarm"],
    "High Liquid Level": ["Close Feed Valve", "Trigger High-Priority Alarm"],
    "Low Liquid Level": ["Stop Recirculation Pump", "Trigger High-Priority Alarm"],
    "Flow Sensor Failure": ["Shutdown Reactor", "Trigger High-Priority Alarm"],
    "High Vibration": ["Shutdown Reactor", "Trigger High-Priority Alarm"]
}

# Safety rationale dictionary: explains why each effect is important
safety_rationale = {
    "Close Feed Valve": "Prevents further material input during unsafe conditions to avoid overpressure or overflow.",
    "Open Vent Valve": "Relieves pressure buildup to prevent rupture or explosion.",
    "Stop Recirculation Pump": "Halts fluid movement that may be contributing to unstable conditions.",
    "Activate Emergency Cooling": "Reduces temperature to prevent thermal runaway or damage.",
    "Trigger High-Priority Alarm": "Notifies operators of abnormal conditions requiring immediate attention.",
    "Shutdown Reactor": "Stops the entire process to prevent escalation of critical faults."
}

def validate_coverage(matrix, effect_list):
    """
    Checks if every defined effect is triggered by at least one cause.
    Reports any missing safety actions.
    """
    covered_effects = set()
    for effect_list in matrix.values():
        for effect in effect_list:
            covered_effects.add(effect)

    missing_effects = set(effect_list) - covered_effects
    if missing_effects:
        print(f"⚠️ Warning: The following effects are not mapped to any cause: {missing_effects}")
    else:
        print("✅ All safety actions are covered.")

def explain_safety_actions(rationale_dict):
    """Prints explanations for each safety action."""
    print("\nSafety Action Rationale:")
    for action, reason in rationale_dict.items():
        print(f"- {action}: {reason}")

def main():
    # Display the full interlock matrix
    print("Cause and Effect Interlock Matrix:")
    for cause, effect in interlock_matrix.items():
        print(f"{cause} → {effect}")
    
    # Validate coverage of safety actions
    validate_coverage(interlock_matrix, effects)
    
    # Explain the rationale behind each safety action
    explain_safety_actions(safety_rationale)

if __name__ == "__main__":
    main()
