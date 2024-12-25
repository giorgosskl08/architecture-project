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
        > Is a single-cycle CPU, meaning it can execute only one instruction per cycle. It acts as the foundational class for the AtomicSimpleCPU and TimingSimpleCPU but cannot be used on its own. The class defines common functions for SimpleCPU models, such as checking for interrupts, controlling setups and actions, and advancing the Program Counter (PC) to the next instruction while maintaining the architected state. It also implements the ExecContext interface, which instructions use to access the CPU.
      * AtomicSimpleCPU:
        > Extends BaseSimpleCPU and utilizes atomic memory accesses. This CPU model reads and writes memory atomically and uses functions to handle the actions performed in each CPU cycle. It determines the overall cache access time using latency estimates from the atomic accesses. Moreover, AtomicSimpleCPU defines the port used to connect to memory and attaches the CPU to the cache, ensuring smooth memory operations.
      * TimingSimpleCPU:
        > Another extension of BaseSimpleCPU, but it uses timing memory accesses, which are more detailed but slower due to delay. This model schedules response functions for future execution after sending out a request. TimingSimpleCPU handles responses from memory to the sent accesses, repeating procedures if a NACK is received. It also defines the port used to connect to memory and connects the CPU to the cache, much like AtomicSimpleCPU.
        
      ii. Minor CPU:
   
   > An in-order processor model with a standard pipeline, but its data structures and execution behavior are configurable. This model provides a framework to match the micro-architectural level of a similar processor with strict in-order execution behavior. It also visualizes the position of an instruction in the pipeline using the MinorTrace/minorview.py format/tool, helping users understand the pipeline stages and execution flow.

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

   b.
   | Parameter | TimingSimpleCPU | MinorCPU |
   | --------| ------ | ----- |
   | host_seconds | 0.02 | 0.09 | 
   | sim_seconds | 0.000046 |  0.000037 |
   | Number of indirect misses | --- | 137 |
   | sim_freq | 1000000000000 | 1000000000000 |
   | host_mem_usage | 674144 |  678756 |
   | Number of instructions committed | 13196 | 13249 |

       
    The differences are mostly related to time. They are justified considering the fact that TimingSimpleCPU lacks speed in comparison to MinorCPU.  
    The similarities are also justified because they concern a) the frequency which is determined by us, b) the memory used by the host, and the number of instructions committed which depend on our program and are not influenced by the CPU.

   c. Changed parameters and comparison of the 2 CPU models:
   
      * Different memory type:
        * For the TimingSimpleCPU we used the "CPUDDR4_2400_8x8" memory type:
          ```
          $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPUDDR4_2400_8x8 configs/example/se.py --cpu-type=TimingSimpleCPU --mem-type=DDR4_2400_8x8 --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
          * There is a small decrement in the time of execution while using the "CPUDDR4_2400_8x8" memory type. A number of seconds simulated before = 0.000046 and after = 0.000045. 
          * Also a increment in the instruction rate is noticed: Simulator instruction rate before=573016  and after=601474. 
        * For the MinorCPU we used the "SimpleMemory" memory type:
          ```
           $ ./build/ARM/gem5.opt -d fib_results_MinorCPUSimpleMemory configs/example/se.py --cpu-type=MinorCPU --mem-type=SimpleMemory --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
           * The instruction rate has increased with the use of the "SimpleMemory" type. Specifically, simulator instruction rate before=147368 and after=346289.
           * The simulated time seems to be decreased. Number of seconds simulated before = 0.000037 and after = 0.000030.

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

Η συνάρτηση κόστους που θα χρησιμοποιηθεί είναι:

**Κόστος = (n1d / 16kB) + (n1i / 16kB) + (k1 / 2)^1.4 + (n2 / 256kB) + (k2 / 2)^1.3 + (c / 8B)**

---

| Benchmark | L1d Size (kB) | L1i Size (kB) | L2 Size (kB) | L1 Assoc | L2 Assoc | Line Size | Cost      | CPI       |
|-----------|---------------|---------------|--------------|----------|----------|-----------|-----------|-----------|
| speclibm0     | 32            | 64            | 512          | 1        | 2        | 64        | 6.305625  | 2.650259  |
| speclibm1     | 64            | 128           | 512          | 2        | 2        | 64        | 9.005625  | 2.631263  |
| speclibm2     | 64            | 128           | 512          | 2        | 4        | 64        | 9.631625  | 2.631263  |
| speclibm3     | 64            | 128           | 512          | 2        | 4        | 128       | 12.131625 | 1.991390  |
| speclibm4     | 64            | 128           | 1024         | 2        | 4        | 128       | 14.131625 | 1.991078  |
| specbzip0      | 32            | 64            | 512          | 1        | 2        | 64        | 6.305625  | 1.589715  |
| specbzip1      | 64            | 128           | 512          | 4        | 2        | 64        | 10.505625 | 1.724184  |
| specbzip2      | 64            | 128           | 512          | 4        | 2        | 128       | 13.005625 | 1.635832  |
| specbzip3      | 64            | 128           | 2048         | 4        | 2        | 128       | 21.005625 | 1.594113  |
| specbzip4      | 64            | 128           | 2048         | 4        | 4        | 128       | 21.631625 | 1.589715  |
| specbzip5      | 128           | 128           | 4096         | 4        | 4        | 128       | 30.631625 | 1.556339  |
| specmcf0      | 32            | 64            | 512          | 1        | 2        | 64        | 6.305625  | 1.185033  |
| specmcf1      | 64            | 128           | 512          | 2        | 2        | 64        | 9.005625  | 1.185033  |
| specmcf2      | 64            | 128           | 512          | 2        | 4        | 128       | 12.131625 | 1.179751  |
| specmcf3      | 64            | 128           | 2048         | 2        | 4        | 128       | 20.131625 | 1.179751  |
| spechmmer0      | 32            | 64            | 512          | 1        | 2        | 64        | 6.305625  | 1.176616  |
| spechmmer1      | 64            | 128           | 512          | 1        | 2        | 64        | 9.005625  | 1.143930  |
| spechmmer2      | 64            | 128           | 512          | 2        | 2        | 128       | 12.131625 | 1.118699  |
| spechmmer3      | 64            | 128           | 512          | 2        | 4        | 128       | 14.131625 | 1.112977  |
| sjeng0    | 32            | 64            | 512          | 1        | 2        | 64        | 6.305625  | 7.041210  |
| sjeng1    | 64            | 128           | 512          | 1        | 2        | 64        | 9.005625  | 7.041210  |
| sjeng2    | 64            | 128           | 512          | 2        | 4        | 128       | 12.131625 | 4.976535  |
| sjeng3    | 64            | 128           | 2048         | 2        | 4        | 128       | 20.131625 | 4.974881  |
 | specmcf | spechmmer | sjeng | speclibm
---

Βέλτιστη Αρχιτεκτονική για Κάθε Benchmark
Με βάση τη σχέση κόστους και απόδοσης, οι βέλτιστες αρχιτεκτονικές είναι:

1. **`limb`:** 
   - Προτεινόμενη Αρχιτεκτονική: `limb3`
   - **Κόστος:** 12.131625
   - **CPI:** 1.991390

2. **`zip`:**
   - Προτεινόμενη Αρχιτεκτονική: `zip4`
   - **Κόστος:** 21.631625
   - **CPI:** 1.589715

3. **`cmf`:**
   - Προτεινόμενη Αρχιτεκτονική: `cmf2`
   - **Κόστος:** 12.131625
   - **CPI:** 1.179751

4. **`mer`:**
   - Προτεινόμενη Αρχιτεκτονική: `mer3`
   - **Κόστος:** 14.131625
   - **CPI:** 1.112977

5. **`sjeng`:**
   - Προτεινόμενη Αρχιτεκτονική: `sjeng2`
   - **Κόστος:** 12.131625
   - **CPI:** 4.976535

---


## Βibliography
- https://stackoverflow.com/questions/58554232/what-is-the-difference-between-the-gem5-cpu-models-and-which-one-is-more-accurat
- https://www.gem5.org/documentation/general_docs/cpu_models/SimpleCPU
- https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu
- https://nitish2112.github.io/post/gem5-memory-model/
- https://cs.stackexchange.com/questions/32149/what-are-system-clock-and-cpu-clock-and-what-are-their-functions
- http://learning.gem5.org/book/part1/example_configs.html








