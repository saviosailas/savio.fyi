# 2. Object-Oriented & Protocol-Oriented Programming

## Q1: What are the four pillars of OOP?

1. **Encapsulation** — Bundling data and methods, controlling access via `private`, `fileprivate`, `internal`, `public`, `open`.
2. **Abstraction** — Hiding complexity, exposing only necessary interfaces.
3. **Inheritance** — A class can inherit properties/methods from a parent class.
4. **Polymorphism** — Objects of different types can be treated through a common interface.

```swift
class Animal {
    func speak() { print("...") }
}

class Dog: Animal {
    override func speak() { print("Woof") }
}

class Cat: Animal {
    override func speak() { print("Meow") }
}

let animals: [Animal] = [Dog(), Cat()]
animals.forEach { $0.speak() } // Polymorphism
```

---

## Q2: What is Protocol-Oriented Programming (POP)?

Swift favors POP over classical OOP. Protocols define a blueprint of methods/properties, and types conform to them.

```swift
protocol Drawable {
    func draw()
}

protocol Resizable {
    func resize(to scale: CGFloat)
}

// Protocol composition
typealias Widget = Drawable & Resizable

// Default implementation via extension
extension Drawable {
    func draw() {
        print("Default drawing")
    }
}

struct Circle: Drawable {
    // Uses default draw() or provides its own
}
```

**Why POP over OOP?**
- Structs and enums can conform to protocols (no inheritance needed).
- Multiple protocol conformance (no diamond problem).
- Protocol extensions provide default implementations.
- Better testability via protocol-based dependency injection.

---

## Q3: What is the difference between `class` and `struct`?

| Feature | Struct | Class |
|---|---|---|
| Type | Value | Reference |
| Inheritance | No | Yes |
| Deinitializer | No | Yes (`deinit`) |
| Memberwise init | Auto-generated | Must write manually |
| Mutability | `mutating` keyword needed | Mutable by default |
| Identity check | Not applicable | `===` operator |

---

## Q4: What are Access Control levels?

From most restrictive to least:

1. `private` — accessible only within the enclosing declaration.
2. `fileprivate` — accessible within the same source file.
3. `internal` — accessible within the same module (default).
4. `public` — accessible from other modules, cannot be subclassed/overridden.
5. `open` — accessible from other modules, can be subclassed/overridden.

---

## Q5: What is the difference between Protocol and Abstract Class?

Swift has no abstract classes. Use protocols instead.

```swift
// Protocol approach (preferred in Swift)
protocol DataService {
    func fetchData() async throws -> Data
    var baseURL: URL { get }
}

extension DataService {
    var baseURL: URL {
        URL(string: "https://api.example.com")!
    }
}

// If you truly need shared state + abstract methods, use a base class:
class BaseViewController: UIViewController {
    func configure() {
        fatalError("Subclasses must override")
    }
}
```

---

## Q6: Explain `final` keyword

`final` prevents a class, method, or property from being overridden or subclassed.

```swift
final class APIClient {
    // Cannot be subclassed
}

class Base {
    final func criticalMethod() {
        // Cannot be overridden
    }
}
```

Benefits: compiler optimization (static dispatch instead of dynamic dispatch).

---

## Q7: What are Protocol Extensions and their limitations?

```swift
protocol Identifiable {
    var id: String { get }
    func describe() -> String
}

extension Identifiable {
    func describe() -> String {
        "ID: \(id)"
    }
}

struct User: Identifiable {
    let id: String
    // Gets describe() for free
}
```

**Limitation — Static dispatch trap:**
```swift
extension Identifiable {
    func greet() -> String { "Hello from protocol" }
}

struct Admin: Identifiable {
    let id: String
    func greet() -> String { "Hello from Admin" }
}

let admin = Admin(id: "1")
admin.greet()                        // "Hello from Admin"
(admin as Identifiable).greet()      // "Hello from protocol" ⚠️
```

Methods declared only in the extension (not in the protocol) use static dispatch.

---

## Q8: What is `@objc` and when do you need it?

`@objc` exposes Swift code to the Objective-C runtime.

Required for:
- Selectors (`#selector`)
- `@IBAction`, `@IBOutlet`
- Optional protocol methods
- KVO observation
- Any Objective-C interop

```swift
@objc protocol TableDelegate {
    func didSelectRow(at index: Int)
    @objc optional func didDeselectRow(at index: Int)
}
```
