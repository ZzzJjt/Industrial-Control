                          Function Block Diagram (ASCII View)

  +-----------------+       
  |  AnalogInput    |       
  |  OUT = AI       |-------+-------------------------+     
  +-----------------+       |                         |     
                            |                         |     
                        +---v---+               +-----v-----+     
                        |  TON  |               |   PID     |     
                        | IN=AI |               | PV = AI   |     
                        | PT=PT |               | SP = SP   |     
                        | Q = Q |               | CV = CV   |     
                        | ET=ET |               +-----------+     
                        +-------+                     |           
                                                      |           
                                               +------v------+     
                                               | AnalogOutput |     
                                               |   IN = CV    |     
                                               +-------------+     
