# Advanced Grand Central Dispatch

# Table of Contents

1. [DispatchWorkItem](#dispatchworkitem)
1. [Dispatch Barriers](#dispatch_bariers)
1. [Dispatch Groups](#dispatch_groups)
    1. [Fetching and Updating Multiple Resources Using Closures In Parallel](#dispatch_groups_closures)
    1. [Fetching and Updating Multiple Resources Using Queues In Parallel](#dispatch_groups_queues)
1. [Dispatch Semaphore](#semaphore)
1. [References](#references)

# `DispatchWorkItem` <a name="dispatchworkitem"></a>

- `DispatchWorkItem` encapsulates block of code that can be dispatched to any queue.
- A dispatch work item has a cancel flag.
    - If it is cancelled before running, the dispatch queue won’t execute it and will skip it. 
    - If it is cancelled during its execution, the cancel property return true. 
        - In that case, we can abort the execution. Also work items can notify a queue when their task is completed.

```swift
let workItem = DispatchWorkItem {
    // Your async code goes in here
}

// Execute the work item after 1 second
DispatchQueue.main.asyncAfter(deadline: .now() + 1, execute: workItem)

/*
// Another way of executing a task on Custom Queue
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)
concurrentQueue.async(execute: task)
*/

// You can cancel the work item if you no longer need it
workItem.cancel()
```

```swift
var workItem: DispatchWorkItem = DispatchWorkItem {
    for i in 1..<6 {
        guard !workItem.isCancelled else {
            print("cancelled")
            break
        }
        sleep(1)
        print(String(i))
    }
}

workItem.notify(queue: .main) {
    print("done")
}

DispatchQueue.global().asyncAfter(deadline: .now() + .seconds(2)) {
    workItem.cancel()
}

DispatchQueue.main.async(execute: workItem)

// Output
// 1
// 2
// 3
// cancelled
// done
```

# Dispatch Barriers <a name="dispatch_bariers"></a>

- A dispatch barrier allows us to **create a synchronization point within a concurrent queue**. 
- A dispatch barrier **works only with custom concurrent queues**.
    - If the queue is a serial queue or one of the global concurrent queues, the barrier would not work.
- It **converts a concurrent queue into a serial queue**.
    - All items submitted to the queue before the dispatch barrier must complete. 
    - When the barrier is executing, the concurrent queue acts as a serial queue (it is the only one task being executed). 
    - After the barrier finishes, the queue returns to its default concurrent behavior.
- It can solve Readers-Writers Problem.
- Usage: `concurrentQueue.async(flags: .barrier)`

```swift
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)

concurrentQueue.async(flags: .barrier) { // Barrier
    for i in 0...3 {
        value = i
        print("\(value) ✴️")
    }
}

concurrentQueue.async {
    print(value)
}

concurrentQueue.async(flags: .barrier) { // Barrier
    for j in 4...6 {
        value = j
        print("\(value) ✡️")
    }
}

concurrentQueue.async {
    value = 14
    print(value)
}

// Output
0 ✴️
1 ✴️
2 ✴️
3 ✴️
3
4 ✡️
5 ✡️
6 ✡️
14
```

# Dispatch Groups <a name="dispatch_groups"></a>

- Dispatch Groups allows for aggregate tasks:
    - It can be used to **submit multiple different work items or blocks** (they might run on different queues).
    - And then track when they all complete.
- You can create groups of multiple tasks and wait for them to complete (using `group.wait()`) or receive a notification once they finish `group.notify(queue:)`.
- There are two ways to call completion block:
    - Using `group.wait()` and then manually execute completion block on a selected queue
    - Call `group.notify(queue:)`
- Use cases:
    - You need to run **two distinct network calls**. Only after they both have finished you have the necessary data to parse the responses.
    - Pull to refresh control and quick API call:
        - The API call returns so quickly that the refresh control doesn't seem to be working.
        - To solve this, we can add a small delay: we can wait for both some minimum time threshold, and the network call, before hiding the refresh control.


## Fetching and Updating Multiple Resources Using Closures In Parallel <a name="dispatch_groups_closures"></a>

Example - `DispatchGroup` + `enter()` + closures + `leave()` + `notify(queue:)`:

```swift
let group = DispatchGroup()
var result1: Int?
var result2: Int?
var result3: Int?

/* ... */

group.enter()
repository.fetch1 { fetchedResult1 in
    defer { group.leave() }
    result1 = fetchedResult1
}


group.enter()
repository.fetch2 { fetchedResult2 in
    defer { group.leave() }
    result2 = fetchedResult2
}


group.enter()
repository.fetch3 { fetchedResult3 in
    result3 = fetchedResult3
}

group.notify(queue: DispatchQueue.main) { [weak self] in
    // Do something with result1, result2, result3
}
```

## Fetching and Updating Multiple Resources Using Queues In Parallel <a name="dispatch_groups_queues"></a>

Example 1 - `DispatchGroup` + queues + `async(group:)` + `notify(queue:)`

```swift
// Create a group
let dispatchGroup = DispatchGroup()

// Create queues
let queue1 = DispatchQueue(label: "com.queue1")
let queue2 = DispatchQueue(label: "com.queue2")
let queue3 = DispatchQueue(label: "com.queue3")

// Put all queues into DispatchGroup
queue1.async(group: dispatchGroup) {
    print("Queue1 complete")
}

queue2.async(group: dispatchGroup) {
    print("Queue2 complete")
}

queue3.async(group: dispatchGroup) {
    print("Queue3 complete")
}

// After the queues in dispatch group are all completed, back to the main thread
dispatchGroup.notify(queue: DispatchQueue.main) {
    print("All tasks are completed")
}
```

Example 2 - download concurrently images and run code at the end:

```swift
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)

let group = DispatchGroup()

// Add concurrent tasks
for i in 1...10 {
    group.enter()
    
    concurrentQueue.async {
        defer { group.leave() }
        // Perform image download
    }
}

group.notify(queue: DispatchQueue.main) {
    print("All images are downloaded")
}

/* 
// Instead of group.notify() you can write below code:

group.wait()
DispatchQueue.main.async {
    print("All images are downloaded")
}
*/
```

# Dispatch Semaphore <a name="semaphore"></a>

- Semaphores give us the ability to **control access to a shared resource by multiple threads**.
- Real life example: 
    - A father sits with his three kids at home, then he pulls out an iPad.
    - The father is the semaphore, the iPad is the shared resource, and the kids are the threads.
- Code template:
    - Create concurrent queue 
    - `semaphore = DispatchSemaphore(value:)`
    - `semaphore.wait()`
    - Run the task
    - `semaphore.signal()`
- Best practises:
    - Never run semaphore `wait()` function on the Main Thread (as it will freeze your app).
    - Use `wait()` function allows us to specify a timeout.
        - Once timeout is reached, the wait will finish regardless of semaphore count value.

Example 
- Download 15 songs from a URL.
- We decided to download 3 songs at a time in order not to take too much CPU time at once.

```swift
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)

let semaphore = DispatchSemaphore(value: 3)
for i in 0 ..> 15 {
    concurrentQueue.async {
        let songNumber = i + 1
      
        semaphore.wait()

        print("Downloading song", songNumber)
        // Download ....
        print("Downloaded song", songNumber)

        semaphore.signal()
    }
}
```

# References <a name="references"></a>

- [Parallel Programming with Swift — Part 2/4](https://medium.com/swift-india/parallel-programming-with-swift-part-2-4-46a3c6262359)
- [The Beauty of Semaphores in Swift ](https://medium.com/@roykronenfeld/semaphores-in-swift-e296ea80f860)
- [Using DispatchWorkItem](https://www.swiftbysundell.com/tips/using-dispatchworkitem/)
