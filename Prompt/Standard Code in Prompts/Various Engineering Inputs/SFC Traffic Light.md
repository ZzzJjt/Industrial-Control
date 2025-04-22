```
+------------------------------------+
|  Traffic Light Control SFC         |
+------------------------------------+
|                                     |
| +---------+       +---------+      |
| | Step 1 |-------| Step 2  |      |
| | RED ON |       | YELLOW  |      |
| +--|-----+       +--|-----+      |
|    ^  60s       ^  3s           |
|    |            |               |
|    +----+------+                |
|         | RED/YELLOW OFF       |
|         +---------------------+ |
|                                     |
| +---------+       +---------+      |
| | Step 3 |-------| Step 4  |      |
| | GREEN  |       | YELLOW  |      |
| +--|-----+       +--|-----+      |
|    ^  60s       ^  3s           |
|    |            |               |
|    +----+------+                |
|         | RED/YELLOW OFF       |
|         +---------------------+ |
|                                     |
| +---------+       +---------+      |
| | Step 5 |-------| Step 1  |      |
| | RED ON |       | RED ON  |      |
| +---------+       +---------+      |
|                                     |
+------------------------------------+

Legend:
+---------+  : Step
| Step #  |  : State description
| Action  |  : What happens in the step
+--|-----+  : Transition point
^  #s    : Delay time in seconds before transition
|         : Transition condition (in this case, the timer elapsing)
+----+----+: Conditional branch (not used here but shown for completeness)

Step Descriptions:
Step 1 (RED ON): The red light is on, indicating stop for all traffic.
Step 2 (YELLOW): The yellow light turns on to warn drivers that the green light is about to turn red.
Step 3 (GREEN): The green light turns on, allowing traffic to proceed.
Step 4 (YELLOW): The yellow light turns on again to warn drivers that the green light is about to turn red.

Transitions:
- From Step 1 to Step 2: After 60 seconds (delay for red light), the yellow light comes on.
- From Step 2 to Step 3: After 3 seconds (delay for yellow light), the green light comes on.
- From Step 3 to Step 4: After 60 seconds (delay for green light), the yellow light comes on.
- From Step 4 to Step 1: After 3 seconds (delay for yellow light), the red light comes on again.

This SFC ensures that the traffic light cycles through its states in a predetermined sequence, with appropriate delays between state changes.
```
