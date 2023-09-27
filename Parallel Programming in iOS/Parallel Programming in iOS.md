# Parallel Programming in iOS

Explore Parallel Programming in iOS in this in-depth article.

# Table of Contents

1. [Concurrency](#concurrency)
    1. [Concurrency vs Parallelism](#concurrency_parallelism)
1. [Threading Terminology](#threading_terminology)
1. [Mechanisms for Concurrency](#mechanisms_for_concurrency)
    1. [Process](#process)
    1. [Run Loops](#run_loops)
    1. [Threads](#threads)
        1. [Moving Away from Threads](#threads_go_away)
    1. [Dispatch Queues](#dispatch_queues)
    1. [Operation Queues](#operation_queues)
1. [References](#references)

# Concurrency <a name="concurrency"></a>

**Concurrency** - the ability to execute more than one program/task simultaneously.

## Concurrency vs Parallelism <a name="concurrency_parallelism"></a>

Concurrent execution has 2 types: 
- **Concurrent, non-parallel** execution.
    - When we quickly switch from one task to another, so it seems that all the tasks run at the same time.
- **Concurrent, parallel** execution (also known as **Parallelism**).
    - When we literally run two or more tasks at the same time, e.g., on a multicore processor.

**Concurrency vs Parallelism**:
- Concurrency refers managing multiple threads of execution.
    - Concurrency is the broader term which can encompass parallelism.
- Parallelism is more specifically, multiple threads of execution executing simultaneously.

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

### Moving Away from Threads <a name="threads_go_away"></a>

- There are other solutions that move thread management code to the system level:
    - Dispatch Queues (GCD).
    - Operation Queues.
- With Queues all you have to do is:
    - Define the tasks you want to execute.
    - Add them to an appropriate dispatch queue.
        - The main decision you have to make is whether to do so synchronously or asynchronously.
- Queues advantages:
    - It eliminates lock-based code:
        - Instead of using a lock to protect a shared resource, you can instead create a queue to serialize the tasks that access that resource. 
    - Improving on loop code:
        - You can perform multiple iterations of the loop concurrently.
        - `dispatch_apply` or `DispatchQueue.concurrentPerform(iterations: 10) { /* ... */ }`.
    - Replacing thread joins.
        - Thread joins allow you to spawn one or more threads and then have the current thread wait until those threads are finished.
        - `DispatchGroup`.
    - Replacing semaphore code.
    - Replacing run-loop code.

## Dispatch Queues <a name="dispatch_queues"></a>

- iOS implementation of the Dispatch Queues is **Grand Central Dispatch** (GCD).
- Advantage: it's easier to think in Dispatch Queues than threads:
    - Instead of thinking in threads, you consider concurrent programming. as blocks of work pushed onto different queues.
- There are different types of queues: 
    - The main queue, which executes on the main thread and 
    - The custom queues, which does not execute on the background.
- There are different types of dispatch queues. 
    - Serial queues: tasks executed in the same order as it was added to the queue. 
    - Concurrent queues: tasks executed concurrently within this queue.

```swift
DispatchQueue.main.async {
    // execute async on main thread
}
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
