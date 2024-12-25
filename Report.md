# Advanced Computer Architecture Project

## Advanced Computer Architecture Project - Part 1

1. Based on the contents of the `starter_se.py` file, assumptions regarding the characteristics that have been passed to gem5 after running the `HelloWorld` program can be made. More specifically:
   * **CPU Type = "atomic"** (the type of the cpu is defined in line 192:
      ```python
      parser.add_argument("--cpu", type=str, choices=list(cpu_types.keys()),
                        default="atomic"
      ```  
      The final value, though, is determined by the argument in the terminal command `--cpu="minor". This means that the cache classes for the "minor" type are the ones defined in starter_se.py.
   * **Operational Frequency = 1GHz** :
      ```python
      parser.add_argument("--cpu-freq", type=str, default="1GHz")
      ```  
   * **Number of Cores = 1**:
      ```python
        parser.add_argument("--num-cores", type=int, default=1,
                        help="Number of CPU cores")
       ```
   * **Memory Type = DDR3_1600_8x8**:
      ```python
      parser.add_argument("--mem-type", default="DDR3_1600_8x8",
                        choices=ObjectList.mem_list.get_names(),
                          help = "type of memory to use")
        ```  
   * **Number of Memory Channels = 2**:
       ```python
      parser.add_argument("--mem-channels", type=int, default=2,
                          help = "number of memory channels")
        ```  
   * **Number of Memory Ranks per Channel = 0**:
       ```python
      parser.add_argument("--mem-ranks", type=int, default=None,
                        help = "number of memory ranks per channel")
        ```  
   * **Memory Size = 2GB**:
       ```python
     parser.add_argument("--mem-size", action="store", type=str,
                        default="2GB",
                        help="Specify the physical memory size")
        ```  
2. The following is information derived from the `stats.txt`, `config.ini`, and `config.json` files after running this command:

        
           $ ./build/ARM/gem5.opt -d hello_result configs/example/arm/starter_se.py --cpu="minor" "tests/test-progs/hello/bin/arm/linux/hello"
        

   a. 
      * Based on the `config.ini` file the CPU-type is "minor"
          ```
             [system.cpu_cluster.cpus]
             type=MinorCPU
          ```
         Although as mentioned in part 1. the default value, defined in `starter_se.py`, for the CPU-type is "atomic", the final value is determined by the argument in the above command `--cpu="minor"`.
      * from `stats.txt` the operational frequency is 1GHZ as expected
         ```
          sim_freq                                 1000000000000                       # Frequency of simulated ticks
         ```
      * from `config.ini` we can see that there are 2 memory channels:
         ```
         memories=system.mem_ctrls0 system.mem_ctrls1
         ```
         The type for both channels is:
          ```
          type=DRAMCtrl
         ```
      
      * from `config.ini` the number of memory ranks per channel is 2 for both memory channels
         ```
          [system.mem_ctrls0]/[system.mem_ctrls1]
          ranks_per_channel=2
         ```

   b. 
      * sim_seconds: the time in seconds that the simulation needs for execution
      * sim_insts: the number of instructions that were simulated
      * host_inst_rate: the rate of instructions simulated per second

   c. From `stats.txt` the number of committed instructions is 5027 and of committed operations is 5831.
  
         
         system.cpu_cluster.cpus.committedInsts           5027                       # Number of instructions committed
  
         system.cpu_cluster.cpus.committedOps             5831                       # Number of ops (including micro ops) committed
  

     The reason that they are not equal is that "committedInsts" is the architectural number of assembly instructions executed while "commmittedOps" is the number of micro-operations. Each instruction can expand to multiple microoperations, so this number is always greater or equal to committedInsts.
  
      The number of instructions simulated is again 5027:
      
          sim_insts                                        5027                       # Number of instructions simulated
  
      The fact that the simulated instructions and committed instruction are the same signifies that all instructions are committed. An instruction commits only if it and all instructions before it have been completed successfully.
       

   d.  from `starter_se.py` and because the type of the CPU is "minor" the L2 Cache class is "devices.L2".
       From `stats.txt` the L2 accesses are the following:

       
       system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.inst          327                       # number of overall (read+write) accesses

       system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.data          147                       # number of overall (read+write) accesses

       system.cpu_cluster.l2.overall_accesses::total          474                       # number of overall (read+write) accesses
        

      The number of times that the L2 cache was accessed is equal to the number of times L1 was accessed and a miss occurs which is 147 (as far as data is concerned):

       
        system.cpu_cluster.cpus.branchPred.indirectMisses          147                       # Number of indirect misses.
        

4. Different in-order CPU models on gem5:
   
     i. SimpleCPU:  
      * BaseSimpleCPU: 

    > This version of SimpleCPU is a single-cycle CPU which means it can execute only one instruction in a cycle. It can be inherited by AtomicSimpleCPU and TimingSimpleCPU. It can not be run on its own. One of the inheriting classes, either AtomicSimpleCPU or TimingSimpleCPU must be used. BaseSimpleCPU defines functions, common for SimpleCPU models, for checking for interrupts, controlling setups and actions, and advancing the PC into the next instruction, holding the architected state. It is also responsible for the ExecContext interface which is used by the instructions to access the CPU.
 
      * AtomicSimpleCPU:
     
     >In this version of SimpleCPU atomic memory accesses are used. It derives from BaseSimpleCPU and uses functions to read and write memory, as well as to tick,  meaning what happens in every CPU cycle. Also, it determines the overall cache access time using the latency estimates from the atomic accesses. Lastly, the AtomicSimpleCPU defines which port is used to connect to memory, and attaches the CPU to the cache.  
       
      * TimingSimpleCPU:

    >This version of SimpleCPU uses timing memory accesses. Timing accesses are the most detailed access but they lack in speed (delay). With the sending of a request function at some time, a response function or a number of functions are scheduled at some time in the future to be executed. It derives from BaseSimpleCPU and and implements the same set of functions as the aforementioned model, AtomicSimpleCPU. It is responsible for handling the response from memory to the accesses sent out.  If the response is a NACK the procedure gets repeated. It, finally, defines the port that is used to hook up to memory and connects the CPU to the cache.
      
    ii. Minor CPU:
    >This in-order processor model has a standard pipeline, but its data structures and execute behavior are configurable. It provides a framework to match the model with a similar processor on a micro-architectural level which has strict in-order execution behavior and visualizes the position of an instruction in the pipeline using the MinorTrace/minorview.py format/too. 

   a. Commands for the `fibonacci.c` file:

      * MinorCPU
         ``` 
            $ ./build/ARM/gem5.opt -d fib_results_MinorCPU configs/example/se.py --cpu-type=MinorCPU --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
      * TimingSimpleCPU
        ```
           $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPU configs/example/se.py --cpu-type=TimingSimpleCPU --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
         ```

      Execution times:
      * MinorCPU
        ```
        sim_seconds                                  0.000037                       # Number of seconds simulated
        ```
      * TimingSimpleCPU
         ```
        sim_seconds                                  0.000046                       # Number of seconds simulated
        ```

   b. Comparing the simulations run with MinorCPU and TimingSimpleCPU we noticed the following:
     * Differences:  
       * host_seconds: 0.02 for TimingSimpleCPU and 0.09 for MinorCPU  
       * sim_seconds: 0.000046 for TimingSimpleCPU and 0,000037 for MinorCPU  
       * Number of indirect misses: 137 for MinorCPU and it is not mentioned for TimingSimpleCPU  
     * Similarities:
       * sim_freq: 1000000000000 for both CPUs
       * host_mem_usage: 674144 for TimingSimpleCPU and 678756 for MinorCPU
       * Number of instructions committed: 13196 for TimingSimpleCPU and 13249 for MinorCPU  
    
    The differences are mostly related to time. They are justified considering the fact that TimingSimpleCPU lacks speed in comparison to MinorCPU.  
    The similarities are also justified because they concern a) the frequency which is determined by us, b) the memory used by the host, and the number of instructions committed which depend on our program and are not influenced by the CPU.

   c. Changed parameters and comparison of the 2 CPU models:
   
      * Different memory type:
        * For the TimingSimpleCPU we used the "LPDDR2_S4_1066_1x32" memory type:
          ```
          $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPUDDR4_2400_8x8 configs/example/se.py --cpu-type=MinorCPU --mem-type=DDR4_2400_8x8 --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
          * There is a small decrement in the time of execution while using the "LPDDR2_S4_1066_1x32" memory type. Number of seconds simulated before = 0.000046 and after = 0.000036. 
          * Also a decrement in the instruction rate is noticed: Simulator instruction rate before=573016  and after=157590. 
        * For the MinorCPU we used the "SimpleMemory" memory type:
          ```
           $ ./build/ARM/gem5.opt -d fib_results_MinorCPUNonCachingSimpleCPU configs/example/se.py --cpu-type=NonCachingSimpleCPU --mem-type=SimpleMemory --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
           * The instruction rate has decreased with the use of the "NonCachingSimpleCPU" type. Specifically, simulator instruction rate before=147368 and after=1369757.
           * The simulated time seems to be decreased. Number of seconds simulated before = 0.000037 and after = 0.000009.

      * Different Operational Frequency:
         * For the TimingSimpleCPU we set the frequency to 4GHz:
            ```
            $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPU4GHz configs/example/se.py --cpu-type=TimingSimpleCPU --cpu-clock=4GHz --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
             ```
         * For the MinorCPU we set the frequency to 4GHz:
            ```
            $ ../build/ARM/gem5.opt -d fib_results_MinorCPU4GHz configs/example/se.py --cpu-type=MinorCPU --cpu-clock=4GHz --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
             ```
          The operational frequency refers to the processor's operational clock cycles per second. This means that by increasing the frequency the execution must be faster. We can confirm that by the results on the `stats.txt` file. More specifically we observed subduplication of the total simulated time in both CPU models with the quadruplication of the operational frequency.

        

## Advanced Computer Architecture Project - Part 2

### STEP 1 ###
 
1. The basic parameters derived from `config.ini`:
     - L1 instruction cache:
       ```
       [system.cpu.icache]
        type=Cache
        assoc=2
        size=32768
       ```
     - L1 data cache
       ```
       [system.cpu.dcache]
        type=Cache
        assoc=2
        size=65536
       ```
     - L2 cache
       ```
        [system.l2]
        type=Cache
        assoc=8
        size=2097152
       ```
     - Cache Line
       ```
        [system]
        type=System
        cache_line_size=64
       ```
2. We are deriving the results from the `stats.txt` files:
   | Parameters | specbzip | specmcf | spechmmer | sjeng | speclibm |
   | ---------- | -------- | ------- | ----- | ------- | ------ |
   | Execution time (`sim_sec`) | 0.083982 | 0.064955 | 0.059396 | 0.513528 | 0.174641 | 
   | CPI | 1.679650 | 1.299095 | 1.187917 | 10.270554 | 3.493415 | 
   | L1 Instruction cache miss rates | 0.000077 | 0.023612 | 0.000221 | 0.000020 | 0.000094 |
   | L1 Data cache miss rates | 0.014798 | 0.002109 | 0.001637 | 0.121831 | 0.060972 |
   | L2 cache miss rates | 0.282163 | 0.055046 | 0.077760 | 0.999972 | 0.999944 |

   
   ![execution_time](https://github.com/user-attachments/assets/58911035-7e6e-46b1-be0e-cb1488eddd58)

   ![cpi](https://github.com/user-attachments/assets/efb0d3c9-f5d1-4a1b-8552-8564d158a73e)

   ![icache](https://github.com/user-attachments/assets/6a2bd62e-9f50-4899-b155-4573528f11e5)

   ![dcache](https://github.com/user-attachments/assets/ce067f71-b390-4b71-822f-f8916712524e)

   ![l2cache](https://github.com/user-attachments/assets/34579404-e692-484d-b5d3-c0bf77a7a266)

   From the graphs, we observe that the Execution Time and CPI are higher for the `sjeng` and `speclibm` benchmarks. This occurs because these two benchmarks are the only ones with high miss rates in both the L1 data cache and L2 cache. When the processor cannot find the requested data in the L1 or L2 cache (resulting in a cache miss), it must fetch the data from the main memory, which takes significantly longer. This delay increases the execution time. The time wasted waiting for data to be fetched from main memory also increases the number of cycles needed to execute instructions, leading to a higher CPI.

3. For all benchmarks the values were equal with the changes in the frequency:
     * Default frequency
         * System cpu clock
          ```python 
          system.cpu_clk_domain.clock                       500                       # Clock period in ticks
          ```
         * System clock
          ```python 
          system.clk_domain.clock                          1000                       # Clock period in ticks
          ```
      * 1GHz
         * System cpu clock
          ```python 
          system.cpu_clk_domain.clock                       1000                       # Clock period in ticks
          ```
         * System clock
          ```python 
          system.clk_domain.clock                          1000                       # Clock period in ticks
          ```
      * 3GHz
         * System cpu clock
          ```python 
          system.cpu_clk_domain.clock                       333                       # Clock period in ticks
          ```
         * System clock
          ```python 
          system.clk_domain.clock                          1000                       # Clock period in ticks
          ```

    We can deduce the following for the default frequency:
    
    - Default Frequency
      - CPU Clock (500 ticks): The processor is clocked at 1/500 = 2 GHz.
      - System Clock (1000 ticks): The rest of the system is clocked at 1/1000 ticks = 1 GHz.
    - 1GHz Configuration
      - CPU Clock (1000 ticks): The processor is clocked at 1 / (1000 ticks) = 1 GHz.
      - System Clock (1000 ticks): The rest of the system remains clocked at 1 GHz.
    - 3GHz Configuration
      - CPU Clock (333 ticks): The processor is clocked at 1 / (333 ticks)= 3 GHz.
      - System Clock (1000 ticks): The rest of the system remains at 1 GHz.
        
    We can also deduce that when the frequency is changed, only the system CPU clock is affected, while the system clock remains at its default value. This behavior can be explained by considering that the system clock synchronizes all the computer's components, while the CPU clock is dedicated solely to the processor.
    Adding another processor introduces a new CPU clock specific to that processor. This means the new CPU operates independently with its own clock frequency, which can be adjusted separately from the existing processor.
    Based on the execution times, we can observe that as the frequency increases, the execution time decreases. However, the decrease in execution time is not proportional to the changes in the CPU clock. There is no perfect scaling with respect to execution time because the relationship between the CPU clock frequency and execution time is not one-to-one. Execution time cannot decrease easily by modifying just one system parameter; it is also influenced by many other factors.

4. Changing the memory type from `DDR3_1600_x64` to `DDR3_2133_8x8` to the specmcf benchmark there seems to be a slight decrease in the simulated time:
   ``` 
      sim_seconds                                  0.064955                       # Number of seconds simulated
      sim_seconds                                  0.064892                       # Number of seconds simulated
    ```
   which is expected since the second memory type has a quicker clock. We, also observe that the cycles simulated are also decreased:
   ```
       system.cpu.numCycles                        129909477                       # number of cpu cycles simulated
       system.cpu.numCycles                        129784377                       # number of cpu cycles simulated
   ```
   

### STEP 2 ###

![specbzip](https://github.com/user-attachments/assets/1c728406-7d95-4831-8395-a66883b96c06)

![specmcf](https://github.com/user-attachments/assets/c63bd677-897d-428b-bc53-0db1d795a448)

![spechmmer](https://github.com/user-attachments/assets/ba942c1e-6f9c-4380-838d-9dde1c4e0e0f)

![specsjeng](https://github.com/user-attachments/assets/ddbec12d-b1e7-4212-9216-42d7b7b37d1a)

![speclibm](https://github.com/user-attachments/assets/1e2d67d4-4a6a-4099-8ab1-86f168e9c0e0)


Based on the results shown on the graphs and general information and knowledge regarding the CPI we can extract the following conclusions:
- Cache Size: Larger L1 and L2 caches help benchmarks with larger working sets by reducing memory misses.
- Associativity: Benchmarks with higher conflict misses benefit from increased associativity.
- Cache Line Size: Benchmarks with sequential memory access patterns benefit greatly from increased cache line size.

### STEP 3 ###


## Βibliography
- https://stackoverflow.com/questions/58554232/what-is-the-difference-between-the-gem5-cpu-models-and-which-one-is-more-accurat
- https://www.gem5.org/documentation/general_docs/cpu_models/SimpleCPU
- https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu
- https://nitish2112.github.io/post/gem5-memory-model/
- https://cs.stackexchange.com/questions/32149/what-are-system-clock-and-cpu-clock-and-what-are-their-functions
- http://learning.gem5.org/book/part1/example_configs.html








