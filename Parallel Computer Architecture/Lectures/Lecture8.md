# GPU programming and CUDA programming

## Main topics of lecture

1. History: how graphics processors originally designed to accelerate 3D games evolved into highly parallel compute engines that are capable of stuff like:
    1. deep learning
    2. Computer vision
    3. Scientific computing
2. Programming gpu's using the cuda language
3. An even more detailed look into GPU architecture

### Recalling the basic GPU architecture

It is a multi core chip with SIMD execution units within a single core with multithreaded execution on a single core 

### Graphics 101 history

GPUS were created for graphics. The problem is: we need to take some description of a world: either surfaces or triangles or meshes etc etc 

##### Rendering in a nutshell is simply computing how each triangle in 3D mesh contributes to appearance of each pixel in this image

--- 

## Sidenote how to explain a "system
* Step 1: Describe the things (key entities) that are manipulated by the system
    * The nouns what are the things we are computing
* Step 2: Describe the operations the system performs on those entities 
    * The verbs 
  
---




## The 3D graphics algorithm / workload

### Real time primitives are very simples

1. Triangles (surfaces)
2. Points   (vertices)
3. Lines  

Once you have a triangle with pixels on screens there is the question of what is covered by that triangle. That is, which of those pixels covered by the triangle should be lit up.

So:

* We have points
* We connect them to make primitives
* Those triangles cover some pieces of the screen which we call fragments
* Once we have the pieces we can compute the color of each fragment


### Rendering a picture

1. We input the vertices to the stream
2. We process them and project them from a given scene camera postion and orientation from 2D to 3D on the screen.
3. Next you gorup into triangles
4. Now for every triangle you must compute what pixels are actually covered by it. This is rasterization.
5. For every pixel you compute it's color by simulating how light bounces 
6. Next put color on the closest fragment to the camera in the output image

This is the graphics pipeline.

The hardest part is computing the color. It is very complex to compute what a pixel actually looks like because material properties are so different

### Early grpahics programming (OpenGL API)

* Graphics programming API's provided the programmer with mechanisms to set parameters of scene lights and materials.
* They realized that there are too many materials and no one simple analytical solution to them so to allow people to create any material they wanted they would leave it up to the graphics programmer to write the material themselves


### Graphics shading languages

* They allow the application to extend the functionality of the graphics pipelline by specifying materials and lilghts programatically
    * Support diversity in the materials
    * Support diversity in lighting conditions

* Programmer provides mini-programs that define pipeline logic for certain stages
    * Pipeline executes shader function for all inputs 


### Why do Gpus have so many cores

* Many simd multi threaded cores provide efficient execution of vertex and fragment kernels

### Observation circa 2001-2003

GPU's are very fast processors for performing the same computation in parallel on large collections of data and they were much cheaper than the parallel cpus


### Early GPU based scientific computation


Set graphics pipeline output image size to be output arraysize

Render two triangles that exactly cover the screen

We can now use the gpu like a data parallel programming system

Fragment shader is mapped over the whole pixel element collection

## GPU compute mode

NVIDIA came up with a way to do general purpose compute for the gpu


### Review: how to run code on a cpu

* Os loads program binary into memory
* OS selects CPU execution context that the main thread will be assigned to
* OS interrupts processor, prepares execution context 
    * Sets registers
    * program counter
    * etc
* Runs!
* Processors begins executin instructions within the environment maintained in the execution context

### NVIDIA tesla architecture 2007

First alternative, non graphics-specifc interface to GPU hardware

Let's say a user wants to run a non-graphics program on the GPU's programmable cores
* Application can allocate buffers in GPU memory and copy to and from the buffers
* Applications via a graphics driver provides GPU a single kernel program binary
* APllication tell GPU to run kernel in SPMD fashion

Aside: this is actually way simpler than drawPrmitiuves()

## CUDA programming language

* very simple programming languages
* Like ISPC but even simpler
* Like OpenCL but cuda is proprrietary
    * CUDA only runs on nvidia gpus
    * Opencl runs on many vendors
    * Cuda is pretty much equal to opencl
    * cuda is better documented and easier to use

--- 
The plan:

1. CUDA programming abstractions
2. Cuda implementation
3. More detail on architecture


Things to consider:
* Is cuda a data parallel programming model
* Is cuda an example of the shared address space model
* Or the message passing model
* Can you find analogies between cuda concepts and ispc instances and tasks. What about c++ threads and pthreads

---

### Clarification

* Cuda thread is a similar abstraction to a pthread which corresponds to logical threads of control but the implementation of a cuda thread is very very different
* We will discuss differneces at the end of the lecture


### Basic spmd programming

* programmers authors one program
* Executes function multiple times, multiple instances of the function is run
    * Behavior of each instance depends on instance ID 

### CUDA programs are SPMD programs

Each program instance is a "CUDA thread"

Cuda threads are organized as a hierarchy: grouped into "thread blocks"

Thread IDs can be up to 3 dimensional

#### Host code: is the serial execution that is running as part of a normal c/c++ application on the GPU

NUmber of spmd threads is explicit in the program

Number of kernel invocations is not determined by size of data collection

* a kernel launch is not map kernel collection as was the case with graphics shader programming


Also important to make sure that we put some kind of bounds to stop a program from going past the length of arrays.

### How to think about memory

There is memory for cpu and memory for gpu

C++ code can allocate memory in ram and cuda can allocate memory in the gpu

In cpp:
* You use new or malloc

In CUDA:
* You use cudaMalloc or cuda free 
* You use memcpy to copy between host and device address spaces

#### Several different types of memory in the GPU

* There is GPU memory, device global memory
* There is readable writeeable cuda threads block memory that is shared between them all
* And there is per thread private memory

This has very important implications to efficiency of gpu implementation of CUDA

##### This is the reason cuda groups threads into thread blocks, so that all threads within thread block can read and write the same memory 

The shared memory is kind of like a cache but with the difference that you as a programmer must memory manage the shared data.


But it is not a cache because you get to put whatever you want in it.

Results in far fewer load instructions

You also need to do a synch threads before continuing

You can also do atomic add etc etc like in ispc


## Summary: Cuda abstractions

* Execution: thread hieararchy
    * Bulk launch of many threads
    * Two level hieararchy: threads are grouped into thread groups

* Distributed address space 
    * Built in memcpy primitives to copy between host and device address space
    * Three different types of address spaces
    * Per thread, per block("shared"), per program ("global")
* Barrier synchronization primitive for threads in thread block
* Atomic primitives for additional synchronization (shared and global variabels)

### Assignment work 

Dynamic work allocation from pool of work

When you compile a cuda program it remembers the instructions but it also remembers the requires resources
* 128 threads per block
* B bytes of local data per thread
* 130 floats (520 bytes ) of shared space per thread block 
