# Memory Management in Swift

# Table Of Contents

1. [Memory Allocation Types](#memory_allocation)
    1. [Stack: Static Memory Allocation](#stack)
    1. [Heap: Dynamic Memory Allocation](#heap)
    1. [Tricky Examples](#memory_allocation_examples)
        1. [Struct Contains a Property of a Class Type](#struct_with_class)
        1. [Class Contains a Property of a Struct](#class_with_struct)
        1. [Closure Captures a Value Type](#closure_with_value_type)
1. [ARC - Automatic Reference Counting](#arc)
    1. [What Is ARC?](#what_is_arc)
    1. [How Does ARC Work?](#arc_works)
    1. [Retain Cycles](#retain_cycles)
    1. [How To Avoid Retain Cycles](#avoid_retain_cycles)
1. [Method Dispatch](#method_dispatch)
1. [Copy-on-write Pattern](#copy_on_write)
1. [References](#references)

# Memory Allocation Types <a name="memory_allocation"></a>

## Stack: Static Memory Allocation <a name="stack"></a>

- The stack is **static** memory allocation.
- Objects allocated on the stack have their memory allocated at **compile time**.
    - Amount of memory required can be calculated at compile time.
- The stack deals with **value types**:
    - **Basic value types**: structs, arrays, dictionaries, enums, tuples, strings, ints, bools, ...
    - **Exception 1: a value type as a class property**: 
        - If a value type is a property of a class, then it's stored on the heap.
    - **Exception 2: a value type captured by a closure**: 
        - If a value type is captured in a closure, then that value will be copied to the heap so that it's still available by the time the closure is executed.
- The stack is efficient for inserting and removing items.
    - Time complexity of `O(1)` for insertion, deletion, and peak operations.
    - The cost of allocation and deallocation is low.
- The stack is **LIFO** (Last In First Out) data structure.
- **Each thread has** its own **stack**.
    - This means objects within it are **thread-safe**.
        - No other thread can access that stack, ensuring the safety of value types within its enclosure context.
    - Global access is not possible.

## Heap: Dynamic Memory Allocation <a name="heap"></a>

- The heap is **dynamic** memory allocation.
- Objects allocated on the heap have their memory allocated at **run time**.
- The heap deals with **reference types**:
    - Classes, actors and closures are stored in the heap.
- Swift utilizes **ARC** to automatically clean up memory and remove unused objects.
- Disadvantages of using the heap:
    - **More complex** data structure than a stack and slower than a stack.
        - **Time complexity**: Insertion and deletion are in `O(log n)`, and peak (min/max) is `O(1)` if you're using a min/max heap.
    - Classes can cause **memory leaks**.
    - **Thread unsafety**: The heap is global memory.
        - This can lead to thread unsafety when multiple threads access heap objects simultaneously.

## Tricky Examples - Value Types in The Heap <a name="memory_allocation_examples"></a>

- Swift guarantee that:
    - The value type will always have the value type features
    - The reference type will always have reference type features

- Swift does not guarantee that:
    - The exact place in memory (heap vs stack).

### Struct Contains a Property of a Class Type <a name="struct_with_class"></a>

- The struct will be stored in the **stack**.
- The class property will be stored in the **heap**

### Class Contains a Property of a Struct <a name="class_with_struct"></a>

- The struct will be stored in the **heap** with all the other fields of the class.

### Closure Captures a Value Type <a name="closure_with_value_type"></a>

- The closure with all captured value types will be stored in the **heap**.

# ARC - Automatic Reference Counting <a name="arc"></a>

## What Is Automatic Reference Counting (ARC)? <a name="what_is_arc"></a>

- Swift uses ARC for **memory management of reference types**.
  - Memory management prevents retain cycles and memory leaks.
- ARC operates during **compile time**.
  - This contrasts with a **garbage collector**, which manages memory at **runtime**.
- It counts the number of instances to reference types.
- This tally helps determine when it's safe to deallocate a class instance.
- In specific situations, ARC requires additional information to do its job, like the `weak` or `unowned` keywords.

## How Does ARC Work? <a name="arc_works"></a>

1. Allocation of Memory
- When you create a new class instance, ARC sets aside memory to store information about that instance. 
- This memory includes details about the instance's type and its stored properties.

2. Freeing Up Memory: 
- When an instance is no longer needed, ARC releases the memory it occupied. 
- This ensures that unused instances don't clog up memory space.

3. Avoiding Crashes: 
- If ARC were to free up memory while an instance is still being used, it could lead to crashes. 
- So, ARC keeps track of how many references are pointing to each instance

4. Strong References: 
- When you assign an instance to a property, constant, or variable, it creates a strong reference. 
- This reference ensures that the instance isn't deallocated as long as at least one strong reference to it exists.

## Retain Cycles <a name="retain_cycles"></a>

- A retain cycle is if two class instances hold a strong reference to each other, such that each instance keeps the other alive.
- It is common issue when dealing with parent-child relationships between classes.

## How To Avoid Retain Cycles <a name="avoid_retain_cycles"></a>

To avoid retain cycles use:
- Weak references (`weak` keywords).
- Unowned references (`unowned` keywords).

# Method Dispatch <a name="method_dispatch"></a>

Method Dispatch is an algorithm that finds the invoked method in memory before it gets executed. there are three types of Method Dispatch:
- Static Dispatch (Direct Dispatch).
- Dynamic Dispatch (Table Dispatch).
- Message Dispatch.

There are few ways to reduce Dynamic Dispatch (using `final` and `private` keywords, enabling *Whole Module Optimization*).

More about Method Dispatch you can find <a href="../Method Dispatch in Swift/Method Dispatch in Swift.md">here</a>.

# Copy-on-write Pattern <a name="copy_on_write"></a>

- The Copy-on-write Pattern is used with value types (e.g. structs, arrays, dictionaries, enums and tuples).
- Every time you use the value type in your code, it is potentially a new value type in memory.
    - Swift copy that new value type to a new memory address only if you modify it.
    - It called *copy-on-write* pattern. It's a memory optimization.

# References <a name="references"></a>

- [Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)
- [Structs, Classes, and Actors in iOS Interviews](https://holyswift.app/structs-classes-and-actors-in-ios-interviews/)
- [Swift stack and heap understanding - Stackoverflow](https://stackoverflow.com/a/42453109/1136128)
