# Lecture 6: Performance Optimization Part 1: Work Scheduling and distribution

## Goals of writing high-performance parallel programs

* Key goals (at odds with each other)
    * Balance workload onto available execution resources
    * Reduce communication to avoid stalls
    * Reduce extra work perofromed to increase parallellism, manage assignemt, reduce communication

* Techniques to achieve these goals

* Optimizing parallel code consists on iteratively going through choices of better decomposition, better assignment and better orchestration

## Tip 1: 

Always implement te simplest solution first, then measure performance to determine if you need to do better.

## Scheduling work onto workers

Ideally this is about balancing the workload. All processors would be computin all the time during program execution. 

Why? Because of Amdahl's law we see that a small amount of imbalance can significantly bound the maximum speedup

## Static assignment

The easiest way to assign work to the processor. If we assume that all work is going to take exactly the same amount of time we can just evenly divide the work between all processors and they should all complete their jobs at the same time.

Static means that before you even run the program you decide what work and what tasks will be done by what workers. No decisions done while program runs all done beforehand

--- 

### But when would it be possible to do this?

When we know how much time it would take and time is predictable. 

When the cost of the work is predictable

##### One example would be when all the tasks have the same cost

##### Another one would be when we know the lenght of the pieces of work ahead of time

## Dynamic assignment 

Program determines assignemnt dynamically at runtime to aensure a well distributed load. The execution time of tasks, or the total number of tasks however is unpredictable.

We use a lock with a counter to implement this work indexing system and dynamic scheduling

##### But there is a performance problem with this implementation

Locking is not free 

Problem for a dynamic scheme is that I am now doing work to synchronize that I would not have done in the sequential program. Extra work to synchronize might not be as good.

Is it a problem?

It is not a problem if the cost of the functions are extremely high, if they are very low then yes.

### A trick to improve performance

We add some granularity by not perfomring just one number per thread but doing more tasks per thread

However, how do we set this number? 

The task size should be small so that you get big balance but you also want it to be big to reduce talking overhead.

## Choosing tasks sizes: Good rule of thumb

* Useful to have many more tasks than processors
    * Many small tasks enable good workload balance via dynamic scheduling
    * Motivates small granularity
* But we also want as few tasks as possible to minimize overhead
    * Motivates large granularity
* Granularity depends on many factors

## Getting smarter scheduling 

If you had a task that is way longer than the rest, you would want to do that one first in a sort of latency hiding technique. This is at the core of smarter scheduling. 

If you know something about scheduling you can get a much better algorithm.

## Semi-static assignment

* Cost of work is predictable for near-term future
    * Results from recent past are a good predictor of near future
* Application periodically profiles itself and re-adjusts assignments
    * Assignment is static in between re-adjustments

Kind of like an adaptive mesh, or a particle simulation.


## Dynamic assignment using a work queue 

The reason why we change our code was to reduce overhead of synchronizing between threads. One solution was to reduce the fighting for one variable by reducing how often you fought for it.

Another solution would be to copy the strategy we used with the diff variable and instead of having one resource shared we now have many queues that each thread has and if your queue finishes you go to the other threads queue.

### Summary of first part of lecture

* Our challenge is to achieve good workload balance
    *  We want all processors working at the same time otherwise we would be idling some resources
    *  But we also want low cost for achieving this balance
        *  Need to minimize scheduling and assignment overhead
        *  Minimize the synchronization costs
*  We fist examined static assignment vs dynamic assignment
    *  But it is not just an either or, more like a continuum
    *  We shuld use up front knowledge of workload as much as possible to reduce load inbalance and task management / synch costs
    *  Realize that in the limit the system knows everything and hence this is equivalent to a fully static assignment


* Issures discussed today span decomposition, assignment and orchestraition


## Scheduling fork-join orchestration

So far we have seen onlytwo types of parallel programs:

##### Data parallelism:
Perform the same sequence of operations on many data elements

foreach

ispc bulk task launch

bulk cuda thread launchh
 
map function

OMP parallel for

Data parallel because every piece of data can be run in parallel

Here are many iterations of a loop, you can do them in parallell if you wish.

### Common parallel programming patterns

Explicit management of parallelism with threads:

Create one thread per execution unit available on the CPU or per amount desired concurrency

### Consider divide and conquer algorithms
Like Quicksort

Sorting the left sub list and the right sublist is an independent task so it can be done in parallel

## Fork-join pattern

* Natural way to express independent work inherent in divide and conquer algorithms
* Cilk Plus
    * C++ language extension
    * Originally developed at MIT, now adapted to standard
* Cilk spawn (fork)
    * Semantics: invoke function, but unlike standard function clal, called may continue executing asynchronously with execution foo
* Cilk synch (join)
    * Semantics: returns when all calls spawned by current function have completed
    * Kind of like synching with the semantic calls

There is an implicit cilk_sync at the end of every function that cotains a cilk_spawn implying that when every spawned function has been done running you are done doing any work associated with that function

Normally in C++ when you call a function the main thread now switches to that other function and executes whatever it is doing. In the fork join pattern the function is executed and then moved to work on that task on another thread while the main thread continues forward

Does not mean they are necessarily running in parallel but just that they are independent, the system actually chooses to run it independently or not 
## Abstraction vs implementation

* Notice that spawn abstraction does not specify how or when spawned calles are scheduled to execute
    * Other that it *may* ru concurrently with caller and with all other caller spawns
* But sync does serve as a constraint on scheduling
    * All spawns must complete before synch returns

## Writing for join programs 

* Programmer is supposed to tell the programmer where the parallel work is and the computer figures out when to schedule it 
  

#### Parallel programming rules of thumb

* Want at least as much work as parallel execution capability (ideally spawn as many threads as there are cores)
* Want more independent work than execution capability to allow for good workload balance
    * Parallel slack, ratio of independent work to machine's parallel execution capability ( 8 is a good ratio)
* Nottoo much independent work so granularity is so small that there is overhead now

## Scheduling fork-join

One possible implementation would be to every time that someone calls spawn you just run it in parallel, but too many threads will be very slow 

## Cilk plus implementation 

Uses a pool of worker threads, one for every processor

*   Imagine all threads are created at launch 
*   Exactly as many threads as execution contexts

Constantly ask what they should be doing next and there is a list that keeps track of what should be done next

### At a spawn sshould the thread run child or continuation?

Run continuation first: record child later for execution (Child stealing)
* Child is made available for stealing by other threads

Run child first: record continuation later for execution
* Continuation is made available for stealing by other  threads 
* It is like a breadth first graph traversal
* A really long list gets created for thread zero 
* Order of execution is the same as for program with spawn removed 

## Cilk uses greedy join scheduling

* Greedy join scheduling policy
    * All threads always attempt to steal if there is nothing to do (thread only goes idle if no work to steal is present in the system )
    * Worker thread that initiated spawn may not be thread that executes logic after sync 

* Remember:
    * Overhead of bookkeeping steals and managing sync points only occurs when steals occur
    * If large pieces of work are stolen, this should not occur frequently
        * Most of hte time, threads are pushing popping local work from their local dequeue

Every thread will pull work from the bottom of its own list and other threads will steal from the top of the list which tends to have the bigger work, in such a way that you reduce stealing

# Cilk Summary

* Fork-join parallism  is a natural way to express divide and conquer
* Cilk plus runtime implements spawn/ sync abstraction with a locality aware work stealing scheduler
    * Always run spawnd child (continuation stealing)
    * Greedy behavior at join, threads do not wait at join immediately look for other work to steal


