# Concurrency in iOS

Explore Concurrency in iOS in this in-depth article.

# Table of Contents

1. [Concurrency](#concurrency)
    1. [Concurrency vs Parallelism](#concurrency_parallelism)
1. [Threading Terminology](#threading_terminology)
    1. [Task](#task)
    1. [Process](#process)
    1. [Thread](#thread)
    1. [The Main Thread](#main_thread)
    1. [Run Loop](#run_loop)
    1. [Time Slicing](#time_slicing)
    1. [Context Switching](#context_switching)
1. [Mechanisms for Concurrency In Swift](#mechanisms_for_concurrency)
    1. [Manual Thread Creation](#manual_threads)
    1. [Grand Central Dispatch](#gcd)
    1. [Operation Queues](#operation_queues)
    1. [Modern Swift Concurrency](#swift_concurrency)
1. [References](#references)

# Concurrency <a name="concurrency"></a>

**Concurrency**:
- The ability to execute multiple at the same time (concurrently). 
- It enables responsiveness of system resources and better performance.

## Concurrency vs Parallelism <a name="concurrency_parallelism"></a>

Concurrent execution has 2 types: 
- **Concurrent, non-parallel** execution (also known as **Concurrency**).
    - A property of a program, is more about **software design**.
    - Achieved through interleaving operation / context switching.
    - Needs just one core.
    - It's all about **managing multiple tasks** (start, run, and complete):
        - At the same time (in overlapping time periods).
        - In no specific order.
        - Abstracted from hardware details.
    - When a **task** is **divided** into multiple parts and we quickly **switch** from one task/part to another, so it seems that all the tasks run at the same time, it produces illusion of parallelism.

- **Concurrent, parallel** execution (also known as **Parallelism**).
    - A property of a **machine** (multiple CPUs), is more about **hardware**.
    - Achieved through using **multiple CPUs**.
        - Needs at least 2 cores.
    - It's all about **executing multiple tasks**:
        - At the same time.
        - By multiple threads.
    - When a **task** is **divided** into multiple parts and we literally run two or more tasks/parts at the exactly same time, e.g., on a multicore processor.

<img src="images/concurrent concepts.jpg" width="500"/>

# Threading Terminology <a name="threading_terminology"></a>

## Task <a name="task"></a>

- **Task** - an **abstract concept of work** that needs to be performed.

## Process <a name="process"></a>

- **Process** - a **running executable**, which can encompass multiple threads.
    - iOS does not support multiple processes for one app. You only have one process.
    - macOS supports multiple processes for one app.

## Thread <a name="thread"></a>

- **Thread** - a **separate path of execution** for code.
- Compared to processes, threads **share their memory** with their **parent process**.
- Threads are a limited resource on iOS - there are **64 threads** at the same time **for one process**.

## The Main Thread <a name="main_thread"></a>

- The initial thread – the one the app is first launched with.
- Always exists for the lifetime of the app.
- User interface work must take place on the main thread:
    - When you try to **update your UI from any other thread**:
        - Nothing happens.
        - The app crashes.
        - Or pretty much anywhere in between.

## Run Loop <a name="run_loop"></a>

- **Run Loop** is a mechanism that **allows threads** to **process events** at any time without exiting.
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

## Time Slicing <a name="time_slicing"></a>

- **Time Slicing** - is the process of allocating time to threads by the CPU.

## Context Switching <a name="context_switching"></a>

- **Context Switching** - is the process of:
    - Storing the state of a process or thread, so that it can be restored and resume execution at a later point.
    - Then restoring a different, previously saved, state.

# Mechanisms for Concurrency In Swift <a name="mechanisms_for_concurrency"></a>

Swift provides different tools to use Concurrency:
1. Manual thread creation.
2. Grand Central Dispatch (GCD).
3. Operation Queues.
4. Modern Swift Concurrency.

## Manual Thread Creation <a name="manual_threads"></a>

- Swift provide `Thread` class to run the code in its own thread of execution.
    - You can start, sleep, terminate created thread, check its state.
- Disadvantage: **the manual thread management is complex**:
    - Developer responsibilities - the burden of creating a scalable solution:
        - Select the optimal number of threads for an application.
        - Keep the optimal number of threads based on current system load and the underlying hardware.
        - Synchronize threads without losing performance and application correctness.
    - Application responsibilities:
        - Most of the costs associated with creating and maintaining any threads it uses.
    - Each thread you create needs to **run somewhere**:
        - If you accidentally end up creating 40 threads when you have only 4 CPU cores, the system will need to spend a lot of time just swapping them.
    - Risks:
        - **Thread Explosion** - when you create many more threads compared to the number of available CPU cores.
- There are other solutions that move thread management code to the system level.

## Grand Central Dispatch <a name="gcd"></a>

**Grand Central Dispatch (GCD)**:
- It's a low-level API, built on top of threads, for **managing concurrency** in Swift using the concept of dispatch queues.
- It offers **easier concurrency model** than the manual thread management.
    - It gives you automatic thread pool management (creating threads, scheduling, optimizing their usage).
    - You **don't care how** some **code runs** on the CPU.
    - You only care about the task and how to execute it:
        - Defining the task you want to run.
        - Deciding **which queue to use** (The Main Queue, The Global Queue, Custom Queues).
        - Deciding on **the execution order** of a task: serially or concurrently.
        - Deciding on how to schedule the task: **synchronously or asynchronously**.
- It uses Dispatch Queues:
    - `DispatchQueue` class.
    - Dispatch queues are thread-safe (you can simultaneously access them from multiple threads).
    - Types:
        - The Main Queue (serial).
        - Global Queues (concurrent).
        - Custom Queues (serial and concurrent).
- Dispatch queue types - how many tasks are executed at a time and in what order?
    - **Serial** queues:
        - **How many tasks are executed at a time**?
            - Executing **one task at a time**:
                - The task must finish before starting new task in the queue (the risk of a deadlock).
        - **The order of execution**:
            - Tasks executed in the same **order** as it was added to the queue (FIFO).
        - Example: The Main Queue.
    - **Concurrent** queues: 
        - **How many tasks are executed at a time**?
            - Executing **multiple tasks at a time**.
        - **The order of execution**:
            - Tasks executed **concurrently** within this queue in **random order**.
        - Example: Global Concurrent Queues.
- Types of task execution in GCD (how can a task be dispatched to a queue):
    - **Synchronously** 
        - Blocks the current execution (the dispatching thread waits).
        - Waits until the task is completed.
        - Then it continues further.
    - **Asynchronously** 
        - Does not block the current execution (the dispatching thread continues execution).
        - Does not wait for the task to complete.
        - Returns immediately and continues further.
- **Task cancellation** in GCD:
    - It's not directly supported. 
    - However, you can check for a cancellation flag within the task code or using `DispatchWorkItem`'s `cancel()` method.

## Operation Queues <a name="operation_queues"></a>

- It's a high-level API, built on top of GCD, for **managing concurrency** in Swift using the concept of operation and operation queue.
    - Instead of a blocks (GCD), you work with operations.
    - You can control state, priority and dependencies of operations.
- Creating an operation can also be done in multiple ways:
    - By creating an `BlockOperation` or `NSInvocationOperation` (only in Objective C)
    - By subclassing an abstract class `Operation`.
- Danger of **memory leak**:
    - Operation queues retain operations until they're finished.
    - Operation queues are retained until all operations are finished. 
    - As a result, suspending an operation queue with operations that aren't finished can result in a memory leak.
- Operation Queues vs GCD:
    - Object-oriented API.
    - Dependencies support
        - However, GCD supports barrier flag which might result in similar behaviour).
    - Operations can be paused, resumed, and cancelled.
        - However, GCD support `DispatchWorkItem` can be cancelled.
    

### `Operation` characteristics
- There are **different states of an operation**, depending on its current execution status:
    - Ready: It's ready to execute
        - Operation become ready to execute when all of its dependent operations have finished executing.
    - Executing: The task is currently running.
    - Finished: Once the process is completed.
    - Cancelled: The task cancelled.
    - You can check the operation state: `isReady`, `isExecuting`, `isFinished`, `isCancelled`.
- `Operation` and `OperationQueue` classes have a number of properties that can be observed, using KVO.
- You can define operation dependencies.
- You can define operation `queuePriority` and `qualityOfService`:
    - If all of the queued operations have the same `queuePriority` and are ready to execute when they are put in the queue, they're executed in the order in which they were submitted to the queue. 
    - Otherwise, the operation queue always executes the one with the highest priority relative to the other ready operations.
- An operation can only execute once: you can't restart the same instance.
- **Operation cancellation**:
    - It's supported by calling the `cancel()` method on an Operation object. 
    - It's important to handle cancellation appropriately within the task code and check for the cancellation flag regularly.
    - Canceling an operation causes the operation to ignore any dependencies it may have. This behavior makes it possible for the queue to invoke the operation’s start() method as soon as possible. The start() method, in turn, moves the operation to the finished state so that it can be removed from the queue.
- Operation lifecycle in the operation queue:
    - The given task gets added to the `OperationQueue` that will start the execution as soon as possible. 
    - The `OperationQueue` will remove the task automatically from its queue once it becomes finished or cancelled.

### `OperationQueue` characteristics

- You can define `qualityOfService`.
- You can define `maxConcurrentOperationCount`.
- When adding an operation you can define synchronous/asynchronous execution using `waitUntilFinished`.
    - `operationQueue.addOperations([op], waitUntilFinished: false)`.
- You can `cancelAllOperations()`.
- You can `waitUntilAllOperationsAreFinished()`.

### Examples

```swift
// 1. Custom subclass of Operation:
final class ImportOperation: Operation {

    let repositoryId: String

    init(repositoryId: String) {
        self.repositoryId = repositoryId
        super.init()
    }

    override func main() {
        guard !isCancelled else {
            return
        }
        print("Importing repository..")
    }
}

// 2. BlockOperation
let operation1 = BlockOperation {
    print("BlockOperation")
}
// You can define completion block
operation1.completionBlock = {
    print("BlockOperation completed")
}

let operation2 = ImportOperation(repositoryId: "xyz")

let operationQueue: OperationQueue = OperationQueue()
operationQueue.maxConcurrentOperationCount = 2
operationQueue.addOperations([operation1], waitUntilFinished: false)
operation2.addDependency(operation1) // execute operation1 before operation2
```

## Modern Swift Concurrency <a name="swift_concurrency"></a>

Read more about Swift Concurrency <a href="../Swift Concurrency/Swift Concurrency.md">here</a>.

# References <a name="references"></a>
- [What is the difference between concurrency and parallelism?](https://stackoverflow.com/a/53215726/1136128)
- [Parallel programming with Swift: Basics](https://medium.com/@jan_olbrich/basics-of-parallel-programming-with-swift-93fee8425287)
- [Run Loops - Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)
- [About Threaded Programming - Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/AboutThreads/AboutThreads.html#//apple_ref/doc/uid/10000057i-CH6-SW2)
- [Migrating Away from Threads - Threading Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW1)
- [What is the difference between concurrency, parallelism and asynchronous methods? - Stack Overflow](https://stackoverflow.com/questions/4844637/what-is-the-difference-between-concurrency-parallelism-and-asynchronous-methods#comment5379841_4844774)
- [Concurrency vs Parallelism: 2 sides of same Coin?](https://www.linkedin.com/pulse/concurrency-vs-parallelism-2-sides-same-coin-khaja-shaik-/)
- [Understanding threads and queues](https://www.hackingwithswift.com/quick-start/concurrency/understanding-threads-and-queues)
- [Getting started with Operations and OperationQueues in Swift](https://www.avanderlee.com/swift/operations/)
- [Parallel Programming with Swift — Part 3/4](https://medium.com/swift-india/parallel-programming-with-swift-part-3-4-28a57584221c)
- [OperationQueue](https://developer.apple.com/documentation/foundation/operationqueue)
