title: CSCI 3341 Operating Systems - Fall 2015 Notes
author: Cieara Meador


[TITLE]


2015.08.18
==========
Assignments:

1. thread
2. semaphore
3. pipe


2015.08.20
==========
An *operating system* is a collection of programs and data which manages computer resources and supports the virtual characteristics of the computer.

*Computer resource* - any hardware or software component of a computer.

### Managing computer resources
* Allocation / deallocation - resources allocated by OS as needed.
* Controlling - OS ensures resources cannot be misused or abused. Also makes sure resource works properly. (An example of abuse: using all memory. An example of misuse: using parts of memory not meant to use.)
* Facilitator - makes sure all needs of your program are provided.

*Virtual characteristics* - properties imagined to exist but which in reality do not. (Ex: natural languages.)

*Kernel* - nucleus portion of OS which must live in main memory while the machine is running.

OS always has empty space in its section of memory for loading subsystems when requested.

Kernel always occupies uppermost or lowermost part of main memory. Dedicated register knows where memory is available to user.

*System programs* - a collection of programs which provide convenience for the user. Tools for delivering OS services. Ex: command interpreter.
    * Application programs - word processors, DBMS, etc..
    * Program development aids - text editors, debug aids, compilers..
    * System utilities - copy, delete, sort, command interpreter

A *command interpreter* is either a collection of utilities, or a program with such a collection at its disposal.

### OS evolution
1. Sign-up scheme - only one person could use computer at a time. Work finished either before, exactly at end time, or slightly after end time is up. Memory dump to debug. Environment had to be set up each time.
2. Hiring operators - setup specialists to quicken environment setup. Reduced idle time. Time wasted shuffling software.
3. Batch processing - reduced time spend shuffling software. All programs needing a certain compiler were run at the same time. Owner of program has to note the needs of the program for the operator. Caused communication gap; attempted to be solved by job control language, or JCL. Invented by IBM Corp., first characters always "//" and last also "//".
4. JCL created
5. Automatic job sequencing - first OS created ("resident monitor")
    * Resident monitor consisted of four parts.
        1. Interrupt and trap vectors - arrays containing addresses to memory locations. INterrupt is a signal used by a resource to get the OS's attention. Every one has an interrupt ID containing the start address of the piece of code tat must be executed to handle the interrupt. Ex: interrupt has ID #5 →  go to vector index 5 →  vector contents at this index contain the memory address to code to handle. Interrupts come from the hardware; traps come from software. (Traps like exceptions, which can be handled within program by developer.)
        2. JCL interpreter
        3. Device driver
        4. Job sequencing

### Terminology
* *program* - written in a programming language. Static.
* *process* - a program as soon as it is in main memory. A candidate for execution by CPU. Dynamic
* *job* - general name for a program or a process.
* *one machine cycle*  - fetch, decode, execute. When a process is in execution, the CPU's registers contain data related to the process. This is the *process status*. A process can be frozen, and the status wl be moved to a location known by the OS. It can later be moved back to resume exactlwhere the process left off. 
* *another definition for OS* - event-driven software.


2015.08.25
==========
*Current status word* - contains contents of registers' contents of CPU while working on a process. Interrupt stops current CPU task, thus the name.

When an IRQ comes in, current status word moved to *old status word*.  

Example interrupt snippet:
``` 
...
while (halt flag not set during execution)
{
  IR = Memory(PC); PC++;
  execute(IR);
}
```  

Fetch, decode, execute. As long as halt flag is not set, process continues.  

Add "IRQ = false" at top. After every cycle, IRQ checked. If (IRQ), save all registers.  

Continues until routine for handling IRQ done. Copies old status word back to current status word.  

An interrupt can't interrupt an interrupt. Procedure will continue until it has finished.  

### OS Evolution
6. Time Sharing - after resident monitor, question was how to share time among several users. CPU time shared among processes.
  Historically, 
    a. How could CPU time be shared? Many algorithms, one called "round robin". CPU clock cycle time divided into a number of slices. 
      (*time slice* or *quantum*) If time slice < time needed, process halted and put in waiting state until its next slice. 
      If ==, process done. If time slice > time needed, CPU starts next time slice early instead of idling.
    b. Finished/waiting programs go to harddrive. Software needed to handle incoming/outgoing, makes sure which program next up to go to main memory. This is the *spooler*.
      (Simulatenous peripheral operational online something-or-other)
    c. I/O handled at smae time as CPU works. HASP is a well known spooler developed by NASA in Houston.  
      
  Real-time system - system provides response to user in predefined time. OS guarantees the output in this time. 
     - Hard real-time - guarantees the return in the time frame given. 
     - Soft real-time - no guarantee. OS does its best. 


From a modern perspective, systems are interactive; no need for spooler.
Multi-program system made possible by time-sharing. Also allows for multi-user systems.  

*Buffers* bridge HDD/main memory. Tries to overlap I/O process. Handles overlapping of CPU and I/O. One process (spooler overlaps several).  

7. PC Systems - didn't need to handle multi-user or multi-program situations, originally.

8. OS for coupled systems
  * A coupled system is a number of computers that work together.
  * A system is tightly coupled or loosely coupled. *Tightly coupled* means CPUs on same clock and bus. May share memory/hdd, other peripherals.
      *Loosely coupled* means distributed, no sharing among CPUs. Connected via networking to allow communication. Serve same purpose, belong to same organization. Allows for load balancing.
  * Multi-process or parallel systems work in one of two ways: *symmetric* or *asymmetric*.
      a. Symmetric - independent, each works on its own task.
      b. Asymmetric - one CPU is the master, it oversees other processors to ensure no idling, load balancing. Master/slave model.
      
9. Clustered systems - for example, three coupled systems, A B & C. When copuled again with each other, the result is a clustered system. Can also be symmetric or asymmetric.

10. Client-server systems
  * Differs from old centralized system because clients have *processing power* - not just terminals.
  * Two types: *file* or *computing* server.
  
11. Handheld systmes - two problems: memory capacity and display size.


2015.08.27
==========
*Device drivers* provide details/interface to I/O device. Details hidden from kernel I/O subsystem.

*Device controller* has a set of registers and a buffer; physically connects to I/O devices.

A read/write request from process is an error. Request produces a trap number, goes to interrupt/trap vector to handle it. 
Handling code is part of I/O routines of OS subsystem.

I/O routines talks to device driver, which puts request in a register in the device controller.

Device controller looks at contents of register, does transfer of data to/from buffer. 
(Note that device controller cannot interpret contents of its registers; each register has a predefined meaning.)

Two methodologies involving transfer to/from buffers: *asynchronous* or *synchronous*  

* Asynchronous (*interrupt-driven*) - IRQ from device controller signals when the data's ready. Interrupt handler 
  takes care of the rest. Until data is transferred from the buffer, the CPU is working on 
  something else.
* Synchronous (*programmed*) - *ready bit* in device controller is set to 1 as soon as buffer is ready. 
  CPU is constantly checking this bit and does the transfer as soon as it's == 1.
  Since the CPU is constantly checking, it isn't working on anything else - blocked.

Note that these two don't coexist on the same system.

File I/O error-based handling to ensure program doesn't handle it; OS assumes all control, 
prevents resource misuse.

### Improvement of I/O Ops
#### Memory-mapped I/O
Device controller register corresponds to section of main memory. Controller thinks it's writing to registers inside itself, but is in reality writing to main memory.
Facilitates CPU giving commands to the controller.

#### Direct memory access (DMA)
Tiny CPU in device controller. This CPU takes care of transferring buffer to main memory and informing CPU it's ready. No interruption of main CPU.

### Memory
*RAM* - volatile, access method is random, access time is fixed (regardless of current location and destination, time always fixed).

*ROM* - burned-in, manufacturer programmed. 

*PROM* - programmable by developer. One write.

*EPROM* - erasable PROM. Two types: electrical voltage and UV light.

*EEPROM* - electrical erasable programmmable read-only memory. Subset of EPROM.

*Cache memory* - very fast memory used to reduce access time. Small capacity. Reduces time 80% of the time.

*Hardware caching / pipelining* - overlap of machine cycles.

*Coherency & consistency* - two issues with cache memory. Usually multiple copies of data in cache memory - if one is outdated, consistency is gone and cache not coherent. 
*Coherency supports consistency*.

### Hardware
*Booting* - process of running bootstrap program, which initializes all necessary registers and memory needed for hardware to run. 
Also locates and loads OS kernel into main memory. An example bootstrapper: BIOS.

ROM chip holds BIOS, aka boot ROM.

(RAM is faster than ROM. Boot ROM contents often copied into RAM (*shadowing*) to make things go faster.)

Plug & ~~pray~~ play (PnP) allows automatic configuration of hardware, depends on BIOS' support.

2015.09.01
==========
Two purposes for OS, and main services provided by the OS: 

1. Support user convenience
2. Ensure system runs efficiently

### Convenience
* Program execution - compiler loads, runs, ends program.
* I/O operations - actual implementation of I/O operations.
* File manipulation - read/write, open/close, create/delete.
* Communication - exchange of information between processes. Two methods: 
  1. shared memory
  2. message passing (via OS kernel)
* Error detection - hardware and software error handling.

OS is dual mode: in *monitor mode*, OS handles instructions. Otherwise, *user mode*. 
Mode bit set by BIOS.

### Efficiency
* Resource allocation - general or specialized (ex. asking for additional memory beyond what has been allocated)
* Accounting - data collection for billing and configuration.
* Protection - OS protects processes from each other and system from intruders.
  - I/O protection - OS handles file operations, not program. (Priviliged vs unprivileged operations)
  - Memory protection - several special registers:
    + *Fence register* - keeps address of first (or last) byte of free memory after OS kernel. Error if a request is above (below).
    + *Lower/upper bound registers* - keep user memory areas safe from outside access. 
    + *Base/limit registesr* - relative. Base contains starting point, which a size is relative to (limit).
    + Changing any of the above is a privileged operation.
  - CPU protection - timer keeps track of when time slice is up. Basically a counter; with every tick of system clock, counter is decremented. 
    After == 0, next tick ends. Where n is counter size, 2^n^ ticks given. (clock cycles)
    Fixed or variable: fixed means manufacturer has set time, cannot be changed; variable means OS can change.

How to invoke OS services - two ways. System calls, system programs.

*System calls* - small programs providing interfaces between a process and the OS.

*System programs* - interface between user and OS.

System calls are error handling routines, inter- and intraprocess interrupts and traps. Addresses stored in interrupt/trap vector.

Process control system calls:  

  - end or abort process stops CPU. *Termination* means the process is completely out of the system, not just the CPU.
  - wait event - external element. Signal - no external impetus.

Other categories of system calls: file manipulation, device mgmt, info. maintenance, 
communications.

Parameter passing into OS three ways: registers, blocks of memory, or stack.

Tangent: compiler terminology.


2015.09.03
==========

OS structure
------------

### Simple structure
ex. writing code as you go (MS-DOS, original UNIX). Bunch of system calls. Divided into 
device control for disk, memory, monitor. Another group for memory management, etc. Two interfaces: kernel interface, 
system call interface. Also device drivers, some system programs.

### Layered structure
Entire functionality of OS divided up into layers of functionality. Hierarchical modules. Assumptions: 

* assume if working on a certain layer, all layers below are free of errors.
* assume that, at any level, the only functionality available is in layers below.

Ex: Windows NT, "THE"

Two issues with layered structure: 

* how to determine what goes in which layer? always a chance you need something in a layer above you
* performance issues from passing parameters up & down so many layers

Advantages of layered structure: modularity, error containment, easy to debug and expand.

### Combination structure
Simple & layered. Attempts ot get advantages from both and reduce disadvantages. Ex: OS/2, Win NT 4.0

### Virtual machine
Software providing illusion that every user has their own resources - CPU, memory, I/O devices. 
(Emerged as CMS in late 70s from IBM.)

Advantages: protection, a research vehicle for OS.

Disadvantages: lack of sharing (solution: minidisk shared by all VMs, networking of VMs)

#### JVM
The Java virtual machine (JVM) includes: class loader, class verifier, interpreter. Bytecode goes to class loader, 
which feeds it to the JVM. Security and reliability of CPU verified by class verified. If all ok, interpreter takes bytecode 
to machine code.

(A compiler takes an entire program to machine code. Interpreter goes line by line.)

JVM can work as an interpreter or a JIT compiler.

OS design & implementation
--------------------------
Hardware, desired OS, user goals, developer goals all influence design goals.

Differentiation between mechanism (how to build a feature) and policy (how to use it).

Assembly language or high-level language.

Processes
---------
A process is a program in execution. Active element (program is passive). Has values for registers, such as the PC. Has a process control block.

### States of a process
In its lifecycle, a process is in several states. (*Lifecycle* - moment created until moment terminated.)

* *New* - process goes to main memory in this state.
* *Ready* - process waiting for CPU execution.
* *Running* - process in execution by CPU.

Three outcomes from running state:

* process finished within time allotted -> goes to *terminated* state.
* process unfinished within time allotted -> goes *back to ready* state.
* process waiting for I/O -> sits in *waiting* state.
  - if the thing it needs arrives, it goes to *ready* state.
  - if the thing never arrives, it moves to *deadlock* state.
  
CPUs are *sequential* - parallel only when there are multiple CPUs in a system.

*Process control block* - piece of memory. Contains pointer space, process state, 
PID, program counter, registers, memory limits, list of open files, accounting info; there may 
be more depending on manufacturer. 

PCBs queued in linked list.


2015.09.10
==========

Queues
------
*Ready queue* - when programs loaded into main memory, they're aedded here. Procs waiting to get CPU attention.

*Job queue* - on hdd in non-interactive systems; nonexistent in interactive systems.

*Other queues* - I/O request queue (waiting procs).

Every device has its own I/O queue. 

If a proc's time slice has expired, it goes to a special queue for that purpose, if the ready queue is full. 

Wait-for queue holds processes waiting for an event. One for each event, one for waiting for a specific time.

Schedulers
----------
Something needed ot feed ready queue with procs from other queues. *Long-term scheduler* software does two things:

* creates job queues by spooling jobs based on priority. Building queues
* selects jobs from job queue, loads them to main memory

(No spooler = no long-term scheduler. So PCs and smartphones don't have.)

*Short-term scheduler* maintains ready queue, selects & feeds to CPU. Speed of this queue is higher; 
must work with CPU time slices. Frequency of selecting a job much higher.

Long-term scheduler also controls multiprogramming of system (at any time more than one program in main memory). 
Controls *degree of multiprogramming* - decides number of procs in main memory. 
*Balances type of loaded programs* - I/O-bound, CPU-bound.

If ready queue at capacity, partially-executed procs moved from there to HDD until ready queue has space. 
Scheduler for this is the *medium-term scheduler*. This is *swapping* - out and in. Uses current status word, 
old status word. Happens whether or not you have a long-term scheduler. 

Dispatcher
----------
*Dispatcher* is software that actually engages process with CPU. This is *context switching* - change from current 
status word to new status word.

How to reduce overhead over context switching? Use more than one set of registers for CPUs. Shortens time, but drawback 
is limited number of additional registers and chance you still need to shuffle things around.

Process creation & termination
------------------------------
A proc can create a new process (*child process*). Can execute two ways: sequential or parallel. Can share all, none, or some resources.

A proc is terminated when: it has finished successfully, or parent proc wishes it. (Because: no further use for child proc, or child proc overstepped its resource boundary.)

Cascading termination occurs when a proc is terminated.

Threads
-------
Procs P1 & P2. P1 shares resources with P2; P1 also executes within same address space as P2. 
Both P1 & P2 part of a bigger process and memory space belongs to this process. Then, these two procs are *threads*.

Same variables, same memory space = thread.

A group of threads handle one task. Peer threads share code, data, resources.

**Differences between threads & processes**

* Threads smaller than processes
* Threads have peers; procs don't
* Context switch is shorter for a thread
* Having a proc with multiple threads is more efficient than multiple procs (assuming the multi processes share same task)
* No protection needed in multithreaded environment because peer threads are dedicated to one task
* Threads allow sequential programs to be executed in parallel

**Similarities between threads & processes**

* Same set of states
* Thread may spawn children
* Thread within process executes sequentially
* Each thread has own stack and program counter (PC)

Two types: user threads, kernel threads. 

*User thread* created in user space, scheduled in user space, managed by user thread library. Kernel has nothing to do with it.

*Kernel thread* created, scheduled in kernel space, managed by kernel. 

User threads are faster than kernel threads in terms of creation and scheduling. Kernel faster in execution.

(Note: important to reference slides here for diagrams.)

### Multithreading models
*One-to-one* - parallel execution, but overhead of thread creation

*Many-to-one* - no parallelism, but less overhead

*Many-to-many* - some parallelism, reduced overhead

2015.09.15
==========

Threading issues
----------------

### *Fork* and *exec* sys calls

* *Fork* - thread/proc that issues fork command is duplicated. This *creates space*.
* *Exec* - cleans up content of duplicate created by *fork* and puts needed code segments in the space. *Provides code*.

No fork without exec.

### Cancellation
Cancellation is termination of a thread before it is finished. Two types:

* *asynchronous cancellation* - immediate cancellation of target thread. Sometimes causes problems: 
  updating data, reallocation of resources. For example, suppose a thread handles updates in a database; it is cancelled 
  before it can finish an update, so the database is left inconsistent and must be *rolled back*.
* *deferred (periodic) cancellation* - "savepoints" in code; at those points, thread checks for a cancellation 
  signal and cancels if needed. Some overhead cost, but avoids async's issues.

### Signal handling
Interrupt and trap signals. Resource issues signal; another receives and handles the signal. 
Receiving resources handles with either *user-defined handler* (in code) or *system handler* (default).
Some resources use both types.

Two types of signals, asynchronous and synchronous. If the sender and receiver resource is the same resource, the signal is *synchronous*.
Otherwise, *asynchronous*.

A signal can go to four places:

1. Thread issued to
2. All peer threads
3. Subset of threads
4. Specific, predefined thread

### Thread pool
*Thread pool* - limits must be placed on number of available threads in a multithreading system to avoid 
overuse of available resources. When a proc is made, a limitation is placed on the number of threads that proc can create. 
That number of empty threads is created, and assigned to the process. If the proc exceeds this number, new threads must wait 
until space is available. As soon as a thread is done, it goes back to the empty pool. 

Every thread has data that is unique to it - the thread ID.

### Lightweight process
*Lightweight process (LWP)* - sits at the border between user and kernel threads. Uses user thread libraries but is executed 
by kernel threads. Connects user threads to kernel threads. Each LWP has one or more user thread(s), one kernel thread for execution (many-to-one).

### Assignment #1
First assignment was given out - program with pthreads. 2 weeks to finish. Hardcopy of results to be delivered on due date. 

Demo day at end of semester, another hardcopy to be brought then to ensure no changes have been made. 