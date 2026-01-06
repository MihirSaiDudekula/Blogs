---
title: "CPU Cores, Threads, and Execution: Trying to make sense of it all"
seoTitle: "Understanding CPU Cores and Threads"
seoDescription: "Learn about CPU cores, threads, concurrency, parallelism, and performance optimization through hardware and software distinctions"
datePublished: Tue Jan 06 2026 17:48:39 GMT+0000 (Coordinated Universal Time)
cuid: cmk2vvcfd000002l53xze8rzh
slug: cpu-cores-threads-and-execution-trying-to-make-sense-of-it-all
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/JMwCe3w7qKk/upload/9dd2deb0ad45c90d54c7baed9607c2ca.jpeg
tags: operating-system, os, process, cpu, threads

---

---

## Introduction: Why This Topic Is More Confusing Than It Should Be

If you ask a room full of CS students how CPUs *actually* execute programs, you will often hear answers like:

* “More threads means more speed.”
    
* “Threads are the same as cores”
    
* “Threads are a hardware concept.”
    
* “There is no difference in hardware and software threads”
    

These aren’t fully correct—and that confusion is not your fault. The modern execution stack spans **hardware, operating systems, and language runtimes**, each solving a *different problem*. When these layers are explained without a unifying mental model, concepts like *cores*, *threads*, and *parallelism* start to blur together.

This article rebuilds that mental model from the ground up.

We will move carefully from **processors and cores**, to **hardware threads**, to **software threads**, and finally to the **separation-of-concerns principle** that explains *why so many layers exist at all*. Along the way, we will draw a sharp line between **concurrency** and **parallelism**.

Let’s begin where execution truly starts—inside the processor.

---

## 1\. The Processor and Its Cores

### What Is a Processor?

A **processor (CPU)** is the **physical chip** installed on your motherboard. It is the unit that fetches, decodes, and executes machine instructions.

Modern processors rarely contain just one execution engine. Instead, they contain **multiple cores**.

### What Is a Core?

A **core** is an **independent execution engine inside a CPU**. It is capable of executing instructions on its own.

A useful analogy is a factory:

* **Processor** → the factory building
    
* **Core** → a fully equipped worker inside the factory
    

Each core typically has:

* Its own instruction pipeline
    
* Its own execution units (ALU, FPU, load/store)
    
* Its own registers
    
* Often its own L1 cache
    

Because of this independence:

> **Multiple cores provide true parallel execution.**

### Example

* 1 core → 1 instruction stream at a time
    
* 4 cores → 4 instruction streams at the same time
    

This is real, physical parallelism.

---

## 2\. What Are Threads?

A **thread** is a **sequence of execution** with:

* Its own stack and registers
    
* A shared address space with other threads in the same process
    

Threads exist to allow programs to *do multiple things at once*. However, not all threads are created at the same layer.

There are **two fundamentally different kinds of threads**:

1. **Hardware threads** (created by the CPU)
    
2. **Software threads** (created by the OS or runtime)
    

Understanding which layer creates which thread is essential.

---

## 3\. Concurrency vs Parallelism

These terms are often used interchangeably, but they are not the same.

### Concurrency

* Structuring a program to handle multiple activities
    
* A *design concept*
    
* Can exist on a single core
    

### Parallelism

* Executing multiple instruction streams simultaneously
    
* A *hardware property*
    
* Requires multiple cores
    

### Key Relationship

* **Concurrency** is expressed using software threads
    
* **Parallelism** is achieved using multiple cores
    

You can have concurrency without parallelism, but not the other way around.

---

## 4\. Hardware Threads (SMT / Hyper-Threading)

### What SMT Actually Means

Simultaneous Multithreading (SMT), known commercially as **Hyper-Threading** on Intel CPUs, allows **one physical core** to maintain **multiple hardware execution contexts**.

Each hardware thread has:

* Its own program counter
    
* Its own registers
    

But crucially, hardware threads **share**:

* Execution units
    
* Pipelines
    
* Caches
    
* Memory bandwidth
    

### Example

A CPU with:

* 4 physical cores
    
* 2 SMT threads per core
    

Will appear to the OS as:

```plaintext
8 CPUs
```

But the physical reality is:

```plaintext
4 execution engines
Shared hardware per core
```

### How SMT Actually Works (At the Hardware Level)

To understand this, forget “threads” as jobs or tasks.  
A CPU core does **not** see jobs, processes, or functions.

> A core only sees **instruction streams**: sequences of machine instructions waiting to be executed.

---

### What a Modern Core Looks Like Internally

A modern CPU core is not a single execution engine. It contains:

* Multiple execution units (ALUs, FP units, load/store units)
    
* Deep pipelines
    
* Large instruction windows
    
* Complex scheduling logic
    

However, **a single instruction stream cannot always keep all of this hardware busy**.

Why?

* Cache misses stall execution
    
* Branch mispredictions flush pipelines
    
* Some instructions depend on previous results
    
* Not all execution units are needed every cycle
    

This means:

> Large parts of the core sit idle during many clock cycles.

---

## What SMT Adds

SMT allows **multiple instruction streams** (hardware threads) to be present **inside the same core at the same time**.

Each hardware thread has:

* Its own architectural state (registers, program counter)
    
* Its own instruction stream
    

But they **share**:

* Execution units
    
* Caches
    
* Pipeline
    
* Core resources
    

---

### What Happens Each Cycle

Every clock cycle, the core:

1. Looks at **multiple instruction streams**
    
2. Selects instructions that are:
    
    * Ready to execute
        
    * Not blocked by dependencies
        
3. Issues them to available execution units
    

If one thread is stalled (for example, waiting on memory),  
another thread’s instructions can **use the idle hardware**.

This is the key idea

### Key Insight

> **More threads ≠ more cores**

SMT improves *throughput*, not *raw parallel compute*.

---

## 5\. Software Threads

### What Is a Software Thread?

A **software thread** is a logical unit of execution created by:

* The operating system (e.g., pthreads)
    
* A language runtime (e.g., Java threads, Go goroutines)
    

Examples include:

* Java `Thread`
    
* POSIX `pthread`
    
* Go goroutines (lighter-weight)
    

Software threads:

* Express concurrency in programs
    
* Are scheduled by the OS
    
* Are **not** tied directly to cores
    

They represent *what* your program wants to do concurrently—not *how* the hardware executes it.

---

## 6\. The Mental Model That Changes Everything

### The Core Insight

Here is the key idea that resolves most confusion:

> **This system is not about dividing work—it is about separation of concerns.**

Each layer exists to solve a different problem.

### The Processing Hierarchy

| Layer | Purpose |
| --- | --- |
| Software thread | Express concurrency |
| OS scheduler | Decide *when* it runs |
| Hardware thread | Provide execution context |
| Core | Execute instructions |

No layer duplicates the responsibility of another.

### End-to-End Example

**Machine:**

* 1 processor
    
* 2 cores
    
* 2 SMT threads per core
    

OS sees:

```plaintext
4 CPUs
```

**Program:**

* Creates 8 software threads
    

**What Happens:**

* 4 software threads run immediately
    
* 4 wait in the run queue
    
* OS time-slices and switches threads
    
* SMT hides short CPU stalls
    
* Cores never “divide” tasks
    

This is coordination—not division.

---

## 7\. User-Level Threads vs Kernel-Level Threads

Threads exist at **two different layers** in a system:

* **User level** – managed by language runtimes or thread libraries
    
* **Kernel level** – managed directly by the operating system
    

This distinction strongly affects performance, parallelism, and how blocking operations behave.

---

## User-Level Threads

User-level threads are managed entirely in **user space**. The OS is unaware of individual threads and sees only **one process or one kernel thread**, even if many user threads exist inside it.

These threads are scheduled by a **user-level thread library or runtime**, not by the kernel.

### How They Work:

User Thread 1  
User Thread 2  
User Thread 3  
     ↓  
Single OS thread

* The application creates multiple user threads.
    
* A **user-level scheduler** decides which user thread runs.
    
* Context switches happen **without entering the kernel**.
    
* The OS scheduler only schedules the **single underlying OS thread**.
    

### Advantages:

* **Very fast creation**  
    Thread creation does not require system calls.
    
* **Fast context switching**  
    Switching threads happens in user space, avoiding kernel overhead.
    
* **No kernel involvement**  
    Makes threads lightweight and portable across operating systems.
    

### Critical Limitations:

* **No true parallelism**  
    Since the OS schedules only one kernel thread, all user threads share a single CPU core—even on multicore systems.
    
* **Blocking system calls block all threads**  
    If one user thread performs a blocking I/O operation, the entire process stops, freezing all other user threads.
    
* **No kernel-level preemption**  
    The kernel cannot interrupt a misbehaving user thread; the runtime must manage fairness.
    

### Key Insight:

**User-level threads provide concurrency, not parallelism.**

They allow multiple tasks to *appear* to run at the same time, but they cannot execute simultaneously on multiple CPU cores.

---

## Kernel-Level Threads

Kernel-level threads are known to and scheduled by the **OS kernel**.  
Each thread is treated as an independent **schedulable entity**.

The kernel manages:

* Thread creation
    
* Scheduling
    
* Preemption
    
* Context switching
    

### Examples:

* POSIX threads (pthreads)
    
* Java threads (modern JVM)
    
* Windows threads
    

### How They Work:

Thread A → Core 1  
Thread B → Core 2  
Thread C → Core 3

* Each thread maps to a kernel thread.
    
* The OS can schedule threads **independently**.
    
* Threads can run simultaneously on **different CPU cores**.
    

### Advantages:

* **True parallel execution**  
    Multiple threads can execute at the same time on multicore processors.
    
* **Blocking affects only one thread**  
    If one thread blocks on I/O, other threads continue running.
    
* **OS ensures fairness and preemption**  
    The kernel can interrupt threads to enforce time-sharing and responsiveness.
    

### Disadvantages:

* **Higher overhead**  
    Thread creation and management require system calls.
    
* **Kernel involvement during context switches**  
    Switching threads is slower compared to user-level threads.
    

---

## Mapping Models (Important for Understanding)

The relationship between user threads and kernel threads is defined by **thread mapping models**:

| Model | Meaning |
| --- | --- |
| **1:1** | One user thread maps to one kernel thread |
| **N:1** | Many user threads map to one kernel thread |
| **M:N** | Many user threads map to many kernel threads |

### Modern Practice:

* **Modern systems predominantly use a 1:1 model**
    
* This model balances simplicity, true parallelism, and OS-level scheduling support.
    
* Examples include Linux pthreads and modern JVM implementations.
    

---

### Summary Insight

* **User-level threads** → lightweight, fast, but limited
    
* **Kernel-level threads** → powerful, parallel, but heavier
    
* The choice affects scalability, performance, and responsiveness in real systems
    

If you want, I can also:

* Add **comparison tables**
    
* Provide **exam-ready short answers**
    
* Explain **M:N model with examples**
    
* Convert this into **lecture notes or slides**
    

---

## 8\. Why OS Multithreading Does NOT Make Your App Parallel

### The Confusing Statement

> “Even if your application is single-threaded, the OS is multi-threaded.”

This is true—but often misunderstood.

### What Actually Runs in Parallel

On a multicore system, at the same time:

* Your application thread
    
* OS scheduler threads
    
* Disk I/O threads
    
* Network stack threads
    
* Device drivers
    

These threads run **alongside** your program—not *inside* it.

### Why This Does Not Help Your Program

Consider:

```java
for (int i = 0; i < N; i++) {
    compute(i);
}
```

* One instruction stream
    
* One core executes it
    

Even on an 8-core CPU:

* Your program uses 1 core
    
* The OS uses others
    

### Why the OS Cannot Parallelize It for You

Because:

* Program order matters
    
* Data dependencies exist
    
* The OS cannot safely infer parallelism
    

Automatic parallelization is extremely hard and unsafe in the general case.

**Analogy:**

* OS = traffic police
    
* Your program = one car
    
* More roads do nothing unless you add more cars
    

---

## Conclusion: The One Mental Model to Keep

If you remember only one thing from this article, make it this:

> **Software threads express concurrency. Hardware threads improve utilization. Cores execute instructions.**

The apparent complexity of modern CPUs is not accidental. It is the result of **clean separation of concerns**, allowing software, operating systems, and hardware to evolve independently—each solving a different problem well.

Once you internalize this hierarchy, topics like SMT, scheduling, and parallel performance stop feeling mysterious and start feeling inevitable.

And that is the point where systems programming truly begins.