         +-----------------+                  
         |   Analog Input  |                  
         |   IN: RawSignal |                  
         |  OUT: AI_Value  |                  
         +--------+--------+                  
                  |                           
                  |                           
         +--------v--------+     +---------------------+     
         |      TON        |     |        PID          |     
         | IN : AI_Value   |     | PV : AI_Value       |     
         | PT : T#5s       |     | SP : SetPoint       |     
         | Q  : TimerDone  |     | CV : PID_Output     |     
         | ET : Elapsed    |     +----------+----------+     
         +-----------------+                |                
                                           |                
                                           |                
                                 +---------v----------+     
                                 |   Analog Output     |     
                                 | IN : PID_Output     |     
                                 +---------------------+     
