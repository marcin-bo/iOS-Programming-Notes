# Methods in Swift

# Table Of Contents

1. [What's the Difference Between a Function and a Method?](#function_vs_method)
1. [Instance Methods and Type Methods](#arc)
    1. [Instance Methods](#instance_methods)
    1. [Type Methods](#type_methods)
1. [References](#references)

# What's the Difference Between a Function and a Method? <a name="function_vs_method"></a>

- A method is a **function associated with a type**.
- Every method is a function.

# Instance Methods and Type Methods <a name="function_vs_method"></a>

Let's consider the following example:

```swift
class Foo {
    init() {}
        
    func instanceMethod() {
        print("An instance method")
    }
    
    static func staticMethod() {
        print("A type method (static)")
    }
    
    class func classMethod() {
        print("A type method (class) that can be overridden in a subclass")
    }
}

let foo = Foo()
foo.instanceMethod()
Foo.staticMethod()
Foo.classMethod()
```

## Instance Methods <a name="instance_methods"></a>

- An instance method can be called on an instance.
- An instance method cannot be called on a type.

## Type Methods <a name="type_methods"></a>

- An type method can be called on a type.
- An type method cannot be called on an instance.
- Type methods can be defined using `static` and `class` keywords:
    - The `class` keyword is used in classes and actors when you want to override the method in a subclass.
    - The `static` keyword is used in enums and structs or in classes and actors when you do want to override the method in a subclass.

# References <a name="references"></a>
- [What Is the Difference Between Instance Methods and Type Methods in Swift](https://cocoacasts.com/swift-fundamentals-what-is-the-difference-between-instance-methods-and-type-methods-in-swift)
- [What Are Functions Types in Swift](https://cocoacasts.com/swift-fundamentals-what-are-functions-types-in-swift)
