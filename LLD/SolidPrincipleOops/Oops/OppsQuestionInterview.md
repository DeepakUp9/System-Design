## Object-Oriented Programming (OOP) Fundamentals

* **Classes and Objects**: 
    * **Class**: A blueprint defining structure (attributes) and behavior (methods).
    * **Object**: An instance of a class that holds specific state and data.
    * **Benefits**: Enables modularity, reusability, and scalability in design.

## Constructors

* **Purpose**: Initializes an object to a valid state upon creation.
* **Types**:
    * **Default Constructor**: Provides initial default values.
    * **Parameterized Constructor**: Accepts arguments to initialize fields with custom data.
    * **Copy Constructor**: Creates a new object by copying another, emphasizing deep copies to avoid reference issues.
    * **Private Constructor**: Restricts direct instantiation, commonly used in design patterns like Singleton.
* **Key Concepts**: Constructors are not inherited, cannot be abstract/static/final, and do not return values.

## The 'this' Keyword

* **Purpose**: Refers to the current instance of the class.
* **Utility**: 
    * Resolves naming conflicts between instance variables and parameters.
    * Facilitates constructor chaining using `this()`.
    * Enables fluent interfaces by returning the current object reference.

## Polymorphism

* **Compile-time (Static)**: Method Overloading—same method name with different parameters (count or type) within the same class.
* **Runtime (Dynamic)**: Method Overriding—a child class provides a specific implementation for a method already defined in its parent class.

## Inheritance

* **Types**: Single, Multi-level, Hierarchical, and Multiple (via interfaces to avoid the Diamond Problem).
* **Core Value**: Promotes code reusability and establishes parent-child relationships.

## Encapsulation

* **Concept**: Data hiding by marking attributes as private and exposing them through public getters and setters.
* **Benefit**: Ensures data integrity and provides controlled access to object states.

## Abstraction

* **Abstract Class**: Defines a base template; can have both implemented methods and abstract (unimplemented) methods. Cannot be instantiated.
* **Interface**: Defines a contract for behavior. Focuses solely on what a class does, not how it does it. Supports multiple inheritance.

## Access Modifiers

* **Public**: Accessible from any class.
* **Private**: Restricted to the declaring class.
* **Protected**: Accessible within the same package and by subclasses.
* **Default**: Restricted to the same package.

## Class Relationships

* **Inheritance**: "Is-a" relationship.
* **Association**: "Has-a" relationship; general structural connection.
* **Aggregation**: Loose association where components exist independently of the container.
* **Composition**: Strong association; the child cannot exist without the parent.
* **Dependency**: Temporary "uses-a" relationship, typically within method scope.
* **Realization**: Implementing an interface contract.

## Generics and Wildcards

* **Generics**: Parameterizes types to ensure type safety and code reusability (e.g., `List<T>`).
* **Wildcards (?)**:
    * **Unbounded**: Any type (`?`).
    * **Upper Bound**: `? extends T` (Read-only operations).
    * **Lower Bound**: `? super T` (Write-heavy operations).
* **Best Practice**: Prefer Generics for type safety; use Wildcards only when necessary for flexibility.