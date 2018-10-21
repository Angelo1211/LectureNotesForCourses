
# Lecture 1: Why parallelism, why efficiency?

## What is a parallel computer? 

* A commom definition is: 
    * A parallel computer is a **collection of processing elements** that cooperate to solve problems **quickly**
    * That is, we care a lot about performance and efficiency
    * We are going to use multiple processors to get them


## Speedup is a major motivation of using parallel computing

For a given problem:

 $\text{speedup} = \frac{\text{execution time using 1 processor}}{\text{execution time using P processors}}$

This is what we normally expect, but sometimes this is not true. As in the example in class when they did not achieve speedup due to communication limiting speedup. (walking across room)

Ideas to go faster:
* Decrease distance
* Increase communication

### Demo 2

* Figuring out how to parallelize the actual work is easy
* But most of the time is actually spent getting people to work together
* Some students ran out of work (went idle) while others had too much to do

### Demo 3  (Massively parallel execution)

* Significant communication more than computation
* Communication costs can dominate a parallel computation, severely limiting computation

# Three Themes

## Theme 1:

Designing and writing parallel programs that scale!

scale = if you add more processors it will go faster

* Thinking about efficiently performing tasks in parallel
    * Decompossing work into pieces that can safely be performed in parallel
    * assigning the work to the processors
    * Managing communications between processors so that it does not limit speedup


* Mechanisms for performing the above tasks
    * writing code in popular paralle programming languages

## Theme 2:

You need to understand how computers work to be able to implement efficiently

* Mechanisms used to implement abstractions efficiently
    * Performance characteristics of implementations
    * Design trade-offs performance vs convenience vs cost 

* why do I need to knwo about hardware?
    *  because characteristics of the hardware really matter ( like communication issues in earlier demos did)
    *  Because you care about efficiency and performance 


## Theme 3:

Thinking about efficiency

* It runs fast != it is efficient
* Just because your program runs faster on parallel computer it does not mean it is using the hardware efficienty
    * is a 2x speedup with 10 processors a good result?

* programmers perspective: make use of provided machine capabilities
* Hardware designer perspective: choosing the right capabilities to put in system (performance/ cost , cost = silicon area?, power?, etc,)

# Three assignments

One for each parallel programming environment
* ISPC programming on multi-core CPU
* CUDA progrmaming on NVIDIA GPU
* fine-grained locking on multi-core CPU

# Four HW's 

Not easy!

60% programming 

40% on written

# REview: Basice of how a computer work

## 1. Simplest computer model

1 processor + memory

**What is a computer program?**

A script that a computer can run. A collection of data structure and algorithms. 

For a computer a program is just a list of instructions.

script -> compiled -> list of instructions 

**What is a processor?**

The machine that decodes thos instructions and executes them.

A processor executes instructions.
Executing an instruction modifies the computer state.

MOst simplest processor:
* Contains a unit that does things
    * Execution unit: performs nstructions which may modify values in registers or memory
* A unit that stores data
    * Registers or memory
    * Maintain program state store value of variables used as inputs and outputs to operation

## example instruction: add

* Step 1: 
Processor gets next program instruction from memory (figure out what processor should do)


```python
add R0 <- R0, R1 
 ``` 
 Essentially saying: 

 *Please add the contents of register 0 to the contents of register 1 and put hte results back in register 0*

* Step 2: obtain inputs to the operations from registers
    * R0 = 32
    * R1 = 64
  
* Step 3: 
    * Perform addition operation:
    * Execution unit performs arithmetic, result is 96

* Step 4: 
    *  Put back into register

## What is memory?

Stores the list of instructions

Just an array of data

Every piece of data has an address
(we assume memory is byte addressable)

One of the instructions that a processor can do is move data from memory to the registers


```python
ld R0  <- mem[R2] 
 ``` 

 ## Review of how computers work (from a processor's perspective)

* Computer program:
    * list of instructions
* What is an instruction
    * An operation that the processor can do
    * Executing an instruction tends to change the computer's state
* What does state mean
    * The variabels in a program manipulating for these instructions

# Why parallel computing?
* Back in early 2000's nobody cared about parallel processing cores were doublign in performance every 18 months approx.
* No point in writing parallel since computers got fast really fast

### Computers kept getting faster
1. Exploiting instruction-level parallelism( running more than one instruction per clock using superscalar execution)
2. Faster clock speeds ( doing instructions faster more per second )

Could computers run instructions in different order? Yes!
#### Instruction level parallelism
* processors leveraged parallel execution but invisible to programmers
* using instruction level parallelism
    * Instructions appear to be executed in order but independent instrucitons can be executed simultaneously without impacting correctness
    * If the processor figures out its okay to do in parallel it does it

$a = x*x + y*y + z*z$

Running this program would take about 5 units of time in a simple scalar.

Yet, all multiplications are independent, so you could paralelize by performign those first simultaneously and then doing the additions individually.

Diminishing benefits of superscalar execution.

After 4 instrctions the benefit is not that big at all.

Execution units not cores.

Currently no more speedup expected from this.

#### Frequency scaling

Also has died out as of 2005.

Essentially we stopped doing more instructions at the same time per core. We stopped increasing clock speeds.

What happened with speed?
* We hit the power wall.

## The power wall

Power consumed by transistor:

Dynamic power and capacitive load x voltage ^2 x frewuncy

static power: transistors burn power even when inactive due to leakage

High power = high heat

Power is a critical design constraint in modern processors

Running faster and faster will burn more power and would melt the chips

**Only remaining way to go faster is to turn more transistors into more cores** 

Parallel computing today is the only way to make stuff run faster


# Summary
* Today single theread performance is improving very slowly
    *  to run faster programs have to use multiple processing elements
    *  you need to know how to write parallel code
* Writign parallel code can be challenging
    * Need to know how to partition, communicate and synchronize 
    * Knowledge of machine characteristics is important

