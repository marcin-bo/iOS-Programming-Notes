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

- **What is ARC?**
    - ARC is a **memory management feature of reference types**.
    - The goal of ARC is to **know which instances** (e.g. an instance of a class) **can be removed from memory**, to free up space.
    - ARC only applies to **reference types**.
- **What are benefits of using ARC?**
    - Helps to avoid memory leaks.
    - You can focus on writing code, not on the manual memory management.
- **When does ARC operate?**
    - ARC operates during **compile time**.
    - This contrasts with a **garbage collector**, which manages memory at **runtime**.
- **How does ARC work?**
    - Every object in Swift has a property called **the retain count**.
        - It represents the **number of owners** for a particular object.
        - When the retain count is greater than zero, the object is kept in memory.
        - When the retain count reaches zero, the object is removed from memory.

**Class vs Instance vs Object vs Reference**
- **Class**: structure, a "template" that is used to create objects; a blueprint for a house design.
- **Object**: all the houses built from that blueprint are objects of that class.
- **Instance**: a given house is an instance.
- **Reference**: an address of an instance.

## Retain Cycles <a name="retain_cycles"></a>

- A retain cycle is if two class instances hold a **strong reference to each other**, such that each instance keeps the other alive in memory.
- It is common issue:
    - When dealing with **parent-child relationships between classes** when two class instance properties hold a strong reference to each other. 
    - When you assign an **escaping closure** to a property of a class instance, and the body of that closure **captures `self`**.

## How To Avoid Retain Cycles <a name="avoid_retain_cycles"></a>

To avoid retain cycles:

1. In parent-child relationships between classes use:
- Weak references (`weak` keywords) 
    - Doesn't keep a strong hold on the instance if refers to.
    - Are optional and become `nil` when the instance that it refers to is deallocated.
- Unowned references (`unowned` keywords).
    - Doesn't keep a strong hold on the instance if refers to.
    - Are not optional and are always considered to have a non-`nil`.
    - The other instance has a longer life time, the instance cannot become `nil`.
    - Use an unowned reference only when you are sure that the reference always refers to an instance that has not been deallocated.
    - If you try to access the value of an unowned reference after that instance has been deallocated, you'll get a runtime error.

2. In closures:
- Use closure capture list.


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
- [Automatic Reference Counting (ARC) in Swift](https://www.appypie.com/automatic-reference-counting-arc-swift)
