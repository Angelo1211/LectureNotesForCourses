# Lecture 2 : Modern Multi-core processors

Understanding forms of parallelism.

## Quick review:

1. Why does single instruction stream performance only improved slowly in recent years
   * Because we have hit the power wall and we cannot increase frequency or instruction level parallelism much more
2. What prevented us from obtaining maximum level speedup in the parallel programmign class demos?
    * Communication delays, lack of organizing the resources properly 


## Part 1: Parallel execution

Entire lesson uses this same example program

```C 
void sinx( int N , int terms, float * x, float* result){
    for (int i =0; i< N; i++){
        float value = x[i];
        float numer = x[i] * x[i] * x[i];
        int denom = 6; //3!
        int sign = -1;

        for (int j=1; j <= terms; j++){
            value += sign * numer / denom;
            numer *= x[i] * x[i];
            denom *= (2*j + 2) * (2*j + 3);
            sign *= -1
        }

        result[i] = value;
    }
}
```

Every single input element in the array can be paralized by running all of the inner for loop code. 

We can then compile these instructions. Looking at the world's simplest processor:

#### Worlds simplest processor

Has three components:
1. Fetch / Decode
2. ALU (execute)
3. Execution Context (state)

If This was one of those super scalar processor what would we add?

#### Superscalar processor
Uses instruction level parallelism

1. Fetch / Decode  
    1. Fetch Decode 1
    1. Fetch Decode 2
2. ALU'S and executions
    1. ALU 1
    2. ALU 2
3. Execution Context (state)

This new fancy processor would not see any speedup for any process that is related to each other as the first couple of instructions are. 

#### Aside 
---
We compare the simple diagram to intel's pentium 4 diagram and although it has a lot more boxed there are some similar ones. 

The not mentioned boxes have jobs like:
* moving stuff to and from registers
* logic to find parallelism 
* Out of order control logic
* Fancy branch prediction
* memory pre-fetching
* Data cache

  
**The majority of a chip's transistors is actually dedicated to perform operations taht help the single instruction stream run fast!**

---

## First big idea  of this course:

Intel was spending most of the time on improving things that would not be beneficial to instructions that might be linked together.

For example:
* All the elements in the array of numbers of our program could have been run in parallel but none of the single stream improvements actually would help us do that.

## 1. GET RID OF ALL STUFF TO MAKE SINGLE STREAM FASTER

That is, rather than use transistors to increase sophistication of single instruction stream use the transistor count to add mroe core to processor.

With the two simple processors now run twice as many elements. 

Core might be slower ( 0.75 total speed ) but if there are 2 we get

$2 * 0.75 = 1.5$  (Speedup potential!)


We can imagine a for all - relationship instead of a for loop
* For loop:
    * Do this and then do this again, etc etc 
* For all: 
    * All of these can be done independently

But what is very nice is that by taking all the fancy stuff out of an advanced processor you can do more than just fit one more processor, you can fit sometimes even  16!

**Could in theory be processing 16 elements of an array at the same time**

## Data parallel expression:

Parallelism is across iterations of the loop.

All iterations of the loop do the same thing: **evaluate the sine of a single input number**

##### If something can be run idependently they can be run in parallel. But not only is each iteration independent, each iteration is running the same code.


## Second big idea of this course

## 2. AMORTIZE COST AND COMPLEXITY OF MANAGING AN INSTRUCTION STREAM ACROSS MANY ALUS

This creates

# SIMD PROCESSING

Single instruction, multiple data

Same instruction broadcast to all ALU's 

Executed in parallel on all ALUS

ALUs are very cheap

1. Fetch / Decode  
2. ALU'S and executions
    1. ALU 1
    2. ALU 2
    2. ALU 3
    2. ALU 4
    2. ALU 5
    2. ALU 6
    2. ALU 7
    2. ALU 8
3. Execution Context (state) MUCH BIGGER

To make use of this we must rewrite most of the code to use intel intrinsics to be able to tell the compiler that we want to perform the operations on multiple numbers at the same time. In the example case it was 8.

Known as AVX instructions or SSE instructions. 

With this, we could now process 16 instruction streams with 8 numbers at a time. 128x speedup possible.

## What about conditional execution?

WE have a problem since we assumed when doing simd that each piece of data would do the same instructions but that is not the case. Any branching will cause this issue.

If this was two cores there would be no problem, however because it is just one core we have a problem. If it were two you could do the true ones in one and the false in the other.


What we do is actually just turn off the ALU's taht would receive the false data adn execute the instructions only in the true. 

Then for the false we turn off the true ALU's and turn on the FALSE. 

(review this!)


So not all ALUS do useful work

At worst case 1/8 performance 

---
## Review 

1. We started with a single processor that can perform one instruction at a time.
2. We noticed that some instructions are independent so we designed a new processor (superscalar processor ) that can decode two instructions and execute two. Still one core though!
3. In many programs you have a lot of data and a lot of that data can be done in parallel and independently. We can then just duplicate the simple core and perform operation on each data once at a time. 
4. Next we noticed that all iterations of the loop do the same thing. Which means that executing iterations should be able to share the same instruction stream.
5. We only need one fetch / decode to get the instructions that we will give to multiple ALUS.
6. ALUS are much smaller than fetch decode transistor wise
7. These can all be combined to have multiple instruction streams that work on multiple pieces of data. 
8. 


## Example of parallel programming in c++ 11

I cannot copy the code but I will describe what happens in words. 

1. The code creates a thread that calls a function that prints hello 100 times
2. This is then called again in the main thread 
3. So we have two threads 0 and 1
4. Thread 0 starts to print fast while thread 1 is setting up.
5. Thread 1 then begins printing while thread 0 is still printing and the results are interleaved
6. Thread 0 finished printing
7. Thread 1 continues printing until the end and completes
8. Join says wait to thread one until zero is done to continue. It kind of is like they both take different roads and at the point the road intersects if one of them is told to join it will wait until the other one gets there to continue. And more than continue it will just merge onto the next road and become one single car. Weird analogy but works

This is what we mean by generating a new instruction stream.

#### Multithreaded means using multiple instruction streams.

If we had four cores it would be like calling 4 instructions streams.

### Example architectures

1. 4core processor with SIMD needs around 128 "pieces of data" to run efficiently
2. 20core nvidia processor with simd needs around 40000

---
# Part 2: accessing Memory

### Terminology

* Memory Latency
    * The amount of time for a memory request ot be serviced form a processor
    * Measured in cycles, or nano seconds
    * Reduce latency: either you walk faster or you reduce the distance
    * **Caches are here to reduce latency**

* Memory Bandwidth
    * The overall rate the memory system can provide data to a processor
    * Examples: 20gb/s
    * More bandwidth does not reduce latency 
    * Adding more highway lanes, vs faster cars

If memory is really far away caches reduce the distance significantly by taking a lot of data from memory in one go and reducing the need for it to be fetched from main memory


### Stalls

* A processor stalls when it cannot run the next instruction in an instruction stream because of a dependency on a previous instruction.
* Accesssing memory is a major resource of stall
* Memory access is 100's of cycles

### Caches

* L1 cache is 32kb 3 cycles
* L2 cache is 256kb 10 cycles
* L3 cache is 8mb 20 cycles
* Main memory is in the gigabyte range
* Bandwidth is about 25gb/s form l3 to main memory


### Prefecthing to reduces stalls by hiding latency

* All modern cpus have the ability to prefetch data into catches
    * Processsor analyzes the program's access patterns and predicts what data the program will access soon
* Reduces stalls since data is resident in cache when accesss 
    * Could also reduce performance is guess is worng

### Multi-threading reduces stalls
* Idea is to interlave processing of multiple threads on the same core to hide stalls
* Like prefetching is a latency **hiding technique,** not actuall latency reducing
* This does not have to be confused with software multithreading. This is multithreading at the hardware level
* To do this you have multiple execution units that contain all of the thread information to execute the given instruction stream. Such as Registers, program counter etc etc  
* Four different states of four different threads
* What happens when a processor issues a load instruction:
    * It stalls the process for like 200 iterations 0.5% efficiency 
    * It's as if it puts the clothes in the washer. It now starts doing somethign else.
    * Might be running instructions for thread two. Which might also stall, at which point you call instructions of thread 3 etc etc 
    * Once thread one is done it continues executing it 
    * So having more of these execution units is like having more logical cores
    * IE hyper threading
* It makes it so it doesn't matter if a process takes a long time the processor is always busy, which makes efficiency close to 100% 
* Multithreading is about efficiency
* SIMD is about adding resources
* Multicore is about adding resources

### Throughput trade off

* Key idea: Potentially increase time to complete work by any one thread, in order to increase overall system throughput when running multiple threads. 

# Key class idea: I'm going to design a computer that runs really well only when there is a lot of software work to be done 

### Storing executing contexts

We must consider on chip storage of execution contexts as a finite resource
(normally l1 cache)

## Kayvon's fictitious multi-core chip

### Key ideas
* 16 cores
* 8 wide SIMD ALUS per core (128 total)
* 4 thread per core
* 16 simultaneous instruction streams
* 64 concurrent instruction streams 
* 512 independent pieces of work are needed to run chip with maximal latency hiding ability 

ALl of these ideas are true for CPU's and are also true for GPU's 

### GPUS: extreme throughput oriented processors

* NVIDIA gpus have 64 hardware threads 
* NVIDIA warp = hardware thread
* Instructions operate on 32 pieces of data at a time (instructions streams are called warps) 
* 32 wide simd instruction stream
* WArp = issuing 32-wide vector instructions
* Different instructions from up to four warps can be executed simultaneously (multi-threading)
* Up to 64 warps are interleaved on the core (64 logical cores?)
* Every clock it chooses 4 of the 64 threads to run  
* It can run 128 operations per core, per clock 

# NVIDIA GPU Architecture

## 20 cores

## 4 hardware threads (Warps)

## 32 wide SIMD

## Superscalar

## 64 logical threads per core

## 2048 independent pieces of work per core to be processed concurrently

## 40960 to be  maximal latency hiding

NVIDIA assumes that you will only run things that have loads of parallelism

### CPU VS GPU memory hierarchies

This is why the cpu has so many caches, you only have two threads, so if one stalls you better have the other one to do stuff at the same tiem so you want to be sure it has its memory.

The Nvidia gpu assumes you  will stall to main memory so the l2 cache is much smaller, but that you will be doiing so much stuff it will compensate. 

## Thought experiment

Task: element wise multiplication of two vectors A and B

Assume vectors contain millions of elements
* load input A[i]
* load input B[i]
* compute A * B
* Store into C[i]


Three memory operations (12 bytes) for every MUL
* 4 bytes for A, 4 bytes for B, 4 Bytes for C
* 


NVIDIA GTX 1080 can do 2560 muls per clock @ 1.6GHZ

Need ~45TB/sec of bandwdith to keep functional units busy (only have 320 GB/s )

I only actually have like 1% of the bandwidth that i actually need. But it is still 4.2x faster than eight core CPU!
(3.2ghz eight core GPU has 76GB/sec memory bus willl exhibit ~3% efficiency on this computation)

## Bandwidth limited problem

Processors requresting data at too high rate the memory system cannot keep up. 

**No amount of latency hiding helps this**

Overcoming bandwidth limits are a common challenge for application developers on throughput-optimized systems. 

Is the processor too fast for hte memory?

Best way to think about throughput is thinking about two pipes

**Caches do not help because you only ever read and write elements of the array one time so saving them is useless**


## Bandwidth is a critical resource

Performant parallel programs will:

* Organize computation to fetch data from memory less often
    * Reuse data previously loaded by the same thread
    * Share data accross threads (inter thread cooperation)

* Request data less often (instead do more arithmetic: it's free)
    * Arithmetic intensitiy: ratio of math operations to data access operations in an instruction stream
    * Main point: programs must have high arithmetic intensity to utilize modern processors efficiently
---

## Terms that are important to know for hte rest of this class
* Multi-core processor
    *  A processor that contains more than one arithmetic logic unit, instruction decode / read. Important to differentiate logical cores form hardware cores. Essentially has multiple instruction streams in one single processor
* SIMD execution
    * Single instruction multiple data. If you know the code that you want to execute is going to be executed in the exact same way for multiple data you can simply use one instruction decode unit to send the same instruction to multiple ALU's that will perform the same instruction for the data in parallel;
* COherent control flow
    * When you use masks to "turn off" simd lanes to only execute code on some of the alu's instead of all
* Hardware multi-threading
    * Interleaved multi threading
        * When you switch your execution context because you are stalled by a memory read and you then interleave threads as they stall and receive their request. used to hide latency
    * Simultaneous multi-threading
        * When you have multiple instruction streams being acted on by your hardware at the same time. Like in NVIDIA GPU core
* Memory latency
    * The delay from a request of data to the memory sytem and that data actually arriving to the cpu
* Memory bandwidth
    * The size of the "data highway". Essentially how much data can be sent to and from memory to the cpu at a given time
* Bandwidth bound application
    * An application that is requesting ddata at a rate higher than what the memory sytem can deliver
* Arithmetic intensity
    * Ratio of math operations to data access operations in one instruction stream


# Review Again 

1. You have some code that you can run on a simple processor that completes one instruction per clock.
2. Without changing the code but still want to run faster you can: 
    1. Use superscalar processors that check for parallelism at the instruction level
3. You can also rewrite the code and spawn threads that will be in charge of a portion of the code. Essentially making the program multi processed.
4. You can rewrite again and also add SIMD cores that realize that the program is running many iterations of the same loop body. So you are essentialyl repeating the same instructions again and again. YOu can instead get the instructions once, then execute them on multiple data
5. You can also add execution context switching that will allow you to switch away from a thread that is stalled and waiting for data to a new thread and keep doing calculations while you wait.
6. Superscaling could even mean that you have both SIMD and scalar processors and you are executing both simultaneously if necessary.
7. You can also add l1 and l2 caches to reduce memory latency by avoiding having to go to main memory


## Thought experiment
Have an application that is spawning two threads

The application runs on this processsor 
* Two cores
* Two execution units context per core
* one instruction per clock
* One instruction is an 8 wide simd 

How do we distribute these?

The Operating system decides how to distribute

