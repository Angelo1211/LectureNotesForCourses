# Lesson 4: SPMD Programming model (Abstraction VS. Implementation)

The difference between an abstraction: 
* Something that is virtually true

And an Implementation
* Something that is physically true

For example: A multithreaded processor is to the operating system really like 8 cores the implementation could have been 8 physical cores or four with two way hardware threading. Most of the time you can think of this as 8 coress in actuality but not always.

Abstraction is like semantics, what the program means for example. 

Implementation is what is actually scheduled in the CPU.

--- 
Another example:

The instruction stream is what we think as the program but in reality the implementation of the actual stream is completely up to the processor as long as the results are the same as if it was executed.

## ISPC 

* Intel SPMD program compiler (ISPC)
* SPMD = "Single program multiple data"

You might have thought that it was implemented using threads. But really it was implemented using SIMD instructions. Abstraction might be multiple threads, implementation was 8 simd.

### Some Keywords

* programCount: number of simultaneously executing instances in the gang (uniform value)
* programIndex: id of the current instance in the gang (a non uniform value: "varying")
* uniform: A type modifier. All instances have the same value for htis variable. It is used purely as an noptimization.

## Two examples

### 1. Version one 

```C++
for (uniform int i=0; i < N; i + programCount){
    int idx = i + programIndex;
    etc etc etc... 
}
```

Interleaves the data for each of the instances in the gang( or so it would seem)

* Current instance 0 does: 0, 4,  8, 12 etc
* Current instance 1 does: 1, 5,  9, 13 etc
* Current instance 2 does: 2, 6, 10, 14 etc
* Current instance 3 does: 3, 7, 11, 15 etc


### 2. Version two

```C++
uniform int count = N / programCount;
int start = programIndex * count;
for( uniform int i =0; i < count; i++){
    int idx = start + i;
    ....
}
```

Doesn't interleave the data for each of the instances in the gang seems to do everything sequentially. Actually not good because the program will create the simd vector in columns not in rows.

* Current instance 0 does: 0,   1,  2,  3 etc
* Current instance 1 does: 4,   5,  6,  7 etc
* Current instance 2 does: 8,   9, 10, 11 etc
* Current instance 3 does: 12, 13, 14, 15 etc

### Raising level of abstraction with foreach

```C++
foreach(i = 0 ... N){
    ....
}
```

Compiler you make a decision about how to split things up.

100% equivalent to example 1.

### ISPC Abstraction vs implementation

* Single program multiple data programming model
    * Programmer thinks: tunning a gang is spawning programCount logical instruction streams (each with a different value of programIndex)
    * This is a programming abstraction
    * Program is written in terms of abstractions

* Single instructin multiple data SIMD implementation
    *  ISPC compiler emits vector instructions (SSE4 or AVX) that carry out the logic performed by a ISPC gang
    *  ISPC compiler handles mapping of conditional control flow to vector instructions by masking vector lanes etc

* Semantics of ISPC can be tricky
    * SPMD abstraction + uniform values
    * Allow implementation details to peak through abstraction a bit

ISPC only allows you to have gang sizes of 4, 8 and 16 based on the SIMD vector lane sizes


## ISPC reduction compute the sum of all array elements in parallel

If you tried to do something like this:

```C++
uniform float sum = 0.0f;
foreach(i = 0 ... N){
    sum += x[i];
}
```

It would nto even compile!! Why?

Sum is of type uniform float (one copy of variable for all program instances)
 
x[i] is not a uniform expression (different value for each program instance)
 
Results in compile time error

## Correct solution would be

```C++
float partial = 0.0f;
foreach(i = 0 ... N){
    partial += x[i];
}
sum = reduce_add(partial);
return sum
```

This would be the correct way. Each instance ( element of the simd vector ) is accumulating every 8th number in an 8 wide simd vector. 

For example, if the array was 1 2 3 4 5 6 7 8

The partial array would only contain 1 2 3 4 5 6 7 8 and would then reduce to 36.

But if you had instead 1 2 3 4 5 6 7 1 1 1 1 1 1 1 

partial: 2 3 4 5 6 7 8 9 

reduce: 44

Partial is kind of like a kernel that is added and slides down the vector array
Only at the end is it ever added 

## SPMD programming model summary

* single program multiple data model
* Define one function, run multiple instances of that function in parallel on different input arguments

1. Single thread of control
2. Call SPMD function
3. SPMD execution, multiple instances of function run in parallel (multiple logical threads of control )
4. SPMD function returns
5. Resume single thread of control

### ISPC gang abstraction is implemented by simd instruction on one core.

### All code in previous slide would have executed on only one core of CPU

### ISPC contains another abstraction called task that allows for multi-core execution. 