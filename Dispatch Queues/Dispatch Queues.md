# Dispatch Queues

# Table of Contents

1. [Dispatch Queues](#dispatch_queues)
    1. [The Main Queue](#dispatch_main_queue)
    1. [Global Concurrent Queues](#global_concurrent_queues)
    1. [Custom Queues](#custom_queues)
1. [Examples](#examples)
    1. [Serial Queue Executing Task Asynchronously](#example_1)
    1. [Serial Queue Executing Task Synchronously](#example_2)
    1. [Concurrent Queue Executing Task Asynchronously](#example_3)
    1. [Concurrent Queue Executing Task Synchronously](#example_4)
1. [References](#references)

# Dispatch Queues <a name="dispatch_queues"></a>

- The Main Queue
- Global Concurrent Queues
- Custom Queues

# The Main Queue <a name="dispatch_main_queue"></a>

- System provided.
- Serial.
- Use Main Thread.
- UI is tied to the Main Thread, UI related operations must be performed on the Main Queue.
- Note: Main Queue is bound to Main Thread. Main Thread is NOT bound to Main Queue.

```swift
DispatchQueue.main.async {
    // Execute async on main thread
}
```

# Global Concurrent Queues <a name="global_concurrent_queues"></a>

- System provided.
- Concurent.
- Do not use Main Thread (with one exception).
    - Task may run in the main thread when you use sync in GCD.
- Priorities are decided through QoS:
    - User-interactive - UI update, animations.
    - User-initiated - immediate results, data required for seamless user experience.
    - Default - falls between user-initiated and utility.
    - Utility - long running tasks, user is aware of the progress (there is a progress bar visible to the user).
    - Background - not visible to user, user is not aware of the task (prefetching, backup).
    - Unspecified - the lowest priority.
    
```swift
DispatchQueue.global().async { }

DispatchQueue.global(qos: .userInteractive).async { }
```
    
# Custom Queues <a name="custom_queues"></a>

- Serial or concurrent.
- Parameters when creating a custom queue:
    - Attributes (a single attribute or an array):
        - `concurrent` - a concurrent queue, by default it;s a serial queue.
        - `initiallyInactive` - allows us to create inactive queues, we call to `activate()` to activate them.
    - Target queue
        - A queue that the custom queue will use behind the scenes.
        - The priority is inherited from its target queue.
        - By default default priority global queue is target queue.
    - Auto Release Frequency
        - `inherit` - inherit from target queue, default behaviour.
        - `workItem` - individual auto release pool.
        - `never` - never setup an individual auto release pool.

Initializers:
```swift
init("queueName")
init("queueName", attributes: ...)
init("queueName", qos: .., attributes: ..., autoReleaseFrequency: ..., target: ...)
```


```swift

// Serial queue
let serialQueue = DispatchQueue(label: "serialQueue")
serialQueue.async { /* ... */ }

// Concurrent queue
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)
concurrentQueue.async { /* ... */ }

// Concurrent queue, but sync execution
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

# Examples <a name="examples"></a>

## Serial Queue Executing Task Asynchronously <a name="example_1"></a>

**Example:**
```swift
import Foundation
import PlaygroundSupport

PlaygroundPage.current.needsIndefiniteExecution = true

var value: Int = 20
let serialQueue = DispatchQueue(label: "com.queue.Serial")

func doAsyncTaskInSerialQueue() {
        for i in 1...3 {
            serialQueue.async {
            if Thread.isMainThread{
                print("task running in main thread")
            } else {
                print("task running in other thread")
            }
            let imageURL = URL(string: "https://upload.wikimedia.org/wikipedia/commons/0/07/Huge_ball_at_Vilnius_center.jpg")!
            let _ = try! Data(contentsOf: imageURL)
            print("\(i) finished downloading")
        }
    }
}

doAsyncTaskInSerialQueue()

serialQueue.async {
    for i in 0...3 {
        value = i
        print("\(value) ✴️")
    }
}

print("Last line in playground 🎉")
```

**Output:**
```
task running in other thread
Last line in playground 🎉
1 finished downloading
task running in other thread
2 finished downloading
task running in other thread
3 finished downloading
0 ✴️
1 ✴️
2 ✴️
3 ✴️
```

## Serial Queue Executing Task Synchronously <a name="example_2"></a>

**Example:**
```swift
import Foundation
import PlaygroundSupport

PlaygroundPage.current.needsIndefiniteExecution = true

var value: Int = 20
let serialQueue = DispatchQueue(label: "com.queue.Serial")

func doSyncTaskInSerialQueue() {
        for i in 1...3 {
            serialQueue.sync {
            if Thread.isMainThread{
                print("task running in main thread")
            } else {
                print("task running in other thread")
            }
            let imageURL = URL(string: "https://upload.wikimedia.org/wikipedia/commons/0/07/Huge_ball_at_Vilnius_center.jpg")!
            let _ = try! Data(contentsOf: imageURL)
            print("\(i) finished downloading")
        }
    }
}

doSyncTaskInSerialQueue()

serialQueue.async {
    for i in 0...3 {
        value = i
        print("\(value) ✴️")
    }
}

print("Last line in playground 🎉")
```

**Output:**
```
task running in main thread
1 finished downloading
task running in main thread
2 finished downloading
task running in main thread
3 finished downloading
Last line in playground 🎉
0 ✴️
1 ✴️
2 ✴️
3 ✴️
```

**Discussion:**
- Task may run in the main thread when you use sync in GCD. 
- Since the main queue needs to wait until the dispatched block completes, the main thread will be available to process blocks from queues other than the main queue. 
- Therefore there is a chance of the code executing on the background queue may actually be executing on the main thread since it’s serial queue, all are executed in the order they are added (FIFO).

## Concurrent Queue Executing Task Asynchronously <a name="example_3"></a>

**Example:**
```swift
import Foundation
import PlaygroundSupport

PlaygroundPage.current.needsIndefiniteExecution = true

var value: Int = 20
let concurrentQueue = DispatchQueue(label: "com.queue.Concurrent", attributes: .concurrent)

func doAsyncTaskInConcurrentQueue() {
        for i in 1...3 {
            concurrentQueue.async {
            if Thread.isMainThread{
                print("task running in main thread")
            } else {
                print("task running in other thread")
            }
            let imageURL = URL(string: "https://upload.wikimedia.org/wikipedia/commons/0/07/Huge_ball_at_Vilnius_center.jpg")!
            let _ = try! Data(contentsOf: imageURL)
            print("\(i) finished downloading")
        }
    }
}

doAsyncTaskInConcurrentQueue()

concurrentQueue.async {
    for i in 0...3 {
        value = i
        print("\(value) ✴️")
    }
}

print("Last line in playground 🎉")
```

**Output:**
```
task running in other thread
task running in other thread
task running in other thread
Last line in playground 🎉
0 ✴️
1 ✴️
2 ✴️
3 ✴️
2 finished downloading
1 finished downloading
3 finished downloading
```

**Discussion:**
- As in concurrent queue, task are processed in the order they are added to queue but with different threads attached to the queue. 
- They are not supposed to finish the task in the order they are added to the queue. 
- Order of task differs each time as threads are handled and assigned by the system. 
- All tasks get executed in parallel.

## Concurrent Queue Executing Task Synchronously <a name="example_4"></a>

**Example:**
```swift
import Foundation
import PlaygroundSupport

PlaygroundPage.current.needsIndefiniteExecution = true

var value: Int = 20
let concurrentQueue = DispatchQueue(label: "com.queue.Concurrent", attributes: .concurrent)

func doSyncTaskInConcurrentQueueQueue() {
        for i in 1...3 {
            concurrentQueue.sync {
            if Thread.isMainThread{
                print("task running in main thread")
            } else {
                print("task running in other thread")
            }
            let imageURL = URL(string: "https://upload.wikimedia.org/wikipedia/commons/0/07/Huge_ball_at_Vilnius_center.jpg")!
            let _ = try! Data(contentsOf: imageURL)
            print("\(i) finished downloading")
        }
    }
}

doSyncTaskInConcurrentQueueQueue()

concurrentQueue.async {
    for i in 1...3 {
        value = i
        print("\(value) ✴️")
    }
}

print("Last line in playground 🎉")
```

**Output:**
```
task running in main thread
1 finished downloading
task running in main thread
2 finished downloading
task running in main thread
3 finished downloading
1 ✴️
Last line in playground 🎉
2 ✴️
3 ✴️
```

**Discussion:**
- Task may run in the main thread when you use sync in GCD. 
- Since the main queue needs to wait until the dispatched block completes, the main thread will be available to process blocks from queues other than the main queue.
- There is a chance of the code executing on the background queue may actually be executing on the main thread. 
- Since its concurrent queue, tasks may not finish in the order they are added to the queue. 

# References <a name="references"></a>

- [Mastering Concurrency in iOS - Part 2 (Dispatch Queues, Quality of Service, Attributes) - iCode](https://www.youtube.com/watch?v=yH0RBTdNi3U)
- [Parallel Programming with Swift — Part 1/4](https://medium.com/swift-india/parallel-programming-with-swift-part-1-4-df7caac564ae)
