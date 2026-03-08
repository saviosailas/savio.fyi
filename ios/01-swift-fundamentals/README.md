# 1. Swift Language Fundamentals

## Q1: What is the difference between `let` and `var`?

- `let` declares a constant — value cannot be changed after assignment.
- `var` declares a variable — value can be reassigned.

```swift
let name = "Kiro"    // immutable
var count = 0        // mutable
count = 5            // OK
// name = "Other"    // Compile error
```

**Follow-up:** `let` on reference types (classes) means the reference is constant, not the object's properties.

---

## Q2: What are Value Types vs Reference Types?

| Value Types | Reference Types |
|---|---|
| `struct`, `enum`, `tuple` | `class`, `closure`, `function` |
| Copied on assignment | Shared reference on assignment |
| Stack allocated (usually) | Heap allocated |
| Thread-safe by default | Need synchronization |

```swift
struct Point {
    var x: Int
    var y: Int
}

var a = Point(x: 1, y: 2)
var b = a       // b is a copy
b.x = 10
print(a.x)     // 1 — unchanged

class Person {
    var name: String
    init(name: String) { self.name = name }
}

let p1 = Person(name: "Alice")
let p2 = p1     // p2 points to same object
p2.name = "Bob"
print(p1.name)  // "Bob" — changed
```

**When to use struct vs class?**
- Prefer `struct` by default (Apple's recommendation).
- Use `class` when you need identity, inheritance, or Objective-C interop.

---

## Q3: What are Optionals and how do you unwrap them?

Optionals represent a value that may or may not exist.

```swift
var name: String? = "Alice"

// 1. Optional Binding (if let / guard let)
if let unwrapped = name {
    print(unwrapped)
}

guard let unwrapped = name else { return }

// 2. Nil Coalescing
let displayName = name ?? "Unknown"

// 3. Optional Chaining
let count = name?.count  // Int?

// 4. Force Unwrap (avoid unless 100% certain)
let forced = name!

// 5. Implicitly Unwrapped Optional
var label: String! = "Hello"
```

**Interview tip:** Always explain why force unwrapping is dangerous and when `guard let` is preferred over `if let`.

---

## Q4: What is the difference between `==` and `===`?

- `==` checks value equality (calls `Equatable` conformance).
- `===` checks reference identity (same object in memory, classes only).

```swift
class Dog {
    var name: String
    init(name: String) { self.name = name }
}

extension Dog: Equatable {
    static func == (lhs: Dog, rhs: Dog) -> Bool {
        lhs.name == rhs.name
    }
}

let d1 = Dog(name: "Rex")
let d2 = Dog(name: "Rex")

d1 == d2   // true — same value
d1 === d2  // false — different objects
```

---

## Q5: Explain Closures in Swift

Closures are self-contained blocks of functionality. They capture and store references to variables and constants from the surrounding context.

```swift
// Basic closure
let greet: (String) -> String = { name in
    return "Hello, \(name)"
}

// Trailing closure syntax
let numbers = [3, 1, 4, 1, 5]
let sorted = numbers.sorted { $0 < $1 }

// Capturing values
func makeCounter() -> () -> Int {
    var count = 0
    return {
        count += 1
        return count
    }
}

let counter = makeCounter()
counter() // 1
counter() // 2
```

**Key concepts:**
- Closures are reference types.
- `@escaping` — closure outlives the function it was passed to.
- `@autoclosure` — wraps an expression in a closure automatically.
- `[weak self]` / `[unowned self]` — to avoid retain cycles.

---

## Q6: What are Generics?

Generics let you write flexible, reusable functions and types.

```swift
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

// Generic type with constraint
struct Stack<Element: Equatable> {
    private var items: [Element] = []

    mutating func push(_ item: Element) {
        items.append(item)
    }

    mutating func pop() -> Element? {
        items.popLast()
    }

    func contains(_ item: Element) -> Bool {
        items.contains(item)
    }
}
```

**Associated Types in Protocols:**
```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
}
```

---

## Q7: What is the difference between `Any`, `AnyObject`, and `any`/`some`?

- `Any` — can represent any type (value or reference).
- `AnyObject` — can represent any class instance only.
- `some` (opaque type) — hides the concrete type but the compiler knows it.
- `any` (existential type) — type-erased protocol container.

```swift
func makeShape() -> some Shape {
    Circle() // compiler knows it's Circle, caller sees "some Shape"
}

func acceptShape(_ shape: any Shape) {
    // type-erased, dynamic dispatch
}
```

---

## Q8: What are Property Wrappers?

Property wrappers add a layer of behavior to property access.

```swift
@propertyWrapper
struct Clamped {
    var wrappedValue: Int {
        didSet { wrappedValue = min(max(wrappedValue, range.lowerBound), range.upperBound) }
    }
    let range: ClosedRange<Int>

    init(wrappedValue: Int, _ range: ClosedRange<Int>) {
        self.range = range
        self.wrappedValue = min(max(wrappedValue, range.lowerBound), range.upperBound)
    }
}

struct Player {
    @Clamped(0...100) var health: Int = 100
}
```

Common built-in: `@State`, `@Binding`, `@Published`, `@AppStorage`.

---

## Q9: What are `map`, `flatMap`, `compactMap`, and `reduce`?

```swift
let numbers = [1, 2, 3, 4]

// map — transform each element
numbers.map { $0 * 2 }  // [2, 4, 6, 8]

// compactMap — transform + remove nils
let strings = ["1", "two", "3"]
strings.compactMap { Int($0) }  // [1, 3]

// flatMap — flatten nested arrays
let nested = [[1, 2], [3, 4]]
nested.flatMap { $0 }  // [1, 2, 3, 4]

// reduce — combine into single value
numbers.reduce(0, +)  // 10
```

---

## Q10: What is `Result` type?

```swift
enum NetworkError: Error {
    case badURL
    case noData
}

func fetchData(from url: String) -> Result<Data, NetworkError> {
    guard let _ = URL(string: url) else {
        return .failure(.badURL)
    }
    return .success(Data())
}

switch fetchData(from: "https://api.example.com") {
case .success(let data):
    print("Got \(data.count) bytes")
case .failure(let error):
    print("Error: \(error)")
}
```
