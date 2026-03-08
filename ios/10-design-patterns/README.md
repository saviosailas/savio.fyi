# 10. Design Patterns

## Q1: Singleton

Ensures a class has only one instance.

```swift
class NetworkManager {
    static let shared = NetworkManager()
    private init() {} // prevent external instantiation

    func fetch(url: URL) async throws -> Data {
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}

// Usage
let data = try await NetworkManager.shared.fetch(url: someURL)
```

**Drawbacks:** Hard to test, hidden dependencies, global mutable state.
**Better alternative:** Dependency injection with protocols.

---

## Q2: Delegation

One object delegates responsibility to another. Ubiquitous in UIKit.

```swift
protocol ImageDownloaderDelegate: AnyObject {
    func didFinishDownloading(_ image: UIImage)
    func didFailWithError(_ error: Error)
}

class ImageDownloader {
    weak var delegate: ImageDownloaderDelegate?

    func download(from url: URL) async {
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            if let image = UIImage(data: data) {
                delegate?.didFinishDownloading(image)
            }
        } catch {
            delegate?.didFailWithError(error)
        }
    }
}
```

**Key:** Always use `weak var delegate` to avoid retain cycles.

---

## Q3: Observer Pattern (NotificationCenter, KVO, Combine)

```swift
// 1. NotificationCenter
NotificationCenter.default.post(name: .userDidLogin, object: nil, userInfo: ["userId": 42])

let observer = NotificationCenter.default.addObserver(
    forName: .userDidLogin, object: nil, queue: .main
) { notification in
    let userId = notification.userInfo?["userId"] as? Int
}

// 2. Combine Publisher
class AuthService: ObservableObject {
    @Published var isLoggedIn = false
}

// Subscribe
let cancellable = authService.$isLoggedIn
    .sink { isLoggedIn in
        print("Login state: \(isLoggedIn)")
    }

// 3. KVO (Key-Value Observing)
let observation = webView.observe(\.estimatedProgress) { webView, _ in
    print("Progress: \(webView.estimatedProgress)")
}
```

---

## Q4: Factory Pattern

Creates objects without exposing creation logic.

```swift
protocol PaymentProcessor {
    func process(amount: Double) async throws
}

class StripeProcessor: PaymentProcessor {
    func process(amount: Double) async throws { /* ... */ }
}

class ApplePayProcessor: PaymentProcessor {
    func process(amount: Double) async throws { /* ... */ }
}

enum PaymentFactory {
    static func makeProcessor(for method: PaymentMethod) -> PaymentProcessor {
        switch method {
        case .creditCard: return StripeProcessor()
        case .applePay: return ApplePayProcessor()
        }
    }
}
```

---

## Q5: Strategy Pattern

Define a family of algorithms, encapsulate each one, and make them interchangeable.

```swift
protocol SortStrategy {
    func sort<T: Comparable>(_ array: inout [T])
}

struct QuickSort: SortStrategy {
    func sort<T: Comparable>(_ array: inout [T]) {
        array.sort() // simplified
    }
}

struct MergeSort: SortStrategy {
    func sort<T: Comparable>(_ array: inout [T]) {
        // merge sort implementation
    }
}

class DataProcessor {
    var sortStrategy: SortStrategy

    init(strategy: SortStrategy) {
        self.sortStrategy = strategy
    }

    func process(_ data: inout [Int]) {
        sortStrategy.sort(&data)
    }
}
```

---

## Q6: Dependency Injection

Pass dependencies from outside rather than creating them internally.

```swift
// Protocol
protocol UserRepository {
    func fetchUser(id: Int) async throws -> User
}

// Production implementation
class RemoteUserRepository: UserRepository {
    func fetchUser(id: Int) async throws -> User {
        // network call
    }
}

// Test mock
class MockUserRepository: UserRepository {
    var mockUser: User?
    func fetchUser(id: Int) async throws -> User {
        guard let user = mockUser else { throw TestError.noData }
        return user
    }
}

// ViewModel accepts protocol, not concrete type
class UserViewModel {
    private let repository: UserRepository

    init(repository: UserRepository) { // injected
        self.repository = repository
    }
}

// Production
let vm = UserViewModel(repository: RemoteUserRepository())

// Test
let vm = UserViewModel(repository: MockUserRepository())
```

**DI types:** Constructor injection (preferred), property injection, method injection.

---

## Q7: Builder Pattern

Construct complex objects step by step.

```swift
class URLRequestBuilder {
    private var url: URL
    private var method = "GET"
    private var headers: [String: String] = [:]
    private var body: Data?

    init(url: URL) { self.url = url }

    func setMethod(_ method: String) -> Self {
        self.method = method
        return self
    }

    func addHeader(_ key: String, _ value: String) -> Self {
        headers[key] = value
        return self
    }

    func setBody(_ body: Data) -> Self {
        self.body = body
        return self
    }

    func build() -> URLRequest {
        var request = URLRequest(url: url)
        request.httpMethod = method
        request.httpBody = body
        headers.forEach { request.setValue($1, forHTTPHeaderField: $0) }
        return request
    }
}

// Usage
let request = URLRequestBuilder(url: apiURL)
    .setMethod("POST")
    .addHeader("Content-Type", "application/json")
    .addHeader("Authorization", "Bearer \(token)")
    .setBody(jsonData)
    .build()
```
