# Method Dispatch

Explore Method Dispatch in this in-depth article.

# Table of Contents

1. [What is Method Dispatch](#method_dispatch)
1. [Type 1: Static Dispatch (Direct Dispatch)](#direct_dispatch)
1. [Type 2: Dynamic Dispatch (Table Dispatch)](#dynamic_dispatch)
1. [Type 3: Message Dispatch](#message_dispatch)
1. [Method Dispatch Usage And Examples](#usage)
1. [Increasing Performance by Reducing Dynamic Dispatch](#increasing_performance)
1. [References](#references)

# What is Method Dispatch <a name="method_dispatch"></a>

Method Dispatch:
- It is a mechanism used in object-oriented languages.
- When **invoking a method** it is an **algorithm that selects which instructions (method) to execute**. 
- Tells your app where to find the method in memory before it gets executed.
- It's something that happens every time a method is called.

# Type 1: Static Dispatch (Direct Dispatch) <a name="direct_dispatch"></a>

- How it works when invoking a method:
    - The **compiler knows** the **memory address of the invoked method** at compile time.
    - The method is executed directly.
- Result of this mechanism:
    - The **fastest** method dispatch style.
    - Enables various compiler optimizations (e.g., devirtualization).
        - Devirtualization: A compilation phase where the compiler attempts to make functions static when applicable.
- Used by:
    - Value types.
        - Imposes restrictions on inheritance (the method cannot be overridden).
    - Extensions.
    - Reference types marked with the `final` keyword.
    - Methods and classes marked with `private` and `static`.

# Type 2: Dynamic Dispatch (Table Dispatch) <a name="dynamic_dispatch"></a>

- How it works when invoking a method:
    - The **compiler doesn't know** the **memory address of the invoked method** at compile time.
    - The **compiler creates** a table with function pointers for each class (known as the witness table or the **virtual table**).
    - At runtime, when the method is invoked, memory address of a method is **searched** in that table.
    - Then the method is **executed**. So the method is executed indirectly.
- Result of this mechanism:
    - **Slower** than the Static Dispatch.
    - This **prevent many compiler optimizations**, making the indirect call even more expensive.
- How the virtual table is constructed for a subclass:
    - A subclass copies the table with a function pointers from the parent class.
    - There are different function pointer for every method that the class has overridden.
    - New methods in the subclass are appended to the end of this array.
- Used by:
    - Reference types without `final` keyword.
    - Protocols.
    - Methods marked with `dynamic` keyword.

# Type 3: Message Dispatch <a name="message_dispatch"></a>

- Utilized in **Objective-C** to provide this mechanism.
- How it works when invoking a method:
    - Works similarly to Dynamic Dispatch.
    - Additionally, allows us to **change which method is called at runtime**.
- Result of this mechanism:
    - **The slowest** among all Dispatch Methods.
- Commonly used for:
    - Method Swizzling.
    - Functions marked with the `dynamic` keyword.
    - Functions marked with the `@objc` keyword.
- Facilitates altering the dispatch behavior at runtime.
 
# Method Dispatch Usage And Examples <a name="usage"></a>

 |                   | Initial Declaration | Extension |
 |-------------------|---------------------|-----------|
 | Value Type        | Static              | Static    |
 | Reference Type    | Table (even for `@objc`) | Static / Message (for `@objc`)    |
 | Protocols         | Table               | Static    |
 | NSObject subclass | Table               | Message   |

```swift
protocol Noisy {
    func makeNoise() -> Int // Protocol: Initial Declaration => TABLE
}
extension Noisy {
    func makeNoise() -> Int { return 0 } // TABLE (Inherited from the protocol (?))
    func isAnnoying() -> Bool { return true } // Protocol: Extension => STATIC
}
class Animal: Noisy {
    func makeNoise() -> Int { return 1 } // Reference Type => TABLE
    func isAnnoying() -> Bool { return false } // Reference Type => TABLE
    @objc func sleep() { } // Reference Type => TABLE
}
extension Animal {
    func eat() { } // STATIC
    @objc func getWild() { } // MESSAGE
}
```

# Increasing Performance by Reducing Dynamic Dispatch <a name="increasing_performance"></a>

1. Use the `final` keyword: the declaration cannot be overridden.

2. Use the `private` access modifier: 
- Restricts the visibility of the declaration to the current file.
- Enables the compiler to infer the final keyword.

3. Use *Whole Module Optimization*:
- All of the module is compiled together at the same time.
- This allows the compiler to infer `final` on declarations with `internal`.


# References <a name="references"></a>

- [Increasing Performance by Reducing Dynamic Dispatch](https://developer.apple.com/swift/blog/?id=27)
- [Method Dispatch in Swift | Static | Dynamic | Message ](https://www.youtube.com/watch?v=Qbam_n4-ebg)
- [Method Dispatch in Swift, and its effect on performance](https://medium.com/@venki0119/method-dispatch-in-swift-effects-of-it-on-performance-b5f120e497d3)
- [Method Dispatch in Swift](https://medium.com/@pallavidipke07/method-dispatch-in-swift-b113a40a713a)
- [Method dispatch in Swift](https://trinhngocthuyen.com/posts/tech/method-dispatch-in-swift/)
