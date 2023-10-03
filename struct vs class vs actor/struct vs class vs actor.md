Explore `struct` vs `class` vs `actor` differences in this in-depth article.

# Table of Contents

1. [`struct` vs `class` vs `actor` - Comparison](#comparison)
1. [Structs](#structs)
    1. [Use Structures When...](#use_structs)
1. [Classes](#classes)
    1. [Use Classes When...](#use_classes)
1. [Actors](#actors)
    1. [Use Actors When...](#use_actors)
    1. [Do Not Use Actors When...](#do_use_actors)
1. [References](#references)

# `struct` vs `class` vs `actor` - Comparison <a name="comparison"></a>

<table>
<thead>
  <tr>
    <th></th>
    <th><span style="font-weight:bold">Structs</span></th>
    <th><span style="font-weight:bold">Classes</span></th>
    <th><span style="font-weight:bold">Actors</span></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><span style="font-weight:bold">Type</span></td>
    <td><p style="text-align: center">Value type</p></td>
    <td><p style="text-align: center">Reference type</p></td>
    <td><p style="text-align: center">Reference type</p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">Inheritance</span></td>
    <td><p style="text-align: center">❌</p></td>
    <td><p style="text-align: center">✅</p></td>
    <td><p style="text-align: center">❌</p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">Thread−safety</span></td>
    <td><p style="text-align: center">✅</p></td>
    <td><p style="text-align: center">❌</p></td>
    <td><p style="text-align: center">✅</p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">Memberwise Initializer</span><br></td>
    <td><p style="text-align: center">✅</p></td>
    <td><p style="text-align: center">❌</p></td>
    <td><p style="text-align: center">❌</p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">Deinitializer</span></td>
    <td><p style="text-align: center">❌</p></td>
    <td><p style="text-align: center">✅</p></td>
    <td><p style="text-align: center">✅</p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">Memory allocation</span></td>
    <td><p style="text-align: center">Stack</p></td>
    <td><p style="text-align: center">Heap</p></td>
    <td><p style="text-align: center">Heap</p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">External access to public</span><br><span style="font-weight:bold">properties and methods</span></td>
    <td colspan="2"><div style="text-align: center">direct</div></td>
    <td><p style="text-align: center">direct, but by using `await`</p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">Protocols</span></td>
    <td></td>
    <td><p style="text-align: center"><ul style="display: inline-block; text-align: left"><li>conform to the `AnyObject`</li><li>can therefore conform to `Identifiable` without adding an explicit id property</li></ul></p></td>
    <td><p style="text-align: center"><ul style="display: inline-block; text-align: left"><li>conform to the `AnyObject`</li><li>can therefore conform to `Identifiable` without adding an explicit id property</li><li>conform to the `Actor`</li></ul></p></td>
  </tr>
  <tr>
    <td><span style="font-weight:bold">Method execution</span></td>
    <td colspan="2"><div style="text-align: center">Can potentially be executing severals methods at a time</div></td>
    <td><p style="text-align: center">Can execute only one method at a time</p></td>
  </tr>
  <tr>
    <td><p style="font-weight:bold">Other</span></td>
    <td colspan="3"><p style="text-align: center">Can:<br><ul style="display: inline-block; text-align: left"><li>have initializers</li><li>have properties</li><li>have methods</li><li>have subscripts</li><li>have extensions</li><li>conform to protocols</li></ul></p></td>
  </tr>
</tbody>
</table>

# Structs <a name="structs"></a>

- It's a value type (along with e.g. arrays, dictionaries, enums and tuples).
    - Every time you use the struct reference in your code, it is potentially a new struct.
        - But Swift copy that new struct to a new memory address only if you modify the struct.
        - It called *copy-on-write* pattern. It's a memory optimization.
- Structs cannot cause memory leaks.

# Use Structures When... <a name="use_structs"></a>

- Apple recommendation: use structures **by default** for storing data and modeling behavior.
- When you don't need class' features:
    - You don't need to **control the object's identity** (use the identity operator `===`).
    - You don't need a **shared mutable state feature**.
    - You don't need **Objective-C interoperability**.
    - You don't need **class inheritance**.
    - You need to **replace class inheritance**:
        - Protocols and structures can replace class inheritance.

# Classes <a name="classes"></a>

- It's a reference type.
- Classes can cause memory leaks.

## Use Classes When... <a name="use_classes"></a>

- You need to **control the object's identity** (use the identity operator `===`).
- You need a **shared mutable state feature**.
- You need **Objective-C interoperability**.
- You need **class inheritance**.

# Actors <a name="actors"></a>

- It's a reference type.
- Actors can cause memory leaks.

## Use Actors When... <a name="use_actors"></a>

- You need to use a reference type.
- You need **thread safety** for shared mutable state.
    - It eliminates all kinds of race conditions and data races.

## Do Not Use Actors When... <a name="do_use_actors"></a>

- For your SwiftUI data models:
    - Instead use a class that conforms to the `ObservableObject` protocol. 
        - If needed, you can optionally also mark that class with `@MainActor` to ensure it does any UI work safely, 
        - But keep in mind that using `@StateObject` / `@ObservedObject` automatically makes a view’s code run on the main actor. 
    - If you desperately need to be able to carve off some async work safely:
        - Create a sibling actor – a separate actor that does not use `@MainActor`, but does not directly update the UI.

# References <a name="references"></a>
- [Choosing Between Structures and Classes](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes#Use-Structures-and-Protocols-to-Model-Inheritance-and-Share-Behavior)
- [What’s the difference between actors, classes, and structs?](https://www.hackingwithswift.com/quick-start/concurrency/whats-the-difference-between-actors-classes-and-structs)
- [Interview Questions on Struct vs Class vs Actor | iOS Interview Questions | Swift 5.5](https://www.youtube.com/watch?v=4GJBxkkql6o)
