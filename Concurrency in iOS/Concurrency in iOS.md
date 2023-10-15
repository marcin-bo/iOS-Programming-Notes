# Concurrency in iOS

Explore Concurrency in iOS in this in-depth article.

# Table of Contents

1. [Concurrency](#concurrency)
    1. [Concurrency vs Parallelism](#concurrency_parallelism)
1. [Threading Terminology](#threading_terminology)
1. [Mechanisms for Concurrency](#mechanisms_for_concurrency)
    1. [Process](#process)
    1. [Run Loops](#run_loops)
    1. [Threads](#threads)
        1. [The Main Thread](#main_thread)
        1. [Problem With Threads](#problem_with_threads)
        1. [Moving Away from Threads](#threads_go_away)
    1. [Dispatch Queues](#dispatch_queues)
        1. [The Main Queue](#dispatch_main_queue)
        1. [Serial Queues](#dispatch_serial_queues)
        1. [Concurrent Queues](#dispatch_concurrent_queues)
    1. [Operation Queues](#operation_queues)
1. [References](#references)

# Concurrency <a name="concurrency"></a>

**Concurrency** - the ability to execute more than one program/task simultaneously.

## Concurrency vs Parallelism <a name="concurrency_parallelism"></a>

Concurrent execution has 2 types: 
- **Concurrent, non-parallel** execution (also known as **Concurrency**).
    - A property of a program, is more about **software design**.
    - Achieved through interleaving operation / context switching.
    - Needs just one core.
    - Managing (start, run, and complete) multiple tasks in overlapping time periods, in no specific order and abstracted from hardware details.
    - When a **task** is **divided** into multiple parts and we quickly **switch** from one task/part to another, so it seems that all the tasks run at the same time, it produces illusion of parallelism.

- **Concurrent, parallel** execution (also known as **Parallelism**).
    - A property of a **machine**, is more about **hardware**.
    - Achieved through using multiple CPUs.
    - Needs at least 2 cores.
    - Multiple threads of execution executing simultaneously.
    - When a **task** is **divided** into multiple parts and we literally run two or more tasks/parts at the exactly same time, e.g., on a multicore processor.

<img src="images/concurrent concepts.jpg" width="500"/>

# Threading Terminology <a name="threading_terminology"></a>

- **Thread** - a **separate path of execution** for code.
- **Process** - a **running executable**, which can encompass multiple threads.
- **Task** - an **abstract concept of work** that needs to be performed.

# Mechanisms for Concurrency <a name="mechanisms_for_concurrency"></a>

iOS provides different tools to use Concurrency.

## Process <a name="process"></a>

- iOS does not support multiple processes for one app. You only have one process.
- macOS supports multiple processes for one app

## Run Loops <a name="run_loops"></a>

- **A Run Loop** is a mechanism that **allows threads** to **process events** at any time without exiting.
    - It is a loop tied to a single thread.
    - Example:
        - There is an incoming event on the thread.
        - The thread enters the run loop (it creates one if needed).
        - Thread uses the run loop tot run event handlers in response to incoming events.
- Implementations:
    - `CFRunLoopRef` 
        - Class from CoreFoundation framework.
        - Provides APIs for pure C functions, all of which are **thread-safe**.
    - `RunLoop` 
        - Wrapper based on `CFRunLoopRef` 
        - Provides **object-oriented APIs**, but these APIs are **not thread-safe**.
- A run loop receives events from two different types of sources:
    - Input sources: they deliver asynchronous events to your threads.
        - Port-Based Sources.
        - Custom Input Sources.
        - Cocoa Perform Selector Sources (`performSelectorOnMainThread`).
    - Timer sources: they deliver synchronous events, at a scheduled time or repeating interval.
        - `Timer` or `CFRunLoopTimer` classes.

## Threads <a name="threads"></a>

- **Thread** - a **separate path of execution** for code.
- Compared to processes, threads **share their memory** with their **parent process**.
- Threads are a limited resource on iOS - there are **64 threads** at the same time **for one process**.
- Disadvantage: **the thread management on the developers shoulders**:
    - Developer responsibilities - the burden of creating a scalable solution:
        - Creating threads.
        - Adjusting the number of threads dynamically as system conditions change.
    - Application responsibilities:
        - Most of the costs associated with creating and maintaining any threads it uses. 

### The Main Thread <a name="main_thread"></a>

- The initial thread – the one the app is first launched with.
– Always exists for the lifetime of the app.
- User interface work must take place on the main thread:
    - When you try to **update your UI from any other thread**:
        - Nothing happens.
        - The app crashes.
        - Or pretty much anywhere in between.

### Problem With Threads <a name="problem_with_threads"></a>

- Managing threads is **complex**.
- Each thread you create needs to **run somewhere**:
    - If you accidentally end up creating 40 threads when you have only 4 CPU cores, the system will need to spend a lot of time just swapping them.
- Swapping threads = **context switch**:
    - It has a **performance cost**: 
        - The app must stash away all the data the thread was using.
        - The app must remember how far it had progressed in its work, before giving another thread the chance to run. 
- When you create many more threads compared to the number of available CPU cores - **thread explosion**:
    – The cost of context switching grows high.

### Moving Away from Threads <a name="threads_go_away"></a>

- There are other solutions that move thread management code to the system level:
    - Dispatch Queues (GCD).
    - Operation Queues.
- With Queues all you have to do is:
    - Define the tasks you want to execute.
    - Add them to an appropriate dispatch queue.
        - The main decision you have to make is whether to do so synchronously or asynchronously.
- Queues advantages - queues are **easier to think about** than threads:
    - We **don't care how** some **code runs** on the CPU.
    - We **care about the order**: (serially) or not (concurrently). 
    - A lot of the time we don't even create the queue – we use **built-in queues**.

## Dispatch Queues <a name="dispatch_queues"></a>

- iOS implementation of the Dispatch Queues is **Grand Central Dispatch** (GCD).
- There are different types of queues: 
    - **The main queue**, which executes on the main thread and 
    - The **custom queues**, which execute on the background.
- Dispatch queue types. 
    - **Serial** queues: 
        - Tasks executed in the same **order** as it was added to the queue.
        - **One task at a time**:
            - The task must finish before starting new task in the queue (the risk of a deadlock).
    - **Concurrent** queues: 
        - Tasks executed **concurrently** within this queue in **random order**.
        - **Many tasks at a time**.
- Dispatch queue task can be run:
    - **Synchronously** - blocks the execution until the task is completed
    - **Asynchronously** - does not block the execution, returns immediately.

#### The Main Queue <a name="dispatch_main_queue"></a>

```swift
DispatchQueue.main.async {
    // execute async on main thread
}
```

#### Serial Queues <a name="dispatch_serial_queues"></a>

```swift
// Serial queue
let serialQueue = DispatchQueue(label: "serialQueue")
serialQueue.async { /* ... */ }
```

#### Concurrent Queues <a name="dispatch_concurrent_queues"></a>

```swift
// Example 1: Concurrent queue
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)
concurrentQueue.async { /* ... */ }

// Example 2: Concurrent queue, but sync execution
// Although it's a concurrent queue, we explicitly specify the sync execution
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)
concurrentQueue.sync { print("started 1"); print("finished 1") } // 1
concurrentQueue.sync { print("started 2"); print("finished 2") } // 2

// The output:
// started 1
// finished 1
// started 2
// finished 2
```

## Operation Queues <a name="operation_queues"></a>

- High-level abstraction of GCD is Operation Queues.
- Instead of a block of discrete units of work you create operations.
- Creating an operation can also be done in multiple ways:
    - By creating an `BlockOperation` or `NSInvocationOperation`
    - By subclassing `Operation`.
    
```swift
let operationQueue: OperationQueue = OperationQueue()
operationQueue.addOperations([operation1], waitUntilFinished: false)
operation2.addDependency(operation1) //execute operation1 before operation2
```

# References <a name="references"></a>
- [What is the difference between concurrency and parallelism?](https://stackoverflow.com/a/53215726/1136128)
- [Parallel programming with Swift: Basics](https://medium.com/@jan_olbrich/basics-of-parallel-programming-with-swift-93fee8425287)
- [Run Loops - Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)
- [About Threaded Programming - Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/AboutThreads/AboutThreads.html#//apple_ref/doc/uid/10000057i-CH6-SW2)
- [Migrating Away from Threads - Threading Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW1)
- [What is the difference between concurrency, parallelism and asynchronous methods? - Stack Overflow](https://stackoverflow.com/questions/4844637/what-is-the-difference-between-concurrency-parallelism-and-asynchronous-methods#comment5379841_4844774)
- [Concurrency vs Parallelism: 2 sides of same Coin?](https://www.linkedin.com/pulse/concurrency-vs-parallelism-2-sides-same-coin-khaja-shaik-/)
- [Understanding threads and queues](https://www.hackingwithswift.com/quick-start/concurrency/understanding-threads-and-queues)
