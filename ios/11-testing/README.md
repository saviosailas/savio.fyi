# 11. Testing

## Q1: What are the types of testing in iOS?

| Type | Tool | Purpose |
|---|---|---|
| Unit Tests | XCTest | Test individual functions/classes |
| Integration Tests | XCTest | Test components working together |
| UI Tests | XCUITest | Test user interactions and flows |
| Snapshot Tests | swift-snapshot-testing | Compare UI screenshots |
| Performance Tests | XCTest `measure` | Benchmark execution time |

---

## Q2: Writing Unit Tests

```swift
import XCTest
@testable import MyApp

final class UserViewModelTests: XCTestCase {

    var sut: UserViewModel!  // system under test
    var mockRepository: MockUserRepository!

    override func setUp() {
        super.setUp()
        mockRepository = MockUserRepository()
        sut = UserViewModel(repository: mockRepository)
    }

    override func tearDown() {
        sut = nil
        mockRepository = nil
        super.tearDown()
    }

    func test_loadUser_success_updatesDisplayName() async {
        // Given
        mockRepository.mockUser = User(name: "Alice", email: "[email]")

        // When
        await sut.loadUser(id: 1)

        // Then
        XCTAssertEqual(sut.displayName, "Alice")
        XCTAssertNil(sut.errorMessage)
        XCTAssertFalse(sut.isLoading)
    }

    func test_loadUser_failure_setsErrorMessage() async {
        // Given
        mockRepository.shouldFail = true

        // When
        await sut.loadUser(id: 1)

        // Then
        XCTAssertNotNil(sut.errorMessage)
        XCTAssertEqual(sut.displayName, "")
    }
}
```

**Naming convention:** `test_methodName_condition_expectedResult`

---

## Q3: Mocking and Protocols for Testability

```swift
// Protocol
protocol NetworkService {
    func fetch<T: Decodable>(from url: URL) async throws -> T
}

// Mock
class MockNetworkService: NetworkService {
    var result: Any?
    var error: Error?
    var fetchCallCount = 0

    func fetch<T: Decodable>(from url: URL) async throws -> T {
        fetchCallCount += 1
        if let error { throw error }
        return result as! T
    }
}

// Test
func test_fetchUsers_callsNetworkOnce() async throws {
    let mock = MockNetworkService()
    mock.result = [User(name: "Test", email: "[email]")]
    let service = UserService(network: mock)

    _ = try await service.getUsers()

    XCTAssertEqual(mock.fetchCallCount, 1)
}
```

---

## Q4: Testing Async Code

```swift
// async/await — straightforward
func test_asyncFetch() async throws {
    let result = try await sut.fetchData()
    XCTAssertFalse(result.isEmpty)
}

// Combine
func test_publisherEmitsValue() {
    let expectation = expectation(description: "Value received")
    var receivedValue: String?

    sut.$name
        .dropFirst()
        .sink { value in
            receivedValue = value
            expectation.fulfill()
        }
        .store(in: &cancellables)

    sut.updateName("Alice")

    wait(for: [expectation], timeout: 1.0)
    XCTAssertEqual(receivedValue, "Alice")
}

// Callback-based (legacy)
func test_callbackFetch() {
    let expectation = expectation(description: "Fetch completes")

    sut.fetchData { result in
        XCTAssertNotNil(result)
        expectation.fulfill()
    }

    wait(for: [expectation], timeout: 5.0)
}
```

---

## Q5: UI Testing with XCUITest

```swift
final class LoginUITests: XCTestCase {
    let app = XCUIApplication()

    override func setUp() {
        continueAfterFailure = false
        app.launchArguments = ["--uitesting"]
        app.launch()
    }

    func test_loginFlow_withValidCredentials_showsHome() {
        let emailField = app.textFields["emailTextField"]
        emailField.tap()
        emailField.typeText("[email]")

        let passwordField = app.secureTextFields["passwordTextField"]
        passwordField.tap()
        passwordField.typeText("password123")

        app.buttons["loginButton"].tap()

        XCTAssertTrue(app.staticTexts["Welcome"].waitForExistence(timeout: 5))
    }
}
```

**Tips:**
- Use accessibility identifiers for reliable element lookup.
- Use `waitForExistence(timeout:)` for async UI changes.
- Launch arguments to configure test state (mock data, skip onboarding).

---

## Q6: What is code coverage and what's a good target?

Code coverage measures the percentage of code executed during tests.

- Enable in Xcode: Edit Scheme → Test → Options → Code Coverage.
- Aim for 70-80% on business logic.
- 100% coverage doesn't mean bug-free — focus on meaningful tests.
- Prioritize: ViewModels, Services, Utilities > Views, AppDelegate.

---

## Q7: Performance Testing

```swift
func test_sortPerformance() {
    let largeArray = (0..<10_000).map { _ in Int.random(in: 0...10_000) }

    measure {
        _ = largeArray.sorted()
    }
    // Xcode shows average time and deviation
}

// Set baselines for regression detection
func test_jsonParsingPerformance() {
    let jsonData = loadLargeJSON()

    measure(metrics: [XCTClockMetric(), XCTMemoryMetric()]) {
        _ = try? JSONDecoder().decode([User].self, from: jsonData)
    }
}
```
