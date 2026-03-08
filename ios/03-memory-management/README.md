# 3. Memory Management

## Q1: How does ARC (Automatic Reference Counting) work?

ARC automatically tracks and manages reference counts for class instances. When the count reaches 0, the instance is deallocated.

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) is initialized")
    }
    deinit {
        print("\(name) is deallocated")
    }
}

var ref1: Person? = Person(name: "Alice") // RC = 1
var ref2 = ref1                            // RC = 2
ref1 = nil                                 // RC = 1
ref2 = nil                                 // RC = 0 → deallocated
```

ARC only applies to reference types (classes, closures).

---

## Q2: What is a Retain Cycle and how do you fix it?

A retain cycle occurs when two objects hold strong references to each other, preventing deallocation.

```swift
// ❌ Retain cycle
class Person {
    var apartment: Apartment?
    deinit { print("Person deinit") }
}

class Apartment {
    var tenant: Person?  // strong reference
    deinit { print("Apartment deinit") }
}

var person: Person? = Person()
var apt: Apartment? = Apartment()
person?.apartment = apt
apt?.tenant = person
person = nil  // Neither is deallocated!
apt = nil

// ✅ Fix with weak
class Apartment {
    weak var tenant: Person?  // weak reference
}
```

---

## Q3: `weak` vs `unowned` — when to use which?

| | `weak` | `unowned` |
|---|---|---|
| Optional? | Always optional (`T?`) | Non-optional |
| When nil? | Set to nil when referenced object deallocates | Dangling pointer → crash if accessed |
| Use when | Referenced object may become nil | Referenced object always outlives |

```swift
// weak — delegate pattern
class ViewController: UIViewController {
    weak var delegate: SomeDelegate?
}

// unowned — parent-child where child can't exist without parent
class Customer {
    let card: CreditCard
    init() { self.card = CreditCard(owner: self) }
}

class CreditCard {
    unowned let owner: Customer
    init(owner: Customer) { self.owner = owner }
}
```

---

## Q4: How do closures cause retain cycles?

Closures capture `self` strongly by default.

```swift
// ❌ Retain cycle
class ViewModel {
    var name = "Test"
    var onUpdate: (() -> Void)?

    func setup() {
        onUpdate = {
            print(self.name) // self captured strongly
        }
    }
}

// ✅ Fix with capture list
func setup() {
    onUpdate = { [weak self] in
        guard let self else { return }
        print(self.name)
    }
}

// ✅ unowned when you're sure self outlives the closure
func setup() {
    onUpdate = { [unowned self] in
        print(self.name)
    }
}
```

---

## Q5: What is the difference between Stack and Heap memory?

| Stack | Heap |
|---|---|
| Value types (struct, enum) | Reference types (class, closure) |
| Fast allocation/deallocation | Slower, requires reference counting |
| Thread-safe (each thread has its own) | Shared across threads |
| Automatic cleanup | ARC manages cleanup |
| Fixed size at compile time | Dynamic size |

---

## Q6: What is `autoreleasepool` and when do you need it?

Used to manage temporary Objective-C objects in tight loops.

```swift
for i in 0..<1_000_000 {
    autoreleasepool {
        let image = processImage(at: i)
        // image released at end of each iteration
    }
}
```

Common scenarios:
- Processing large numbers of temporary objects in loops.
- Working with Objective-C APIs that return autoreleased objects.
- Command-line tools without a run loop.

---

## Q7: How do you detect memory leaks?

1. **Xcode Memory Graph Debugger** — Visual graph of all live objects and their references.
2. **Instruments → Leaks** — Detects leaked memory blocks at runtime.
3. **Instruments → Allocations** — Track memory growth over time.
4. **`deinit` logging** — Add print statements in `deinit` to verify deallocation.
5. **Debug Memory Graph** — Look for unexpected retain cycles.

```swift
class MyViewController: UIViewController {
    deinit {
        print("MyViewController deallocated ✅")
    }
}
```

If you don't see the deinit message after dismissing, you have a leak.

---

## Q8: What is Copy-on-Write (COW)?

Swift collections (Array, Dictionary, Set, String) use COW for performance. The actual copy only happens when a shared instance is mutated.

```swift
var a = [1, 2, 3]
var b = a          // No copy yet, shares same buffer

b.append(4)        // NOW the copy happens (b gets its own buffer)
```

You can implement COW for custom types:
```swift
final class Storage<T> {
    var value: T
    init(_ value: T) { self.value = value }
}

struct MyData {
    private var storage: Storage<[Int]>

    var data: [Int] {
        get { storage.value }
        set {
            if !isKnownUniquelyReferenced(&storage) {
                storage = Storage(newValue)
            } else {
                storage.value = newValue
            }
        }
    }
}
```
