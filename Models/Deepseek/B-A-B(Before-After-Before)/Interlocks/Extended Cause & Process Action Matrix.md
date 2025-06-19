# Define potential causes and corresponding safety actions
causes = ["High Pressure", "Low Pressure", "High Temperature", "Low Temperature", "Level Sensor Fault", "Flow Anomaly"]
actions = ["Close Feed Valve", "Vent Reactor", "Activate Emergency Cooling", "Stop Process"]

# Create an interlock matrix mapping each cause to one or more actions
interlock_matrix = {
    "High Pressure": ["Close Feed Valve", "Vent Reactor"],
    "Low Pressure": ["Stop Process"],
    "High Temperature": ["Activate Emergency Cooling"],
    "Low Temperature": ["Stop Process"],
    "Level Sensor Fault": ["Stop Process", "Vent Reactor"],
    "Flow Anomaly": ["Close Feed Valve"]
}

def validate_coverage(matrix, all_actions):
    """
    Validate that each safety action is triggered by at least one cause.
    Identifies any action that is not covered by the matrix.
    """
    covered_actions = set()
    for action_list in matrix.values():
        for action in action_list:
            covered_actions.add(action)
    
    missing_actions = set(all_actions) - covered_actions
    if missing_actions:
        print(f"⚠️ Missing coverage for the following actions: {missing_actions}")
    else:
        print("✅ All safety actions are covered.")

def main():
    # Display the full interlock matrix
    print("Interlock Matrix:")
    for cause, action in interlock_matrix.items():
        print(f"{cause}: {action}")
    
    # Validate coverage of all defined safety actions
    validate_coverage(interlock_matrix, actions)

if __name__ == "__main__":
    main()
