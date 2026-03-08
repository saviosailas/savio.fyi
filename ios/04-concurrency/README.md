# 4. Concurrency & Multithreading

## Q1: Explain GCD (Grand Central Dispatch)

GCD manages a pool of threads. You submit work to dispatch queues.

```swift
// Serial queue — tasks execute one at a time
let serial = DispatchQueue(label: "com.app.serial")

// Concurrent queue — tasks can run in parallel
let concurrent = DispatchQueue(label: "com.app.concurrent", attributes: .concurrent)

// Main queue — serial, runs on main thread (UI updates)
DispatchQueue.main.async {
    self.label.text = "Updated"
}

// Global queues — system-provided concurrent queues
DispatchQueue.global(qos: .userInitiated).async {
    let data = self.processData()
    DispatchQueue.main.async {
        self.updateUI(with: data)
    }
}
```

**QoS levels (highest to lowest):**
1. `.userInteractive` — animations, UI
2. `.userInitiated` — user-triggered tasks
3. `.default`
4. `.utility` — long tasks with progress
5. `.background` — prefetching, backups

---

## Q2: What is the difference between `sync` and `async`?

- `async` — returns immediately, work runs on the queue.
- `sync` — blocks the calling thread until work completes.

```swift
// ⚠️ DEADLOCK — never sync on main queue from main thread
DispatchQueue.main.sync {
    // This will deadlock!
}

// Safe: sync on a different queue
let queue = DispatchQueue(label: "com.app.work")
queue.sync {
    // Blocks calling thread until done
}
```

---

## Q3: What is DispatchGroup?

Used to wait for multiple async tasks to complete.

```swift
let group = DispatchGroup()

group.enter()
fetchUsers { users in
    // process
    group.leave()
}

group.enter()
fetchPosts { posts in
    // process
    group.leave()
}

group.notify(queue: .main) {
    print("Both requests finished")
    self.reloadUI()
}
```

---

## Q4: What is DispatchSemaphore?

Controls access to a resource by limiting concurrent access count.

```swift
let semaphore = DispatchSemaphore(value: 3) // max 3 concurrent

for url in urls {
    DispatchQueue.global().async {
        semaphore.wait()    // decrement, block if 0
        self.download(url)
        semaphore.signal()  // increment
    }
}
```

---

## Q5: Explain Swift Concurrency (async/await)

Introduced in Swift 5.5. Structured, readable async code.

```swift
func fetchUser(id: Int) async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: userURL(id))
    return try JSONDecoder().decode(User.self, from: data)
}

// Calling
Task {
    do {
        let user = try await fetchUser(id: 1)
        // Already on MainActor if called from SwiftUI
    } catch {
        print(error)
    }
}
```

---

## Q6: What is a Task and TaskGroup?

```swift
// Unstructured task
Task {
    let result = try await fetchData()
}

// Detached task — no inherited context
Task.detached(priority: .background) {
    await self.syncData()
}

// Task group — parallel execution
func fetchAllUsers(ids: [Int]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask {
                try await self.fetchUser(id: id)
            }
        }

        var users: [User] = []
        for try await user in group {
            users.append(user)
        }
        return users
    }
}
```

---

## Q7: What are Actors?

Actors protect mutable state from data races. Only one task can access an actor's state at a time.

```swift
actor BankAccount {
    private var balance: Double = 0

    func deposit(_ amount: Double) {
        balance += amount
    }

    func withdraw(_ amount: Double) -> Bool {
        guard balance >= amount else { return false }
        balance -= amount
        return true
    }

    func getBalance() -> Double {
        balance
    }
}

let account = BankAccount()
await account.deposit(100)
let balance = await account.getBalance()
```

---

## Q8: What is @MainActor?

Ensures code runs on the main thread. Essential for UI updates.

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []

    func loadItems() async {
        let fetched = await service.fetchItems()
        items = fetched  // Safe — guaranteed main thread
    }
}

// Or mark individual functions
@MainActor
func updateUI() {
    label.text = "Done"
}
```

---

## Q9: What is Sendable?

`Sendable` marks types as safe to pass across concurrency boundaries.

```swift
// Value types are implicitly Sendable
struct Point: Sendable {
    let x: Int
    let y: Int
}

// Classes must be final with immutable stored properties
final class Config: Sendable {
    let apiKey: String
    init(apiKey: String) { self.apiKey = apiKey }
}

// @unchecked Sendable — you guarantee thread safety manually
class ThreadSafeCache: @unchecked Sendable {
    private let lock = NSLock()
    private var store: [String: Any] = [:]
}
```

---

## Q10: What are common concurrency problems?

1. **Race Condition** — Multiple threads access shared data simultaneously.
2. **Deadlock** — Two threads wait for each other forever.
3. **Priority Inversion** — Low-priority task holds a resource needed by high-priority task.
4. **Thread Explosion** — Too many threads created, exhausting system resources.

**Solutions:** Use actors, serial queues, locks (`NSLock`, `os_unfair_lock`), or `DispatchBarrier` for reader-writer patterns.
