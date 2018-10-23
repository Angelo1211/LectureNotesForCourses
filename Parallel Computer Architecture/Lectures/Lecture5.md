# Lecture5: Parallel Programming Basics

Where is the parallelism in the ISPC code we saw from the other time?

#### The parallelism happens when you call the function and the parallelism comes from running the program multiple times

## Todays's topic : the process of parallelizing a program

### Creating a parallel program though process

1. Identify work that cane be performed in paralle
2. Partition problem into pieces of work that can be performed in parallel (and also all associated data with the work)
3. Manage Data Access communication and synchronization between the pieces of work

* Recall our main goals of achieving speedup

## Creating a parallel program

1. Start with some problem to solve
2. Break the problem into small subproblems
    * AKA tasks, jobs, work to do 
3. Then you need to assign that work to workers
    * Pieces of work that needs to be smartly assigned to workers
    * Assign worked to the actual pieces of hardware


## problem decomposition
* Break up a program into tasks that can be carried out in parallel
* If I have N processors you want to be assigning N tasks for them
* Create at least enough tasks to keep all execution units busy

### Main Challenge of Decomposition: identifying dependencies ( or lack there-of)


## Amdahl's law: dependencies limit maximum speedup due to parallelism

* Say you have a program that has a program that is sequential
* Say that S is the fraction of sequential execution of that is inherently sequential, that is dependencies prevent full apralle execution

If 50% of your program cannot be parallelix S = 0.5

Even if you parallelized the other 50% you would never be able to exceed a speedup of 1/S 

### Example

You have two tasks that take N^2 time to complete even if you perfectly parallelized the task this would take:

$\text{Speedup} = \frac{2 N^2}{\frac{N^2}{p} + N^2}$

speedup <= 2

### Amdahl's law graph summary

Even if your code has only 10% serial code and you manage to parallelize 90% of it perfectly No matter what you will only get a max of like 6x speedup

The key takeaway is that MOST (like 99.999% of your code) better be parallel or there is no reason to run it in a large super computer


### Who is responsible for decompostion?

In most cases a programmer

* Automatic decomposition of sequential programs is a very challenging research problem
    * Compilers must analyze program, identify dependencies 
    * But what if dependencies are data dependent ( not known at compile time )
    * Researchers have done very well with very simple nested loops
    * The magic parallelizing compiler for general purpose code has not been achieved yet

## Once decomposition is complete we need to think about assignment

* Assigning tasks to threads 
    * Think of tasks as things to do do
    * Threads are like workers
* the goal is to balance workload and reduce communication costs
* ALthough programmers are responsible for decomposition many languages can take responsability for assignment
* Assignment can be performed statically or dynamically at runtime

If you dont have a big problem it doesn't make any sense to paralellize at all.

Spawning threads is expensive. Typically you try to create the appropriate amount of threads for the given computer.

## Orchestration

* Involves:
    * Communicating between workers
    * Adding synchronization to preserve dependencies if necessary
    * Organizing data structurs in memory
    * Scheduling tasks
* Goals: reduce costs of communication / sync, preserve locality of data reference reduce overhead
* Machine details impact many of these decisions

## Mapping to Hardware

Usually do not worry a lot about it as a programmer

* Usually the operating system does it
* Mapped by the compiler 
    * like lanes in SIMD instruction
* Mapping the hardware 
    * Like cuda threads

We saw a similar question last lecture with mapping two threads to two cores

If you only have two threads but have 2 cores with 4 execution contexts total why is it better to map them each to one core instead of both to one?

Better performance because you don't have to share the resources of one core between threads.

Modern operating system will split it up properly


# A parallel programming example

## A 2D-grid based solver

* Goal: solve partial differential equations on a N+2 x (N+2) grid
* Iterative solution
    * Perform Gauss-Seidel sweeps over grid until convergence
  
* Problem: program is very sequential if you went in row it would depende on previous answers
* Solution: it is not sequential in the rows or the columns but there is parallelism in the diagonal 
  1.    Partition grid cells on the diagonal into tasks
  2.    Update values in parallel
  3.    When complete move to the next diagonal

* Problems with this approach: independent work is hard to exploit
    * Not much parallelism in the beginning and end of computation.
    * Frequent synchronization needed after completing a diagonal


* Another solution would be to simply change the thing you're trying to compute to something easier that still converges to something similar so you can paralellize it better. 

## New data flow dependency graph

1. Perform red update in parallel
2. Wait until all processors done with update
3. Communicate updated red cells to other processors
4. Perform black update in parallel
5. Wait until all processors done with update
6. Communicate updated black cells to other processors
7. repeat


## Three ways to think about writing this program

# 1. Data parallel way

Kind of how you think in matlab A + B data parallel for all elements.

This data parallelism also exists in ISPC 

But careful about non-deterministic problems

Some data parallel languages do not let you access elements except only forwards or repeats or other stream management tools


### Gather

I will give you two collections. An input array and a list of numbers. Gather means, just grab the numbers from the input array and creating a new one.


### Scatter

Is the opposite, given a list of numbers and some places put the numbers in those places 

## Returning to grid solver

We can also do this in a very Data parallel way but with care to use reduceAdd to avoid race condition when adding to the diff variable.

# Shared address space model ( abstraction )

* programs might create multiple threads 
* Threads communicate by reading / writing to shared variables
* Shared variables are like a big bulletin board
    * Anyone can write to these

* Need synchronization primitives to coordinate all the threads (programmer must handle synchronization )

Two primitive sin this example

* Lock: provide mutual exclusion, only one thread can acess given region at a time
* Barriers: we gotta wait for all threads to get here before continuing

In the data parallel version  we need to communicate and organize the workers

#### Locks 

Because each thread might be simultaneously reading the diff value and then writing to it there is no guarantee that the results will be overwritten. By adding a lock we can guarantee that the value is only ever read and written by one thread at a time. This guarantees that they will be incremented one by one and reading the correct previous values.

The three set of instructions must be "atomic"

Some places will use the word lock, other use atomic keyword or atomic add that essentially perform the same operations

# Summary 

## Amdahl's law
* Overall maximum speedup from parallellism is limited by amount of serial execution in a program


## Aspects of creating a parallel program

* Decomposition to create independent work, assignment of work to workers, orchestration (to coordinate processing of work by workers), mapping to hardware


## Focus of today: identifying dependencies

## IDentifying locality, reductin synchronization