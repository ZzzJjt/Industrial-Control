```
  +--------------------------------------+
  | Analog Input (AI)                    |
  | Input Signal: 0.0 to 10.0 V          |
  +------------+-------------------------+
               |                         |
               v                         v
  +------------+     +-------------------+
  | Timer      |     | PID               |
  | Input: AI  |---->| Input: AI         |
  | Timer Set: 5 s  | | P: 0.5           |
  | Timer Out: TO   | | I: 0.2           |
  +------------+     | | D: 0.1           |
                     | | Output: PID_OUT |
                     +-------------------+
                                |
                                v
  +-----------------------------+
  | Analog Output (AO)          |
  | Input Signal: PID_OUT       |
  | Output Signal: 4.0 to 20.0 mA|
  +-----------------------------+
```
