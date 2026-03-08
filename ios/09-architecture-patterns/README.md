# 9. Architecture Patterns

## Q1: MVC (Model-View-Controller)

Apple's default pattern. Often becomes "Massive View Controller."

```
Model ←→ Controller ←→ View
```

```swift
// Model
struct User {
    let name: String
    let email: String
}

// Controller (does too much in practice)
class UserViewController: UIViewController {
    @IBOutlet weak var nameLabel: UILabel!
    private var user: User?

    override func viewDidLoad() {
        super.viewDidLoad()
        fetchUser()
    }

    func fetchUser() {
        // networking + parsing + UI update all here
        nameLabel.text = user?.name
    }
}
```

**Problem:** Controller handles networking, business logic, navigation, and UI formatting.

---

## Q2: MVVM (Model-View-ViewModel)

Separates presentation logic from the view. Most popular in iOS.

```
Model ←→ ViewModel ←→ View
```

```swift
// Model
struct User: Codable {
    let name: String
    let email: String
    let joinDate: Date
}

// ViewModel
@Observable
class UserViewModel {
    var displayName = ""
    var memberSince = ""
    var isLoading = false
    var errorMessage: String?

    private let service: UserService

    init(service: UserService) {
        self.service = service
    }

    func loadUser(id: Int) async {
        isLoading = true
        defer { isLoading = false }

        do {
            let user = try await service.fetchUser(id: id)
            displayName = user.name
            memberSince = "Member since \(user.joinDate.formatted(.dateTime.year().month()))"
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// View (SwiftUI)
struct UserView: View {
    @State private var viewModel: UserViewModel

    var body: some View {
        VStack {
            if viewModel.isLoading {
                ProgressView()
            } else {
                Text(viewModel.displayName)
                Text(viewModel.memberSince)
            }
        }
        .task { await viewModel.loadUser(id: 1) }
    }
}
```

---

## Q3: MVVM-C (with Coordinator)

Adds a Coordinator to handle navigation, removing it from ViewModels.

```swift
protocol Coordinator: AnyObject {
    var navigationController: UINavigationController { get }
    func start()
}

class AppCoordinator: Coordinator {
    let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        showUserList()
    }

    func showUserList() {
        let vm = UserListViewModel()
        vm.onUserSelected = { [weak self] user in
            self?.showUserDetail(user)
        }
        let vc = UserListViewController(viewModel: vm)
        navigationController.pushViewController(vc, animated: true)
    }

    func showUserDetail(_ user: User) {
        let vm = UserDetailViewModel(user: user)
        let vc = UserDetailViewController(viewModel: vm)
        navigationController.pushViewController(vc, animated: true)
    }
}
```

---

## Q4: VIPER

Very granular separation. Common in large enterprise apps.

```
View ←→ Presenter ←→ Interactor → Entity
              ↕
           Router
```

| Component | Responsibility |
|---|---|
| View | Displays data, sends user actions to Presenter |
| Interactor | Business logic, data fetching |
| Presenter | Formats data for View, handles user actions |
| Entity | Plain data models |
| Router | Navigation |

---

## Q5: Clean Architecture / TCA

**Clean Architecture** — layers with dependency rule (inner layers don't know about outer).

```
Entities → Use Cases → Interface Adapters → Frameworks
```

**The Composable Architecture (TCA)** — popular in SwiftUI:
```swift
@Reducer
struct CounterFeature {
    @ObservableState
    struct State {
        var count = 0
    }

    enum Action {
        case increment
        case decrement
    }

    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .increment:
                state.count += 1
                return .none
            case .decrement:
                state.count -= 1
                return .none
            }
        }
    }
}
```

---

## Q6: How do you choose an architecture?

| Project Size | Recommended |
|---|---|
| Small / prototype | MVC or simple MVVM |
| Medium | MVVM + Coordinator |
| Large / team | MVVM-C, Clean Architecture, or VIPER |
| SwiftUI-heavy | MVVM with @Observable, or TCA |

**Key principles regardless of architecture:**
- Single Responsibility Principle.
- Dependency Injection for testability.
- Protocol-based abstractions.
- Unidirectional data flow when possible.
