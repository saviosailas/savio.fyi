# 19. Swift 6 — What's New and What Changed

> A comprehensive guide covering Swift 6.0, 6.1, and 6.2 — designed to support a 36-hour lecture series on the Swift 6 era.

---

## Table of Contents

| Module | Topic | Suggested Hours |
|--------|-------|-----------------|
| 1 | [Overview & Philosophy](#module-1-overview--philosophy) | 1 hr |
| 2 | [Complete Concurrency by Default](#module-2-complete-concurrency-by-default) | 5 hrs |
| 3 | [Typed Throws](#module-3-typed-throws) | 2 hrs |
| 4 | [Noncopyable Types Upgrades](#module-4-noncopyable-types-upgrades) | 3 hrs |
| 5 | [Access-Level Modifiers on Imports](#module-5-access-level-modifiers-on-imports) | 1.5 hrs |
| 6 | [New Standard Library Additions](#module-6-new-standard-library-additions) | 2 hrs |
| 7 | [Parameter Packs & Pack Iteration](#module-7-parameter-packs--pack-iteration) | 2 hrs |
| 8 | [Swift 6.1 Refinements](#module-8-swift-61-refinements) | 3 hrs |
| 9 | [Swift 6.2 — Approachable Concurrency](#module-9-swift-62--approachable-concurrency) | 4 hrs |
| 10 | [Swift 6.2 — Language Additions](#module-10-swift-62--language-additions) | 3 hrs |
| 11 | [Swift 6.2 — Performance & Low-Level](#module-11-swift-62--performance--low-level) | 2.5 hrs |
| 12 | [Swift Testing Evolution](#module-12-swift-testing-evolution) | 3 hrs |
| 13 | [Migration Strategy & Best Practices](#module-13-migration-strategy--best-practices) | 2 hrs |
| 14 | [Hands-On Labs & Exercises](#module-14-hands-on-labs--exercises) | 2 hrs |
| | **Total** | **36 hrs** |

---

## Module 1: Overview & Philosophy

### Why Swift 6?

Swift 6 is not just a version bump — it represents a fundamental shift in how the language approaches safety. While Swift has always been about memory safety (no dangling pointers, no buffer overflows), Swift 6 extends that promise to **data-race safety**. The compiler now proves at compile time that your concurrent code is free from data races.

### The Swift 6 Timeline

| Version | Release | Shipped With | Key Theme |
|---------|---------|-------------|-----------|
| Swift 6.0 | Sep 2024 | Xcode 16 | Complete concurrency checking by default |
| Swift 6.1 | Mar 2025 | Xcode 16.3 | Refinements, trailing commas, package traits |
| Swift 6.2 | Sep 2025 | Xcode 26 | Approachable concurrency, InlineArray, WebAssembly |

### Language Mode vs Compiler Version

SE-0441 formalizes the distinction between the **Swift compiler version** and the **Swift language mode**. You can use the Swift 6 compiler while running in Swift 5 language mode — the compiler is capable of all the latest features, but they are disabled for compatibility.

```swift
// In Package.swift
let package = Package(
    name: "MyApp",
    platforms: [.iOS(.v17)],
    // This controls the language mode, not the compiler version
    swiftLanguageModes: [.v6]
)
```

### Key Principles of Swift 6

1. **Data-race safety at compile time** — the compiler proves your code is safe
2. **Incremental adoption** — migrate module by module, not all at once
3. **Backward compatibility** — Swift 5 language mode still works with the Swift 6 compiler
4. **Approachable concurrency** (6.2) — you don't need to understand concurrency to write safe code

---

## Module 2: Complete Concurrency by Default

*Estimated lecture time: 5 hours*

This is the single biggest change in Swift 6. Complete concurrency checking, which was opt-in during Swift 5.10, is now **enabled by default** in Swift 6 language mode.

### 2.1 What Changed

Before Swift 6, the compiler would warn about potential data races. In Swift 6, those warnings become **errors**. The compiler uses a concept called **data isolation** to ensure that mutable state is never accessed from multiple concurrent contexts simultaneously.

### 2.2 Isolation Regions (SE-0414)

The most impactful concurrency change. The compiler can now prove that different parts of your code can run concurrently by analyzing **isolation regions**.

```swift
class User {
    var name = "Anonymous"
}

struct ContentView: View {
    var body: some View {
        Text("Hello, world!")
            .task {
                let user = User()
                await loadData(for: user)
            }
    }

    func loadData(for user: User) async {
        print("Loading data for \(user.name)…")
    }
}
```

**Before Swift 6:** The call to `loadData()` would produce a warning: *"passing argument of non-sendable type 'User' outside of main actor-isolated context may introduce data races."*

**After Swift 6:** No warning. The compiler detects that `user` isn't accessed from multiple places at once, so it's safe. The compiler analyzes program flow to prove safety — a dramatic simplification for developers.

### 2.3 The `sending` Keyword (SE-0430)

When you need to send values between isolation regions, use the new `sending` keyword:

```swift
func processData(_ data: sending Data) async {
    // `data` is guaranteed to be the only reference
    // Safe to use across isolation boundaries
}
```

The `sending` keyword tells the compiler that ownership of the value is being transferred — no other code retains a reference to it.

### 2.4 Caller-Isolated Async Functions (SE-0420)

Async functions can now be isolated to the same actor as their caller:

```swift
actor DataStore {
    var items: [String] = []

    func fetchItems() async -> [String] {
        // This function runs on the DataStore actor
        return items
    }
}
```

### 2.5 Global Variable Safety (SE-0412)

Global variables and static properties must now be safe in concurrent environments:

```swift
// ❌ Error in Swift 6 — mutable global state is unsafe
var globalCounter = 0

// ✅ Option 1: Make it a constant
let maximumRetries = 3

// ✅ Option 2: Restrict to a global actor
@MainActor
var currentUser: String = "Anonymous"

// ✅ Option 3: Use nonisolated(unsafe) if you know it's protected
nonisolated(unsafe) var legacyCache = [String: Data]()
```

### 2.6 Property Wrapper Actor Inference Removed (SE-0401)

Previously, using `@StateObject` or `@ObservedObject` would automatically make your entire SwiftUI view `@MainActor`. This implicit inference is removed in Swift 6.

```swift
@MainActor  // Now required explicitly
class ViewModel: ObservableObject {
    func authenticate() {
        print("Authenticating…")
    }
}

@MainActor  // Must be explicit in Swift 6
struct LogInView: View {
    @StateObject private var model = ViewModel()

    var body: some View {
        Button("Log In", action: startAuthentication)
    }

    func startAuthentication() {
        model.authenticate()
    }
}
```

### 2.7 Default Value Isolation (SE-0411)

Function default values now inherit the same isolation as the function they belong to:

```swift
@MainActor
class Logger { }

@MainActor
class DataController {
    // Logger() inherits @MainActor isolation — this is now valid
    init(logger: Logger = Logger()) { }
}
```

### 2.8 Improved Objective-C Interop (SE-0423)

Concurrency support when working with Objective-C frameworks has been improved, reducing friction when bridging between Swift concurrency and Objective-C callback patterns.

### 2.9 Lecture Exercises

1. Take an existing SwiftUI project and enable Swift 6 language mode. Catalog every error.
2. Identify which errors are "real" data races vs false positives that the compiler can't prove safe.
3. Practice converting global mutable state to actor-isolated or `nonisolated(unsafe)`.
4. Refactor a class hierarchy to use `sending` for cross-isolation transfers.

---

## Module 3: Typed Throws

*Estimated lecture time: 2 hours*

### 3.1 The Problem

Before Swift 6, all `throws` functions threw `any Error`. You always needed a catch-all clause even when you'd handled every possible error:

```swift
// Pre-Swift 6: What errors can this throw? Who knows!
func loadData() throws -> Data { ... }
```

### 3.2 Typed Throws (SE-0413)

Swift 6 lets you specify exactly what error type a function can throw:

```swift
enum CopierError: Error {
    case outOfPaper
}

struct Photocopier {
    var pagesRemaining: Int

    mutating func copy(count: Int) throws(CopierError) {
        guard count <= pagesRemaining else {
            throw CopierError.outOfPaper
        }
        pagesRemaining -= count
    }
}
```

Now the call site knows exactly what can go wrong:

```swift
do {
    var copier = Photocopier(pagesRemaining: 100)
    try copier.copy(count: 10)
} catch CopierError.outOfPaper {
    print("Please refill the paper")
}
// No catch-all needed!
```

### 3.3 Key Rules

- `throws(SomeError)` — only `SomeError` can be thrown
- `throws(any Error)` — equivalent to plain `throws`
- `throws(Never)` — equivalent to a non-throwing function
- You **cannot** write `throws(A, B, C)` — only one error type per function

### 3.4 Typed Throws with Generics

This is where typed throws really shine — `rethrows` can now be expressed more clearly:

```swift
public func count<E>(
    where predicate: (Element) throws(E) -> Bool
) throws(E) -> Int {
    // If predicate doesn't throw, this function doesn't throw either
    // If predicate throws SomeError, this function throws SomeError
}
```

### 3.5 When to Use Typed Throws

- **Use for:** Embedded Swift, performance-critical code, internal APIs with stable error types
- **Avoid for:** Library public APIs (locks you into a contract), code where error types may evolve
- The Swift Evolution authors themselves say: *"Even with typed throws, untyped throws is better for most scenarios."*

### 3.6 Lecture Exercises

1. Refactor an existing networking layer to use typed throws.
2. Build a generic retry function that preserves the error type using typed throws.
3. Discuss: when would typed throws break your API contract?

---

## Module 4: Noncopyable Types Upgrades

*Estimated lecture time: 3 hours*

### 4.1 Background

Noncopyable types (`~Copyable`) were introduced in Swift 5.9 to model unique ownership — values that cannot be duplicated. Swift 6 brings major upgrades.

### 4.2 Automatic Copyable Conformance (SE-0427)

Every struct, class, enum, generic type parameter, and protocol in Swift 6 **automatically conforms to `Copyable`** unless you explicitly opt out:

```swift
// This struct is automatically Copyable
struct Point {
    var x: Double
    var y: Double
}

// Explicitly opt out
struct UniqueToken: ~Copyable {
    let id: UUID
    consuming func use() {
        print("Token \(id) consumed")
    }
}
```

### 4.3 Noncopyable Types with Generics

Because `Optional` is a generic enum, noncopyable types can now be optional:

```swift
struct FileHandle: ~Copyable {
    let descriptor: Int32

    consuming func close() {
        print("Closing file descriptor \(descriptor)")
    }
}

// This now works — Optional<FileHandle> is valid
var handle: FileHandle? = FileHandle(descriptor: 42)
handle?.close()
handle = nil
```

### 4.4 Noncopyable Protocol Conformance

Noncopyable types can conform to protocols, but only if those protocols are also `~Copyable`:

```swift
protocol Consumable: ~Copyable {
    consuming func consume()
}

struct Message: ~Copyable, Consumable {
    var agent: String
    private var content: String

    init(agent: String, content: String) {
        self.agent = agent
        self.content = content
    }

    consuming func consume() {
        print("\(agent): \(content)")
        // Message is destroyed after this call
    }
}
```

### 4.5 Partial Consumption (SE-0429)

Noncopyable types that contain other noncopyable types can now be partially consumed:

```swift
struct Package: ~Copyable {
    var from: String = "HQ"
    var message: Message

    consuming func read() {
        message.consume()
    }
}
```

This was previously a compiler error. Now it works as long as the types don't have deinitializers.

### 4.6 Pattern Matching with Borrowing (SE-0432)

You can now use `switch` with `where` clauses on noncopyable values:

```swift
enum Order: ~Copyable {
    case signed(Package)
    case anonymous(Message)
}

func processOrder(_ order: consuming Order) {
    switch consume order {
    case .signed(let package):
        package.read()
    case .anonymous(let message) where message.agent == "Ethan Hunt":
        print("Play dramatic music")
        message.consume()
    case .anonymous(let message):
        message.consume()
    }
}
```

### 4.7 Lecture Exercises

1. Model a database transaction using noncopyable types (must be committed or rolled back, never duplicated).
2. Build a `UniqueResource<T: ~Copyable>` wrapper that enforces single-ownership semantics.
3. Implement a file handle abstraction that prevents use-after-close bugs at compile time.

---

## Module 5: Access-Level Modifiers on Imports

*Estimated lecture time: 1.5 hours*

### 5.1 The Problem

Before Swift 6, importing a module in one file could leak some of that module's API to other files in your project. This caused surprising behavior and accidental dependency exposure.

### 5.2 Access-Level Imports (SE-0409)

You can now control the visibility of your imports:

```swift
// Only this file can use SomeInternalLib
private import SomeInternalLib

// The module is available within this module but not to consumers
internal import Networking

// Explicitly expose a dependency to consumers
public import CoreModels
```

### 5.3 Practical Example

Consider a banking app architecture:

```
App → Banking Library → Transactions, Networking (internal)
```

```swift
// In Banking library
internal import Transactions  // Not exposed to App

public func sendMoney(from: Int, to: Int) -> TransactionResult {
    // Can use Transactions internally
    let tx = BankTransaction(from: from, to: to)
    return tx.execute()
}
```

If `sendMoney` tried to return a `BankTransaction` type from the `Transactions` module, the compiler would reject it — you can't expose types from an `internal` import in your `public` API.

### 5.4 Default Behavior

- **Swift 6 language mode:** imports default to `internal`
- **Swift 5 language mode:** imports default to `public` (backward compatible)

### 5.5 Lecture Exercises

1. Audit a multi-module project for accidental dependency leakage.
2. Refactor imports to use appropriate access levels.
3. Discuss: how does this change affect SPM package design?

---

## Module 6: New Standard Library Additions

*Estimated lecture time: 2 hours*

### 6.1 count(where:) (SE-0220)

A long-awaited addition — count elements matching a predicate in a single pass:

```swift
let scores = [100, 80, 85, 92, 45, 78]
let passCount = scores.count { $0 >= 85 }
// passCount == 3

let names = ["Terry Gilliam", "Terry Jones", "John Cleese", "Eric Idle"]
let terryCount = names.count { $0.hasPrefix("Terry") }
// terryCount == 2
```

This is more efficient than `filter({ ... }).count` because it doesn't create an intermediate array.

### 6.2 128-bit Integer Types (SE-0120)

Swift 6 adds `Int128` and `UInt128`:

```swift
let bigNumber: Int128 = 170_141_183_460_469_231_731_687_303_715_884_105_727
let unsignedBig: UInt128 = 340_282_366_920_938_463_463_374_607_431_768_211_455

// Useful for cryptography, unique identifiers, and high-precision math
```

### 6.3 BitwiseCopyable Protocol (SE-0426)

A marker protocol that tells the compiler a type can be copied with a simple memory copy (like C's `memcpy`), enabling optimizations:

```swift
// Automatically inferred for most value types
struct Coordinate: BitwiseCopyable {
    var x: Double
    var y: Double
}

// Opt out explicitly if needed
@frozen
public enum CommandLine: ~BitwiseCopyable { }
```

Key rules:
- Automatically applied to structs/enums with all-BitwiseCopyable properties
- Disabled for `public`/`package` types unless marked `@frozen`
- Must opt out at the declaration site, not in an extension

### 6.4 Collection Operations on Noncontiguous Elements (SE-0270)

New operations for working with noncontiguous ranges within collections, enabling efficient manipulation of scattered elements.

### 6.5 Lecture Exercises

1. Benchmark `count(where:)` vs `filter().count` on large datasets.
2. Implement a UUID-like type using `UInt128`.
3. Explore which standard library types conform to `BitwiseCopyable`.

---

## Module 7: Parameter Packs & Pack Iteration

*Estimated lecture time: 2 hours*

### 7.1 Background

Parameter packs (introduced in Swift 5.9) allow functions to accept a variable number of type parameters. Swift 6 adds the ability to **iterate** over them.

### 7.2 Pack Iteration (SE-0408)

You can now loop over parameter packs using `for-in`:

```swift
func == <each Element: Equatable>(
    lhs: (repeat each Element),
    rhs: (repeat each Element)
) -> Bool {
    for (left, right) in repeat (each lhs, each rhs) {
        guard left == right else { return false }
    }
    return true
}
```

This single function enables tuple comparison for **any arity** — no more limits at 6 elements:

```swift
// This now works!
let a = (1, "hello", true, 3.14, [1, 2], "world", 42)
let b = (1, "hello", true, 3.14, [1, 2], "world", 42)
print(a == b) // true
```

### 7.3 Practical Applications

```swift
// A generic zip that works with any number of sequences
func zipAll<each S: Sequence>(_ sequences: repeat each S) -> [(repeat (each S).Element)] {
    // Implementation using pack iteration
}

// Type-safe heterogeneous containers
func printAll<each T: CustomStringConvertible>(_ values: repeat each T) {
    for value in repeat each values {
        print(value)
    }
}

printAll(42, "hello", true, 3.14)
// Prints: 42, hello, true, 3.14
```

### 7.4 Lecture Exercises

1. Implement a type-safe `combine` function that merges results from multiple async operations with different return types.
2. Build a variadic `validate` function that checks multiple conditions of different types.
3. Discuss: how do parameter packs compare to variadic generics in other languages?

---

## Module 8: Swift 6.1 Refinements

*Estimated lecture time: 3 hours*

### 8.1 Trailing Commas Everywhere (SE-0439)

Trailing commas are now allowed in all comma-separated lists — arrays, dictionaries, tuples, function calls, generic parameters, and string interpolation:

```swift
func add<T: Numeric,>(_ a: T, _ b: T,) -> T {
    a + b
}

let result = add(1, 5,)

// Most useful for multi-line calls:
let range = message.range(
    of: "impossible",
    options: .caseInsensitive,  // Can comment this out without breaking
)
```

Note: This does **not** apply to enum case lists or other non-delimited lists.

### 8.2 Metatype Key Paths (SE-0438)

Key paths now support static properties:

```swift
struct WarpDrive {
    static let maximumSpeed = 9.975
    var currentSpeed = 8.0
}

// Instance property key path (unchanged)
let currentSpeed = \WarpDrive.currentSpeed

// Static property key path (new in 6.1)
let maxSpeed = \WarpDrive.Type.maximumSpeed

// With type annotation
let specificType: KeyPath<WarpDrive.Type, Double> = \.maximumSpeed
```

### 8.3 TaskGroup Child Type Inference (SE-0442)

No more specifying `of:` when creating task groups:

```swift
// Before Swift 6.1
let result = await withTaskGroup(of: String.self) { group in
    group.addTask { "Hello" }
    // ...
}

// Swift 6.1 and later — type is inferred
let result = await withTaskGroup { group in
    group.addTask { "Hello" }
    group.addTask { "World" }

    var collected = [String]()
    for await value in group {
        collected.append(value)
    }
    return collected.joined(separator: " ")
}
```

### 8.4 nonisolated on Types (SE-0449)

Types can now opt out of inherited global actor isolation:

```swift
@MainActor
protocol DataStoring {
    var controller: DataController { get }
}

// Opts out of @MainActor inheritance
nonisolated struct BackgroundProcessor: DataStoring {
    let controller = DataController()

    init() async {
        await controller.load()  // Must use await now
    }
}
```

### 8.5 Member Import Visibility (SE-0444)

Importing a module in one file no longer leaks its extensions to other files. Enable with the `MemberImportVisibility` upcoming feature flag:

```swift
// File1.swift
import Maps  // Maps adds toRadians() to Double

// File2.swift
import GeoKit  // GeoKit also adds toRadians() to Double

// Before 6.1: Both toRadians() visible everywhere — ambiguity!
// After 6.1: Each file only sees its own imports
```

### 8.6 Diagnostic Groups (SE-0443)

Fine-grained control over compiler warnings:

```swift
// Add -print-diagnostic-groups to see group names in warnings
// Then control them:
// -Werror DeprecatedDeclaration  → upgrade deprecation warnings to errors
// -Wwarning DeprecatedDeclaration → keep as warnings even with -warnings-as-errors
```

### 8.7 Package Traits (SE-0450)

Swift packages can now define optional traits:

```swift
// Package.swift
let package = Package(
    name: "MyLib",
    traits: [
        .trait(name: "Experimental"),
        .trait(name: "Logging", enabledByDefault: true),
    ],
    // ...
)

// Consumer can opt in
.package(url: "...", from: "1.0.0", traits: ["Experimental"])
```

### 8.8 Lecture Exercises

1. Refactor a project to use trailing commas in multi-line function calls.
2. Enable `MemberImportVisibility` and fix any resulting import issues.
3. Create a Swift package with optional traits for debug/release configurations.

---

## Module 9: Swift 6.2 — Approachable Concurrency

*Estimated lecture time: 4 hours*

This is the most significant concurrency evolution since async/await was introduced in Swift 5.5.

### 9.1 Default Actor Isolation Inference (SE-0466)

The headline feature: opt your entire module into running on the main actor by default.

```swift
// Add to compiler flags: -default-isolation MainActor

// Now this just works — no @MainActor annotations needed
class DataController {
    func load() { }
    func save() { }
}

struct App {
    let controller = DataController()

    init() {
        controller.load()  // No await needed — everything is on MainActor
    }
}
```

**Why this matters:**
1. Applied per-module — external modules still run on their own actors
2. Networking code like `URLSession.shared.data(from:)` still runs on its own task
3. A single modern CPU core runs at 4+ GHz — most iOS apps can do all work serially
4. Many developers were already using "make everything @MainActor" as their default
5. Part of the broader "Improving the approachability of data-race safety" vision

### 9.2 Nonisolated Async Functions Run on Caller's Actor (SE-0461)

Before Swift 6.2, nonisolated async functions would hop off the caller's actor. Now they stay:

```swift
struct Measurements {
    func fetchLatest() async throws -> [Double] {
        let url = URL(string: "https://example.com/readings.json")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Double].self, from: data)
    }
}

@MainActor
struct WeatherStation {
    let measurements = Measurements()

    func getAverage() async throws -> Double {
        // In Swift 6.2: fetchLatest() runs on MainActor (caller's actor)
        // Before 6.2: fetchLatest() would hop to a background executor
        let readings = try await measurements.fetchLatest()
        return readings.reduce(0, +) / Double(readings.count)
    }
}
```

Use `@concurrent` to opt back into the old behavior:

```swift
struct Measurements {
    @concurrent
    func fetchLatest() async throws -> [Double] {
        // Explicitly runs off the caller's actor
    }
}
```

### 9.3 Immediate Tasks (SE-0472)

Tasks can now start executing immediately rather than being queued:

```swift
print("Starting")

Task {
    print("In Task")  // Queued — runs later
}

Task.immediate {
    print("In Immediate Task")  // Runs NOW
}

print("Done")

// Output: Starting, In Immediate Task, Done, In Task
```

Immediate tasks run synchronously until the first suspension point, then behave like regular tasks. Also available in task groups via `addImmediateTask()`.

### 9.4 Global-Actor Isolated Conformances (SE-0470)

Protocol conformances can be restricted to a specific actor:

```swift
@MainActor
class User: @MainActor Equatable {
    var id: UUID
    var name: String

    init(name: String) {
        self.id = UUID()
        self.name = name
    }

    static func ==(lhs: User, rhs: User) -> Bool {
        lhs.id == rhs.id
    }
}
```

Without `@MainActor Equatable`, the `==` method could be called from any context, violating the class's main-actor isolation.

### 9.5 Isolated Synchronous Deinit (SE-0371)

Deinitializers can now be actor-isolated:

```swift
class User {
    var isLoggedIn = false
}

@MainActor
class Session {
    let user: User

    init(user: User) {
        self.user = user
        user.isLoggedIn = true
    }

    isolated deinit {
        user.isLoggedIn = false  // Safe — runs on MainActor
    }
}
```

### 9.6 Task Priority Escalation APIs (SE-0462)

Detect and manually control task priority escalation:

```swift
let fetcher = Task(priority: .medium) {
    try await withTaskPriorityEscalationHandler {
        let url = URL(string: "https://example.com/data.json")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    } onPriorityEscalated: { oldPriority, newPriority in
        print("Escalated from \(oldPriority) to \(newPriority)")
    }
}

// Manually escalate
fetcher.escalatePriority(to: .high)
```

### 9.7 Task Naming (SE-0469)

Give tasks human-readable names for debugging:

```swift
let task = Task(name: "FetchUserProfile") {
    print("Current task: \(Task.name ?? "Unknown")")
}

// In task groups
await withTaskGroup { group in
    for i in 1...5 {
        group.addTask(name: "FetchPage-\(i)") {
            // ...
        }
    }
}
```

### 9.8 Lecture Exercises

1. Enable `-default-isolation MainActor` on an existing project and observe the impact.
2. Refactor a networking layer to use `@concurrent` where background execution is needed.
3. Build a task priority monitoring dashboard using `withTaskPriorityEscalationHandler`.
4. Compare `Task.immediate` vs `Task` behavior with timing measurements.

---

## Module 10: Swift 6.2 — Language Additions

*Estimated lecture time: 3 hours*

### 10.1 Raw Identifiers (SE-0451)

Backtick identifiers can now contain spaces, numbers, and special characters:

```swift
// HTTP error codes as enum cases
enum HTTPError: String {
    case `401` = "Unauthorized"
    case `404` = "Not Found"
    case `500` = "Internal Server Error"
}

let error = HTTPError.`404`

// Human-readable test names
import Testing

@Test
func `Strip HTML tags from string`() {
    // No need for @Test("description") anymore
}

// Functions with descriptive names
func `calculate tax for fiscal year 2024`() -> Decimal {
    // ...
}
```

Rules: Raw identifiers may contain operator characters but cannot consist **only** of operator characters.

### 10.2 Default Values in String Interpolation (SE-0477)

Provide fallback values for optional interpolation:

```swift
var name: String? = nil
var age: Int? = nil

// Before: nil coalescing doesn't work across types
print("Age: \(age ?? 0)")          // Works — both Int
// print("Age: \(age ?? "Unknown")")  // ❌ Error — Int? vs String

// Swift 6.2: default parameter works across types
print("Hello, \(name, default: "Anonymous")!")
print("Age: \(age, default: "Unknown")")  // ✅ Works!
```

### 10.3 enumerated() Conforms to Collection (SE-0459)

The return type of `enumerated()` now conforms to `Collection`, enabling direct use in SwiftUI:

```swift
import SwiftUI

struct ContentView: View {
    var names = ["Bernard", "Laverne", "Hoagie"]

    var body: some View {
        List(names.enumerated(), id: \.offset) { values in
            Text("User \(values.offset + 1): \(values.element)")
        }
    }
}
```

Also enables constant-time operations like `(1000..<2000).enumerated().dropFirst(500)`.

### 10.4 Method and Initializer Key Paths (SE-0479)

Key paths now support methods:

```swift
let strings = ["Hello", "world"]

// Property key path (existing)
let capitalized = strings.map(\.capitalized)

// Method key path (new) — invoke the method
let uppercased = strings.map(\.uppercased())

// Uninvoked method reference
let functions = strings.map(\.uppercased)
// functions[0]() returns "HELLO"

// Disambiguate overloads with argument labels
let prefixUpTo = \Array<String>.prefix(upTo:)
let prefixThrough = \Array<String>.prefix(through:)
```

Note: Key paths to `async` or `throws` methods are not supported.

### 10.5 weak let (SE-0481)

Weak references can now be constants:

```swift
final class User: Sendable {
    let id = UUID()
}

final class Session: Sendable {
    weak let user: User?  // Can't be reassigned, but can be deallocated

    init(user: User?) {
        self.user = user
    }
}

var user: User? = User()
let session = Session(user: user)
print(session.user?.id ?? "No ID")  // Prints the UUID

user = nil
print(session.user?.id ?? "No ID")  // Prints "No ID"

// session.user = nil  // ❌ Error — it's a let
```

Key benefit: `weak let` enables `Sendable` conformance, which `weak var` cannot.

### 10.6 Transactional Observation (SE-0475)

Monitor `@Observable` changes outside of SwiftUI:

```swift
@Observable
class Player {
    var score = 0
}

let player = Player()
let scores = Observations { player.score }

// Watch for changes
for await score in scores {
    print("Score: \(score)")  // Emits initial value + all changes
}
```

Key behaviors:
- Emits the initial value immediately
- Coalesces rapid changes into single emissions
- Runs potentially forever — use a separate task
- Set observed value to `nil` to end iteration

### 10.7 Regex Lookbehind Assertions (SE-0448)

```swift
let text = "Items cost $100 and $59.99 respectively."
let regex = /(?<=\$)\d+(?:\.\d{2})?/

for match in text.matches(of: regex) {
    print(match.output)  // "100", "59.99"
}
```

### 10.8 Lecture Exercises

1. Refactor a test suite to use raw identifiers for human-readable test names.
2. Build a logging system that uses default string interpolation values.
3. Create a reactive data layer using `Observations` outside of SwiftUI.
4. Implement a `weak let` delegate pattern and verify `Sendable` conformance.

---

## Module 11: Swift 6.2 — Performance & Low-Level

*Estimated lecture time: 2.5 hours*

### 11.1 InlineArray (SE-0453)

Fixed-size arrays stored inline (no heap allocation):

```swift
// Explicit size and type
var names1: InlineArray<4, String> = ["Moon", "Mercury", "Mars", "Jupiter"]

// Type inference
var names2: InlineArray = ["Moon", "Mercury", "Mars", "Jupiter"]

// Fixed size — no append() or remove()
names1[2] = "Venus"  // Mutation at index is fine

// Iteration via indices (not Sequence/Collection)
for i in names1.indices {
    print(names1[i])
}
```

Powered by SE-0452 (integer generic parameters). Ideal for performance-critical code where array size is known at compile time.

### 11.2 Opt-in Strict Memory Safety (SE-0458)

Flag unsafe code usage with compiler warnings:

```swift
// With strict memory safety enabled:
let name: String?

// Must acknowledge unsafe operations
unsafe print(name.unsafelyUnwrapped)

// New attributes
@safe   // Default — marks code as safe
@unsafe // Marks code as requiring the `unsafe` keyword to call
```

Similar to how `try` and `await` work — the `unsafe` keyword is an acknowledgment for readers.

### 11.3 Swift Backtrace API (SE-0419)

Capture and symbolicate call stacks programmatically:

```swift
import Runtime

func functionA() { functionB() }
func functionB() { functionC() }

func functionC() {
    if let frames = try? Backtrace.capture().symbolicated()?.frames {
        for frame in frames {
            print(frame)
        }
    }
}

functionA()
// Prints: functionC, functionB, functionA with file/line info
```

### 11.4 Non-Escapable Types (SE-0446) & Span (SE-0447)

Types that cannot outlive their scope, enabling safe memory access:

```swift
// Span provides safe, bounds-checked access to contiguous memory
// without the overhead of Array or the danger of UnsafeBufferPointer
func processData(_ span: Span<UInt8>) {
    for byte in span {
        // Safe, bounds-checked access
    }
}
```

### 11.5 Duration Attoseconds (SE-0457)

`Duration` now exposes total attoseconds as `Int128`:

```swift
let duration: Duration = .seconds(1)
let attoseconds = duration.totalAttoseconds  // Int128
// 1 second = 1_000_000_000_000_000_000 attoseconds
```

### 11.6 Yielding Accessors (SE-0474)

Read and write values without copies:

```swift
struct Container {
    private var _value: LargeStruct

    var value: LargeStruct {
        _read { yield _value }
        _modify { yield &_value }
    }
}
```

### 11.7 @abi Attribute (SE-0476)

Rename APIs without breaking ABI:

```swift
@abi(func oldConfusingName())
public func betterName() {
    // Implementation
}
// Binary consumers using oldConfusingName() still work
// Source consumers use betterName()
```

### 11.8 Lecture Exercises

1. Benchmark `InlineArray` vs `Array` for small, fixed-size collections.
2. Enable strict memory safety and audit a project for `unsafe` usage.
3. Build a crash reporter using the Backtrace API.
4. Implement a zero-copy data processing pipeline using `Span`.

---

## Module 12: Swift Testing Evolution

*Estimated lecture time: 3 hours*

### 12.1 Range-Based Confirmations (ST-0005) — Swift 6.1

Test that something happens within a range of times:

```swift
import Testing

@Test func feedsLoadedInRange() async throws {
    await confirmation(expectedCount: 5...10) { confirm in
        for await _ in NewsLoader() {
            confirm()
        }
    }
}

// At least 5 times
await confirmation(expectedCount: 5...) { confirm in
    // ...
}
```

Note: Ranges without lower bounds (e.g., `...10`) are disallowed to avoid ambiguity.

### 12.2 Return Errors from #expect(throws:) (ST-0006) — Swift 6.1

Errors are now returned for further validation:

```swift
enum GameError: Error {
    case disallowedTime
}

func playGame(at time: Int) throws(GameError) {
    if time < 9 || time > 20 {
        throw GameError.disallowedTime
    }
}

@Test func playGameAtNight() {
    let error = #expect(throws: GameError.self) {
        try playGame(at: 22)
    }
    #expect(error == .disallowedTime)
}
```

### 12.3 Test Scoping Traits (ST-0007) — Swift 6.1

Set up precise, concurrency-safe test environments:

```swift
struct Player {
    var name: String
    var friends = [Player]()
    @TaskLocal static var current = Player(name: "Anonymous")
}

struct DefaultPlayerTrait: TestTrait, TestScoping {
    func provideScope(
        for test: Test,
        testCase: Test.Case?,
        performing function: () async throws -> Void
    ) async throws {
        let player = Player(name: "Natsuki Subaru")
        try await Player.$current.withValue(player) {
            try await function()
        }
    }
}

extension Trait where Self == DefaultPlayerTrait {
    static var defaultPlayer: Self { Self() }
}

@Test(.defaultPlayer)
func welcomeScreenShowsName() {
    let result = createWelcomeScreen()
    #expect(result.contains("Natsuki Subaru"))
}
```

### 12.4 Exit Tests (ST-0008) — Swift 6.2

Test code that causes fatal errors:

```swift
struct Dice {
    func roll(sides: Int) -> Int {
        precondition(sides > 0)
        return Int.random(in: 1...sides)
    }
}

@Test func invalidDiceRollsFail() async throws {
    let dice = Dice()

    await #expect(processExitsWith: .failure) {
        let _ = dice.roll(sides: 0)
    }
}
```

Behind the scenes, this spawns a dedicated process for the test, so the crash doesn't take down the test runner.

### 12.5 Attachments (ST-0009) — Swift 6.2

Attach debug data to failing tests:

```swift
import Foundation
import Testing

struct Character: Codable, Attachable {
    var id = UUID()
    var name: String
}

@Test func defaultCharacterNameIsCorrect() {
    let result = makeCharacter()
    #expect(result.name == "Rem")

    // Attach the actual result for debugging
    Attachment.record(result, named: "Character")
}
```

Supports `String`, `Data`, and `Encodable` types out of the box. Image attachments are not yet supported.

### 12.6 Evaluate ConditionTrait (ST-0010) — Swift 6.2

Evaluate test conditions programmatically:

```swift
struct TestManager {
    static let inSmokeTestMode = true
}

func checkTestMode() async throws {
    let trait = ConditionTrait.disabled(if: TestManager.inSmokeTestMode)
    if try await trait.evaluate() {
        print("Smoke test mode — skip long tests")
    }
}
```

### 12.7 Lecture Exercises

1. Build a comprehensive test suite using all Swift Testing features.
2. Implement custom test scoping traits for database and network mocking.
3. Write exit tests for all `precondition` and `fatalError` calls in a project.
4. Create a test attachment system that captures screenshots on failure.

---

## Module 13: Migration Strategy & Best Practices

*Estimated lecture time: 2 hours*

### 13.1 The Incremental Approach

Apple recommends migrating **module by module**, not all at once:

```
Step 1: Stay on Swift 5 language mode with Swift 6 compiler
Step 2: Enable "Complete Concurrency Checking" (warnings only)
Step 3: Fix warnings one module at a time, starting with leaf modules
Step 4: Switch each module to Swift 6 language mode as warnings are resolved
Step 5: Consider Swift 6.2's -default-isolation MainActor for UI modules
```

### 13.2 Common Migration Patterns

**Pattern 1: Global mutable state**
```swift
// Before
var sharedCache = [String: Data]()

// After — pick one:
@MainActor var sharedCache = [String: Data]()           // Actor-isolated
actor CacheActor { var data = [String: Data]() }         // Dedicated actor
nonisolated(unsafe) var sharedCache = [String: Data]()   // Last resort
```

**Pattern 2: Non-Sendable types crossing isolation boundaries**
```swift
// Before — warning in Swift 5.10, error in Swift 6
class UserData {
    var name: String = ""
}

// After — Option A: Make it Sendable
final class UserData: Sendable {
    let name: String
    init(name: String) { self.name = name }
}

// After — Option B: Use sending
func process(_ data: sending UserData) async { }

// After — Option C: Use an actor
actor UserDataStore {
    var name: String = ""
}
```

**Pattern 3: Singletons**
```swift
// Before — incompatible with Swift 6 strict concurrency
class NetworkManager {
    static let shared = NetworkManager()
    var session: URLSession = .shared
}

// After — use an actor
actor NetworkManager {
    static let shared = NetworkManager()
    var session: URLSession = .shared
}
```

**Pattern 4: @preconcurrency for third-party code**
```swift
// Suppress warnings for modules you don't control
@preconcurrency import SomeOldLibrary
```

### 13.3 The Swift 6.2 Shortcut

For UI-heavy apps, Swift 6.2's approachable concurrency dramatically simplifies migration:

1. Enable `-default-isolation MainActor` for your app module
2. Most concurrency errors disappear immediately
3. Mark background work explicitly with `@concurrent` or dedicated actors
4. Gradually introduce concurrency where it provides real benefit

### 13.4 What NOT to Do

- Don't sprinkle `@unchecked Sendable` everywhere — it hides real bugs
- Don't use `nonisolated(unsafe)` as a default — it's an escape hatch
- Don't try to migrate everything at once — incremental is key
- Don't ignore the warnings — they represent real potential data races

### 13.5 Lecture Exercises

1. Take a real-world open-source iOS app and create a migration plan.
2. Practice the incremental migration on a multi-module project.
3. Identify which modules benefit from `-default-isolation MainActor` vs explicit concurrency.
4. Audit a codebase for `@unchecked Sendable` and replace with proper solutions.

---

## Module 14: Hands-On Labs & Exercises

*Estimated lecture time: 2 hours*

### Lab 1: Build a Concurrent Image Pipeline (4 exercises)

Build an image loading and caching system that:
- Uses actors for thread-safe cache access
- Leverages `sending` for cross-isolation image transfer
- Uses `Task.immediate` for cache hits
- Implements `InlineArray` for a fixed-size LRU cache

### Lab 2: Type-Safe API Client (3 exercises)

Build a networking layer that:
- Uses typed throws for domain-specific errors
- Leverages parameter packs for multi-endpoint batch requests
- Uses noncopyable types for one-time-use authentication tokens

### Lab 3: Observable Data Layer (3 exercises)

Build a data management system that:
- Uses `Observations` for reactive updates outside SwiftUI
- Implements `weak let` for delegate patterns with `Sendable`
- Uses `@concurrent` for background data processing

### Lab 4: Comprehensive Test Suite (3 exercises)

Write tests that:
- Use exit tests for precondition validation
- Implement custom test scoping traits for mock injection
- Attach debug data on test failure
- Use raw identifiers for readable test names

### Lab 5: Migration Workshop (2 exercises)

Take a provided Swift 5 project and:
- Enable complete concurrency checking
- Fix all warnings using proper patterns (not escape hatches)
- Switch to Swift 6 language mode
- Optionally enable `-default-isolation MainActor`

---

## Quick Reference: Swift Evolution Proposals by Version

### Swift 6.0
| SE | Title |
|----|-------|
| SE-0220 | `count(where:)` |
| SE-0401 | Remove actor isolation inference from property wrappers |
| SE-0408 | Pack iteration |
| SE-0409 | Access-level modifiers on import declarations |
| SE-0411 | Isolated default value expressions |
| SE-0412 | Strict concurrency for global variables |
| SE-0413 | Typed throws |
| SE-0414 | Region-based isolation |
| SE-0420 | Async functions isolated to caller |
| SE-0423 | Dynamic actor isolation enforcement from Objective-C |
| SE-0426 | BitwiseCopyable |
| SE-0427 | Noncopyable generics |
| SE-0429 | Partial consumption of noncopyable values |
| SE-0430 | `sending` parameter and result values |
| SE-0432 | Borrowing and consuming pattern matching for noncopyable types |

### Swift 6.1
| SE | Title |
|----|-------|
| SE-0387 | Swift SDKs for cross-compilation |
| SE-0436 | Objective-C implementations in Swift |
| SE-0438 | Metatype key paths |
| SE-0439 | Allow trailing comma in comma-separated lists |
| SE-0441 | Formalize language mode terminology |
| SE-0442 | TaskGroup child task type inference |
| SE-0443 | Precise control flags over compiler warnings |
| SE-0444 | Member import visibility |
| SE-0449 | Allow `nonisolated` to prevent global actor inference |
| SE-0450 | Package traits |

### Swift 6.2
| SE | Title |
|----|-------|
| SE-0371 | Isolated synchronous deinit |
| SE-0419 | Swift Backtrace API |
| SE-0446 | Non-escapable types |
| SE-0447 | Span: safe access to contiguous storage |
| SE-0448 | Regex lookbehind assertions |
| SE-0451 | Raw identifiers |
| SE-0452 | Integer generic parameters |
| SE-0453 | InlineArray (fixed-size array) |
| SE-0457 | Duration.totalAttoseconds as Int128 |
| SE-0458 | Opt-in strict memory safety |
| SE-0459 | enumerated() conforms to Collection |
| SE-0461 | Nonisolated async inherits caller isolation |
| SE-0462 | Task priority escalation APIs |
| SE-0463 | @Sendable for Objective-C completion handlers |
| SE-0466 | Default actor isolation inference |
| SE-0469 | Task naming |
| SE-0470 | Global-actor isolated conformances |
| SE-0472 | Starting tasks synchronously |
| SE-0474 | Yielding accessors |
| SE-0475 | Transactional observation of values |
| SE-0476 | @abi attribute |
| SE-0477 | Default value in string interpolation |
| SE-0479 | Method and initializer key paths |
| SE-0480 | Diagnostic severity in Swift packages |
| SE-0481 | `weak let` |

---

## Recommended Resources

- [Swift Evolution Proposals](https://github.com/swiftlang/swift-evolution)
- [Hacking with Swift — What's New in Swift](https://www.hackingwithswift.com/swift/)
- [WWDC24 — Migrate your app to Swift 6](https://developer.apple.com/videos/play/wwdc2024/10169/)
- [Swift.org — Swift 6.2 Released](https://www.swift.org/blog/swift-6.2-released/)
- [Swift Migration Guide](https://www.swift.org/migration/documentation/migrationguide/)
- [Improving the Approachability of Data-Race Safety (Vision Document)](https://github.com/swiftlang/swift-evolution/blob/main/visions/approachable-concurrency.md)
