# JVM Components
+-----------------+     +-----------------+               +-----------------+     +-----------------+
|   Classloader   | --> | Bytecode Verifier| -->          |   Memory Area   | --> | Execution Engine |
+-----------------+     +-----------------+               +-----------------+     +-----------------+
 loads .class files     crucial for class loading          |      Heap           |     Responsible for actual execution of code
into JVM memory         linking process. performs          |     Method Area     |       +----------------------------------+ 
                        checkes like compatible datatypes  |      Stack          |       | Interpretor (Executes bytecode   |
                        flow control, operand stack checks,|   PC Registers      |       | instructions one by one) - Slower|
                         potential problems that could     |  Native Method Stack|       +----------------------------------+
                         cause java security               | +-----------------+ |
                                                                         
                                                                         
+-----------+
|  Garbage  |
| Collector |
+-----------+
