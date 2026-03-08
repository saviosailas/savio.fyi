# 7. Networking

## Q1: How does URLSession work?

```swift
// Basic GET request
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://api.example.com/users")!
    let (data, response) = try await URLSession.shared.data(from: url)

    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }

    return try JSONDecoder().decode([User].self, from: data)
}

// POST request
func createUser(_ user: User) async throws -> User {
    var request = URLRequest(url: URL(string: "https://api.example.com/users")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(user)

    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(User.self, from: data)
}
```

**URLSession configurations:**
- `.default` — disk-based caching, credentials stored in keychain.
- `.ephemeral` — no persistent storage (like private browsing).
- `.background` — uploads/downloads continue when app is suspended.

---

## Q2: How do you handle Codable (JSON encoding/decoding)?

```swift
struct User: Codable {
    let id: Int
    let fullName: String
    let email: String
    let createdAt: Date

    enum CodingKeys: String, CodingKey {
        case id
        case fullName = "full_name"   // map snake_case
        case email
        case createdAt = "created_at"
    }
}

// Custom decoder configuration
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .iso8601

// Custom decoding
struct Event: Codable {
    let name: String
    let date: Date

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        name = try container.decode(String.self, forKey: .name)

        // Try multiple date formats
        let dateString = try container.decode(String.self, forKey: .date)
        if let date = ISO8601DateFormatter().date(from: dateString) {
            self.date = date
        } else {
            throw DecodingError.dataCorruptedError(forKey: .date, in: container, debugDescription: "Invalid date")
        }
    }
}
```

---

## Q3: How do you build a clean Network Layer?

```swift
// Protocol-based for testability
protocol NetworkService {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

// Endpoint definition
struct Endpoint {
    let path: String
    let method: HTTPMethod
    let headers: [String: String]
    let body: Data?
    let queryItems: [URLQueryItem]

    enum HTTPMethod: String {
        case get = "GET"
        case post = "POST"
        case put = "PUT"
        case delete = "DELETE"
    }
}

// Implementation
class APIClient: NetworkService {
    private let session: URLSession
    private let baseURL: URL
    private let decoder: JSONDecoder

    init(baseURL: URL, session: URLSession = .shared) {
        self.baseURL = baseURL
        self.session = session
        self.decoder = JSONDecoder()
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
    }

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        var components = URLComponents(url: baseURL.appendingPathComponent(endpoint.path), resolvingAgainstBaseURL: true)!
        components.queryItems = endpoint.queryItems.isEmpty ? nil : endpoint.queryItems

        var request = URLRequest(url: components.url!)
        request.httpMethod = endpoint.method.rawValue
        request.httpBody = endpoint.body
        endpoint.headers.forEach { request.setValue($1, forHTTPHeaderField: $0) }

        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(httpResponse.statusCode)
        }

        return try decoder.decode(T.self, from: data)
    }
}
```

---

## Q4: What is Certificate Pinning?

Prevents man-in-the-middle attacks by validating the server's certificate.

```swift
class PinnedSessionDelegate: NSObject, URLSessionDelegate {
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {

        guard let serverTrust = challenge.protectionSpace.serverTrust,
              let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }

        // Compare with pinned certificate
        let serverCertData = SecCertificateCopyData(certificate) as Data
        let pinnedCertData = loadPinnedCertificate()

        if serverCertData == pinnedCertData {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

---

## Q5: How do you handle pagination?

```swift
class PaginatedViewModel: ObservableObject {
    @Published var items: [Item] = []
    @Published var isLoading = false

    private var currentPage = 1
    private var hasMorePages = true
    private let service: NetworkService

    func loadNextPageIfNeeded(currentItem: Item?) async {
        guard let currentItem,
              let index = items.firstIndex(where: { $0.id == currentItem.id }),
              index >= items.count - 3, // prefetch threshold
              !isLoading,
              hasMorePages else { return }

        isLoading = true
        defer { isLoading = false }

        do {
            let page: PageResponse<Item> = try await service.request(
                .items(page: currentPage)
            )
            items.append(contentsOf: page.items)
            hasMorePages = page.hasNext
            currentPage += 1
        } catch {
            // handle error
        }
    }
}
```

---

## Q6: What is URLCache and HTTP caching?

```swift
// Configure cache
let cache = URLCache(
    memoryCapacity: 50 * 1024 * 1024,  // 50 MB memory
    diskCapacity: 100 * 1024 * 1024     // 100 MB disk
)

let config = URLSessionConfiguration.default
config.urlCache = cache
config.requestCachePolicy = .returnCacheDataElseLoad

// Cache policies:
// .useProtocolCachePolicy — default, follows HTTP headers
// .returnCacheDataElseLoad — use cache if available
// .returnCacheDataDontLoad — offline mode
// .reloadIgnoringLocalCacheData — always fetch from network
```
