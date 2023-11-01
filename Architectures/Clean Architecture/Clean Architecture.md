# Clean Architecture

# Table of Contents
1. [Clean Architecture](#clean_architecture)
    1. [Presentation Layer (User Interface)](#)
    1. [Domain Layer (Business Logic)](#layer2)
    1. [Data Layer (Data Access)](#layer3)
1. [Data Flow](#data_flow)
    1. [User → Data](#data_flow_1)
    1. [Data ← User](#data_flow_2)
1. [Dependency Direction](#dependency_direction)
1. [References](#references)

# Clean Architecture <a name="clean_architecture"></a>

Clean Architecture contains 3 main layers:
1. Presentation Layer (User Interface).
2. Domain Layer (Business Logic).
3. Data Layer (Data Access).

## Presentation Layer (User Interface) <a name="layer1"></a>

### Responsibility

Handles **interactions** between **the user** and **the app**.

### Implementation
- Design Paterns used by that layer:
    - MVC, MVP, MVVM, ... - MV*-patterns 
- Examples:
    - View models
    - Views (UIKit or SwiftUI)

## Domain Layer (Business Logic) <a name="layer2"></a>

### Responsibility

It contains:
- Business logic.
- Business rules:
    - Are described with **use cases**.
        - Use case defines the behavior of the app in terms of user inputs and matching outputs.
            - Example: what the app should do when user submits a login and a password.
- Entities.

### Implementation

The implementation should contain:
- **Entities**
    - Objects that encapsulate critical business rules and data.
    - Business rules are the most important part of this definition. 
        - A struct with several fields is not enough to make for an entity.
    - Some business concepts are naturally described by operations rather than data, which leads us to the notion of services.
- **Repository Interfaces / Interfaces**
    - They abstract operations that will be performed by data access layer.
- **Use Cases / Interactors / Services**
    - A **stateless object** with **operations** on data.
        - However, they can mutate global data, leading to side-effects.
    - Input and output: **entities** and **language primitives**.
    - Should interact with Repository Interfaces and Entities.
        - Repository Interfaces typically injected in a constructor.
    - Fulfill following operations:
        - Business logic.
        - CRUD (create, read, update, delete) on entities.
        - Networking.
        - Validations.

## Data Layer (Data Access) <a name="layer3"></a>

### Responsibility

Contains **communication** with external **data stores** such as databases and network.
    
### Implementation

- Design Paterns used by that layer:
    - **Gateway**
        - Encapsulates **access** to external system or resource.
        - It transforms complex API into a convenient one for your app to use.
        - Is client-oriented and not intended for a general-purpose use (Repository is more general-purpose).
    - **Repository**
        - Encapsulates:
            - **Object** contained in a data source (e.g. API or database).
            - **Operations** on those objects.
        - Is more general-purpose compared to Gateway.
        - Can provide authorization capabilities if any required by the underlying data source.
    - **Domain Transfer Object**
        - An object that contains data without any behavior.
        - Extensively used for communication with network.
            - For this purpose, an intermediate Codable structs are created which reflect the structure of request and response.
- Implementation:
    - Repositories Implementations + API (Network) + Persistence DB
        - Mapping Codable
        - User Defaults, Core Data
        - Cache, File System
        - REST API

# Data Flow <a name="data_flow"></a>

## User → Data <a name="data_flow_1"></a>

```
User Interface -> Business Logic -> Data Access
```

Process can start with a user interaction:
1. User Interface: A user interacting with the app (UI). 
2. Business Logic: Then the data goes to business logic layer, where it results in CRUD operations on entities. 
3. Data Access: Entities are saved to a backend or a database.


## Data ← User <a name="data_flow_2"></a>

```
Data Access -> Business Logic -> Presentation Layer
```
The reversed process is also the case:
1. Data Access: The data is fetched from the backend or the database.
2. Business Logic: The data is transformed by the business rules.
3. User Interface: The data is finally presented to the user.


# Dependency Direction <a name="dependency_direction"></a>

```
User Interface -> Business Logic <- Data Access
```

Lower-level layers, which are user interface and data access, should depend on the business logic layer.

# References <a name="references"></a>
- [Template iOS App using Clean Architecture and MVVM](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/tree/master)
- [Layered Architecture to Design iOS Apps](https://www.vadimbulavin.com/layered-architecture-ios/)
