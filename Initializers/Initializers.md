# Initializers in Swift

# Table Of Contents

1. [Memberwise Initializers in Structs](#memberwise_init_in_structs)
    1. [Memberwise Initializer](#memberwise_init)
    1. [When We Get a Memberwise Initializer?](#when_memberwise_init)
1. [Designated Initializers in Classes](#designated_init)
1. [Convenience Initializers in Classes](#convenience_init)
1. [Failable Initializers](#failable_init)
1. [Required Initializers in Classes](#required_init)
1. [References](#references)

# Memberwise Initializers in Structs <a name="memberwise_init_in_structs"></a>

## Memberwise Initializer <a name="memberwise_init"></a>

- An initializer generated automatically by a compiler for a struct.
- Access level is `internal`.
    - We can use memberwise initializers internally within the module in which their type is defined.

## When We Get a Memberwise Initializer? <a name="when_memberwise_init"></a>

We get a memberwise initializer when:
- All of its members are either:
    - Visible (`public` or `internal`),
    - Or computed,
    - Or wrapped by a wrapper that provides a default value (SwiftUI’s `State`).
- There is no custom initializer.
    - Workaround to have a custom initializer and memberwise initializer: add a custom initializer in the extension.

# Designated Initializers in Classes <a name="designated_init"></a>

- What is a designated initializer?
    - Fully initializes all properties introduced by that class.
    - Calls an appropriate superclass initializer.
- How to use a designated initializer?
    - Use the `init` keyword before the designated initializer.
- What a designated initializer can/cannot do?
    - Can call an initializer of the superclass.
    - Cannot call a convenience initializer of the superclass.

# Convenience Initializers in Classes <a name="convenience_init"></a>

- What is a convenience initializer?
    - A special, secondary initializer.
    - Make it easier to create instances of a type by providing alternative ways to set its initial state.
- How to use a convenience initializer?
    - Use the `convenience` keyword before the convenience initializer.
- What a convenience initializer can/cannot do?
    - Can call other initializers (designated or convenience) from the same class.
    - Cannot call the initializers of the superclass.

# Failable Initializers <a name="failable_init"></a>

- What is a failable initializer?
    - A special initializer that can return nil if the initialization process fails, allowing for the creation of optional instances.
- How to use a failable initializer?
    - Use the `init?` keyword before the failable initializer.

# Required Initializers in Classes <a name="required_init"></a>

- What is a required initializer?
    - A special initializer.
    - Must be implemented by any subclass, ensuring that certain properties are properly initialized in the inheritance chain.
    - We don't have to provide an explicit implementation of a required initializer if we can satisfy the requirement with an inherited initializer.
- How to use a required initializer?
    - Use the `required` keyword before the required initializer.

# References <a name="references"></a>
- [Initialization](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/initialization/#Memberwise-Initializers-for-Structure-Types)
- [What Is a Memberwise Initializer](https://cocoacasts.com/swift-fundamentals-what-is-a-memberwise-initializer)
- [When can a struct’s memberwise initializer be used?](https://www.swiftbysundell.com/tips/when-can-memberwise-initializers-be-used/)
