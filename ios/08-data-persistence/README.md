# 8. Data Persistence

## Q1: What are the data persistence options in iOS?

| Method | Best For | Complexity |
|---|---|---|
| `UserDefaults` | Small key-value pairs (settings, flags) | Low |
| `Keychain` | Sensitive data (tokens, passwords) | Medium |
| `FileManager` | Files, documents, images | Medium |
| `Core Data` | Complex object graphs, relationships | High |
| `SwiftData` | Modern Core Data replacement (iOS 17+) | Medium |
| `SQLite` | Direct SQL, lightweight DB | Medium |
| `CloudKit` | iCloud sync | High |

---

## Q2: UserDefaults — when and when not to use?

```swift
// Store
UserDefaults.standard.set("Alice", forKey: "username")
UserDefaults.standard.set(true, forKey: "hasOnboarded")

// Retrieve
let name = UserDefaults.standard.string(forKey: "username") ?? "Guest"
let onboarded = UserDefaults.standard.bool(forKey: "hasOnboarded")

// Custom object (must be Codable)
func save<T: Codable>(_ object: T, forKey key: String) {
    let data = try? JSONEncoder().encode(object)
    UserDefaults.standard.set(data, forKey: key)
}

// @AppStorage in SwiftUI
@AppStorage("username") var username = "Guest"
```

**Do NOT use for:** sensitive data, large data, frequently changing data, or anything over a few KB.

---

## Q3: How does Keychain work?

Keychain is the secure storage for sensitive data. Persists across app reinstalls.

```swift
import Security

func saveToKeychain(key: String, data: Data) -> Bool {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data
    ]

    SecItemDelete(query as CFDictionary) // Remove old value
    let status = SecItemAdd(query as CFDictionary, nil)
    return status == errSecSuccess
}

func loadFromKeychain(key: String) -> Data? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecReturnData as String: true,
        kSecMatchLimit as String: kSecMatchLimitOne
    ]

    var result: AnyObject?
    SecItemCopyMatching(query as CFDictionary, &result)
    return result as? Data
}
```

**Tip:** In production, use a wrapper library or create your own `KeychainManager` to simplify the API.

---

## Q4: Explain Core Data

Core Data is an object graph and persistence framework.

```swift
// Define model in .xcdatamodeld or code
// NSManagedObject subclass
@objc(Task)
class Task: NSManagedObject {
    @NSManaged var title: String
    @NSManaged var isCompleted: Bool
    @NSManaged var createdAt: Date
}

// Core Data Stack
class CoreDataStack {
    static let shared = CoreDataStack()

    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "Model")
        container.loadPersistentStores { _, error in
            if let error { fatalError("Core Data error: \(error)") }
        }
        return container
    }()

    var context: NSManagedObjectContext {
        persistentContainer.viewContext
    }

    func save() {
        guard context.hasChanges else { return }
        try? context.save()
    }
}

// CRUD operations
func createTask(title: String) {
    let task = Task(context: CoreDataStack.shared.context)
    task.title = title
    task.isCompleted = false
    task.createdAt = Date()
    CoreDataStack.shared.save()
}

func fetchTasks() -> [Task] {
    let request: NSFetchRequest<Task> = Task.fetchRequest()
    request.sortDescriptors = [NSSortDescriptor(keyPath: \Task.createdAt, ascending: false)]
    request.predicate = NSPredicate(format: "isCompleted == %@", NSNumber(value: false))
    return (try? CoreDataStack.shared.context.fetch(request)) ?? []
}
```

**Key concepts:**
- `NSManagedObjectContext` — scratchpad for changes.
- `NSPersistentStoreCoordinator` — bridges model and store.
- `NSFetchRequest` — queries with predicates and sort descriptors.
- `NSFetchedResultsController` — efficient UITableView/UICollectionView binding.
- Background contexts for heavy operations.

---

## Q5: What is SwiftData (iOS 17+)?

SwiftData is the modern replacement for Core Data using macros.

```swift
import SwiftData

@Model
class Task {
    var title: String
    var isCompleted: Bool
    var createdAt: Date

    init(title: String) {
        self.title = title
        self.isCompleted = false
        self.createdAt = Date()
    }
}

// Setup in App
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Task.self)
    }
}

// Usage in SwiftUI
struct TaskListView: View {
    @Query(sort: \Task.createdAt, order: .reverse) var tasks: [Task]
    @Environment(\.modelContext) var context

    var body: some View {
        List(tasks) { task in
            Text(task.title)
        }
    }

    func addTask(title: String) {
        let task = Task(title: title)
        context.insert(task)
    }
}
```

---

## Q6: FileManager — storing files

```swift
func saveImage(_ image: UIImage, name: String) throws -> URL {
    let data = image.jpegData(compressionQuality: 0.8)!
    let url = FileManager.default
        .urls(for: .documentDirectory, in: .userDomainMask)[0]
        .appendingPathComponent(name)
    try data.write(to: url)
    return url
}

func loadImage(name: String) -> UIImage? {
    let url = FileManager.default
        .urls(for: .documentDirectory, in: .userDomainMask)[0]
        .appendingPathComponent(name)
    guard let data = try? Data(contentsOf: url) else { return nil }
    return UIImage(data: data)
}
```

**Directories:**
- `.documentDirectory` — user-generated content, backed up.
- `.cachesDirectory` — re-creatable data, not backed up, system may purge.
- `.temporaryDirectory` — short-lived files.
