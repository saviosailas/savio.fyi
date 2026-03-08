# 12. App Lifecycle & System Frameworks

## Q1: Explain the iOS App Lifecycle

```
Not Running → Inactive → Active → Background → Suspended → Terminated
```

| State | Description |
|---|---|
| Not Running | App hasn't been launched or was terminated |
| Inactive | Running in foreground but not receiving events (e.g., incoming call) |
| Active | Running in foreground, receiving events |
| Background | Running code but not visible (limited time ~30s) |
| Suspended | In memory but not executing code |

```swift
// UIKit — AppDelegate
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // App launched
        return true
    }
}

// UIKit — SceneDelegate (iOS 13+)
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    func sceneDidBecomeActive(_ scene: UIScene) { }
    func sceneWillResignActive(_ scene: UIScene) { }
    func sceneDidEnterBackground(_ scene: UIScene) { }
    func sceneWillEnterForeground(_ scene: UIScene) { }
}

// SwiftUI
@main
struct MyApp: App {
    @Environment(\.scenePhase) var scenePhase

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { _, newPhase in
            switch newPhase {
            case .active: print("Active")
            case .inactive: print("Inactive")
            case .background: print("Background")
            @unknown default: break
            }
        }
    }
}
```

---

## Q2: What is the difference between AppDelegate and SceneDelegate?

- `AppDelegate` — app-level events (launch, push notifications, background fetch).
- `SceneDelegate` — per-window/scene lifecycle (iOS 13+ for multi-window on iPad).

With SwiftUI's `@main` App protocol, both are often unnecessary unless you need specific delegate callbacks.

---

## Q3: Background Execution Modes

```swift
// Info.plist — UIBackgroundModes
// audio, location, fetch, remote-notification, processing

// Background Tasks (iOS 13+)
import BackgroundTasks

func scheduleBackgroundFetch() {
    let request = BGAppRefreshTaskRequest(identifier: "com.app.refresh")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)
    try? BGTaskScheduler.shared.submit(request)
}

// Register in AppDelegate
BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.app.refresh", using: nil) { task in
    self.handleAppRefresh(task: task as! BGAppRefreshTask)
}
```

---

## Q4: Push Notifications

```swift
// Request permission
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, error in
    if granted {
        DispatchQueue.main.async {
            UIApplication.shared.registerForRemoteNotifications()
        }
    }
}

// AppDelegate callbacks
func application(_ application: UIApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
    // Send token to your server
}

// Handle notification
func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {
    let userInfo = response.notification.request.content.userInfo
    // Handle deep link
    completionHandler()
}
```

---

## Q5: Deep Linking and Universal Links

```swift
// URL Scheme — myapp://profile/123
// Universal Links — https://example.com/profile/123

// Handle in SceneDelegate
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url else { return }
    handleDeepLink(url)
}

func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard let url = userActivity.webpageURL else { return }
    handleDeepLink(url)
}

func handleDeepLink(_ url: URL) {
    let components = URLComponents(url: url, resolvingAgainstBaseURL: true)
    guard let path = components?.path else { return }

    switch path {
    case let p where p.hasPrefix("/profile/"):
        let id = String(p.dropFirst("/profile/".count))
        navigateToProfile(id: id)
    default:
        break
    }
}
```

---

## Q6: What are common System Frameworks?

| Framework | Purpose |
|---|---|
| Foundation | Data types, collections, networking basics |
| UIKit | UI components, event handling |
| SwiftUI | Declarative UI |
| Combine | Reactive programming |
| CoreLocation | GPS, geofencing |
| MapKit | Maps and annotations |
| AVFoundation | Audio/video playback and recording |
| CoreAnimation | Advanced animations |
| CoreImage | Image processing and filters |
| StoreKit | In-app purchases |
| WidgetKit | Home screen widgets |
| ActivityKit | Live Activities and Dynamic Island |
