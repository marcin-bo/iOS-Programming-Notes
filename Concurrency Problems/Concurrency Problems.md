# Concurrency Problems

# Table of Contents

1. [Priority Inversion](#priority_inversion)
1. [Deadlock](#deadlock)
    1. [Examples](#deadlock_examples)
    1. [How To Prevent From Getting a Deadlock?](#deadlock_how_to_prevent)
1. [Livelock](#livelock)
1. [Race Condition](#race_condition)
    1. [Examples](#race_condition_example)
    1. [How To Prevent From Getting a Race Condition?](#race_condition_how_to_prevent)
1. [Data Races](#data_races)
    1. [Examples](#data_race_examples)
    1. [How To Prevent From Getting a Data Race?](#data_race_prevent)
1. [Thread Explosion, Excessive Thread Creation](#thread_creation)
1. [Thread Starvation](#thread_starvation)
1. [References](#references)

# Priority Inversion <a name="priority_inversion"></a>

**A priority inversion occurs when:**
- The higher priority tasks wait for the smaller priority tasks to be finished.

## Examples
- A lower priority thread is locking a resource.
- A thread with higher priority wants to access that resource.
- As a result the thread with higher priority has to wait.
- Can result in the higher priority thread starving to death, as it never gets executed (Thread Starvation).

# Deadlock <a name="deadlock"></a>

**A deadlock occurs when:**
- A thread or queue waits infinitely for resources already blocked by another thread or queue.

## Examples <a name="deadlock_examples"></a>

#### Example: Calling a `sync` On a Serial Queue From The Same Serial Queue

```swift
let serialQueue = DispatchQueue(label: "serialQueue")

serialQueue.sync { // Block 1 (sync)
    print("Hello 1")
    
    serialQueue.sync { // Block 2 (sync). Deadlock resulting in a crash.
        print("Hello 2")
    }
}
```

```swift
let serialQueue = DispatchQueue(label: "serialQueue")

serialQueue.async { // Block 1 (async)
    print("Hello 1")
    
    serialQueue.sync { // Block 2 (sync). Deadlock resulting in a crash.
        print("Hello 2")
    }
}
```

#### Example: Calling `sync` On The Main Queue

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // The Main 
        DispatchQueue.main.sync { /* */ } // Deadlock and crash.
    }
}
```

## How To Prevent From Getting a Deadlock? <a name="deadlock_how_to_prevent"></a>

How to prevent getting a deadlock?
- Use properly GCD queues:
    - Do not call a `sync` on a serial queue from the same serial queue.
        - Example: we cannot call `DispatchQueue.main.sync` on the Main Queue which a serial queue.
- Use actors for synchronous access to shared resources.
- `NSRecursiveLock`
 
# Livelock <a name="livelock"></a>

**A livelock occurs when:**
- Threads constantly change their state. 
- As a result the threads switch between theirs not making any progress.
- A livelock is similar to a deadlock.

Example:
- A real-world example of livelock occurs when two people meet in a narrow corridor, and each tries to be polite by moving aside to let the other pass, but they end up swaping from side to side without making any progress because they both repeatedly move the same way at the same time.

# Race Condition <a name="race_condition"></a>

**A race condition occurs when:**
- The result of the code depends on the order of the code parts execution.
    - The timing or order of events affects the correctness of a piece of code.
- A data race can cause a race condition, but not always.

## Examples <a name="race_condition_example"></a>

#### Example 1

We have two threads:
- One doing a calculation and storing the result in `x`. 
- The other one started later (maybe from a different thread, or e.g. user interaction) will print the result to the screen.

#### Example 2

```swift
func write(_ text: String) {
    let words = text.split(separator: " ")
    for word in words {
        title.append(String(word))
    }
}

write("Concurrency with Swift:") // Thread 1
write("What could possibly go wrong?") // Thread 2

// Output:
// "Concurrency with What could possibly Swift: go wrong?"
```

## How To Prevent From Getting a Race Condition? <a name="race_condition_how_to_prevent"></a>

- Leverage synchronization mechanisms in shared mutable state:
    - Atomics.
    - Locks (`NSLock`, `NSRecursiveLock`, `os_unfair_lock`, `Mutex`).
    - Serial dispatch queues
        - Perform operations on a serial queue asynchronously.

# Data Races <a name="data_races"></a>

**A data race occurs when:**
- **Two (or more) threads** concurrently **access** the same **data** without synchronization.
    - One of these accesses is **a write / mutation**.
    - Usually involves **shared mutable state**.
- A data race is a type of **race condition**.
- What can be a result of a data race:
    - Unpredictable behavior.
    - Memory corruption.
    - Flaky tests.
    - Wrong results.
        - A data race **can cause a race condition**, but not always.
    - A crash (`EXC_BAD_ACCESS`).

## Examples <a name="data_race_examples"></a>

Data race which results in race condition and can be detected by Thread Sanitizer:

```swift
private var name: String = ""

func updateName() {
    DispatchQueue.global().async {
        self.name.append("George Bush") // 1.
    }
    
    print(self.name) // 2.
}
```

1. The background thread is writing to the name.

2. The main thread is accessing the name => unpredictable behavior:
- It depends on whether the print statement or the write is executed first.

## How To Prevent From Getting a Data Race? <a name="data_race_prevent"></a>

- Leverage synchronization mechanisms in shared mutable state:
    - Atomics.
    - Locks (`NSLock`, `NSRecursiveLock`, `os_unfair_lock`, `Mutex`).
    - Serial dispatch queues
        - Perform operations on a serial queue asynchronously.
        
```swift
private let lockQueue = DispatchQueue(label: "serialQueue")
private var name: String = "George Bush"

func updateNameSync() {
    DispatchQueue.global().async {
        self.lockQueue.async {
            self.name.append("George Bush")
        }
    }

    // Executed on the Main Thread
    lockQueue.async {
        // Executed on the lock queue
        print(self.name)
    }
}

// Prints:
// George Bush
// George Bush
```

2. Use actors.

```swift
actor NameController {
    private(set) var name: String = "My name is: "
    
    func updateName(to name: String) {
        self.name = name
    }
}

func updateName() async {
    DispatchQueue.global(qos: .userInitiated).async {
        Task {
            await self.nameController.updateName(to: "George Bush")
        }
    }
    
    // Executed on the Main Thread
    print(await nameController.name)
}
```

However, data races can still occur when using Actors. The difference is that we no longer access the data while it's being modified. The race condition below is defined as: "which thread is going to be the first to start isolated access?"
 
```swift
queueOne.async {
    await feeder.chickenStartsEating()
}
queueTwo.async {
    print(await feeder.numberOfEatingChickens)
} 
```

# Thread Explosion, Excessive Thread Creation <a name="thread_creation"></a>

- Working with GCD may result in **Thread Explosion**.
- Thread Explosion can lead to **memory and performance issues**.
- Thread Explosion in GCD:
    - In some scenarios GCD is very eager to **create new threads** to handle work.
    - This can result in having **more threads** than **CPU cores**.
- Thread Explosion in GCD might occur when:
    - **Too many blocking tasks** are added to the global **concurrent** queue or private concurrent queues.
    - **Too many private concurrent queues** exist that all consume thread resources.

```swift
// Thread Explosion example 1: too many blocking tasks.

// The code below will spawn a total of 150 threads, causing thread explosion to occur

final class HeavyWork {
    static func dispatchGlobal(seconds: UInt32) {
        DispatchQueue.global(qos: .background).async {
            sleep(seconds)
        }
    }
}

for _ in 1...150 {
    HeavyWork.dispatchGlobal(seconds: 3)
}
```

```swift
// Thread Explosion example 2: creating a lot of concurrent queues, one thread for each queue

let concurrentQueue1 = DispatchQueue(label: "concurrentQueue1", attributes: .concurrent)
concurrentQueue1.async { /* Perform a task */ }

let concurrentQueue2 = DispatchQueue(label: "concurrentQueue2", attributes: .concurrent)
concurrentQueue2.async { /* Perform a task */ }

/* ... */
```

## How To Prevent From Excessive Thread Creation? <a name="thread_creation_prevent"></a>

You can make use of the **global concurrent queue**:
```swift
DispatchQueue.global().async {
    /// Concurrently execute a task using the global concurrent queue.
    /// Also known as the background queue.
}
```
However, adding hundreds of tasks to global concurrent queue might result in Thread Explosion.

# Thread Starvation <a name="thread_starvation"></a>

**A thread starvation occurs when:**
- One process never gets the chance to run.

## Example

- We add only a few tasks to a low priority thread or queue.
- We add a lot of tasks to a high priority thread or queue.
- The low priority thread/queue will starve to death, as it will get next to no execution time. The result is, the task will not be executed or takes a long time.

# References <a name="references"></a>
- [A crash course of async await (Swift Concurrency) - Shai Mishali - Swift Heroes 2022](https://www.youtube.com/watch?v=uWqy5KZXSlA)
- [Data Race vs Race Condition in Swift](https://byby.dev/data-race-vs-race-condition)
- [Thread Sanitizer explained: Data Races in Swift](https://www.avanderlee.com/swift/thread-sanitizer-data-races/)
- [Parallel Programming with Swift: What could possibly go wrong?](https://medium.com/@jan_olbrich/parallel-programming-with-swift-what-could-possibly-go-wrong-f5bcc38b1814)
- [Concurrent vs Serial DispatchQueue: Concurrency in Swift explained](https://www.avanderlee.com/swift/concurrent-serial-dispatchqueue/)
- [How Does Swift Concurrency Prevent Thread Explosions?](https://swiftsenpai.com/swift/swift-concurrency-prevent-thread-explosion/)
- [What's the difference between deadlock and livelock?](https://stackoverflow.com/a/6155978/1136128)
