# 6. SwiftUI

## Q1: How does SwiftUI's declarative approach differ from UIKit?

| UIKit (Imperative) | SwiftUI (Declarative) |
|---|---|
| You tell HOW to update UI | You describe WHAT the UI should look like |
| Manual state → UI sync | Automatic state → UI sync |
| ViewControllers + Storyboards | View structs + modifiers |
| Delegate/DataSource patterns | Data-driven with property wrappers |

```swift
// SwiftUI — declare the desired state
struct ContentView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") { count += 1 }
        }
    }
}
```

---

## Q2: Explain SwiftUI State Management

```swift
// @State — local view state (value types)
@State private var isOn = false

// @Binding — two-way connection to parent's state
struct ToggleRow: View {
    @Binding var isOn: Bool
    var body: some View { Toggle("Enable", isOn: $isOn) }
}

// @StateObject — owns an ObservableObject (created once)
@StateObject private var viewModel = MyViewModel()

// @ObservedObject — observes an ObservableObject (passed in, not owned)
@ObservedObject var viewModel: MyViewModel

// @EnvironmentObject — shared object from ancestor
@EnvironmentObject var settings: AppSettings

// @Environment — system environment values
@Environment(\.colorScheme) var colorScheme
```

**Rule of thumb:**
- Use `@StateObject` when the view creates the object.
- Use `@ObservedObject` when the object is passed in.
- Use `@EnvironmentObject` for deeply shared state.

---

## Q3: What is the `@Observable` macro (iOS 17+)?

Replaces `ObservableObject` + `@Published` with simpler syntax.

```swift
// Old way
class ViewModel: ObservableObject {
    @Published var name = ""
    @Published var items: [Item] = []
}

// New way (iOS 17+)
@Observable
class ViewModel {
    var name = ""
    var items: [Item] = []
}

// Usage — no @StateObject/@ObservedObject needed
struct ContentView: View {
    @State private var viewModel = ViewModel()

    var body: some View {
        Text(viewModel.name)
    }
}
```

---

## Q4: How does SwiftUI view identity and diffing work?

SwiftUI uses two types of identity:

1. **Structural identity** — position in the view hierarchy.
2. **Explicit identity** — using `.id()` or `ForEach` with `Identifiable`.

```swift
// ⚠️ Bad — SwiftUI can't track identity
ForEach(0..<items.count, id: \.self) { i in
    Text(items[i].name)
}

// ✅ Good — stable identity
ForEach(items) { item in  // item conforms to Identifiable
    Text(item.name)
}
```

SwiftUI compares the view tree before and after state changes, only re-rendering what changed.

---

## Q5: What are View Modifiers?

Modifiers return a new view wrapping the original. Order matters.

```swift
Text("Hello")
    .padding()           // padding first
    .background(.blue)   // background covers padding
    .cornerRadius(10)

// vs
Text("Hello")
    .background(.blue)   // background only covers text
    .padding()           // padding outside background
```

**Custom modifier:**
```swift
struct CardStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(.white)
            .cornerRadius(12)
            .shadow(radius: 4)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardStyle())
    }
}

// Usage
Text("Card").cardStyle()
```

---

## Q6: Navigation in SwiftUI

```swift
// iOS 16+ NavigationStack
struct ContentView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            List(items) { item in
                NavigationLink(value: item) {
                    Text(item.name)
                }
            }
            .navigationDestination(for: Item.self) { item in
                DetailView(item: item)
            }
        }
    }
}
```

---

## Q7: How do you integrate UIKit views in SwiftUI?

```swift
// UIViewRepresentable
struct MapView: UIViewRepresentable {
    func makeUIView(context: Context) -> MKMapView {
        MKMapView()
    }

    func updateUIView(_ uiView: MKMapView, context: Context) {
        // Update the view
    }
}

// UIViewControllerRepresentable
struct ImagePicker: UIViewControllerRepresentable {
    @Binding var image: UIImage?

    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.delegate = context.coordinator
        return picker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let parent: ImagePicker
        init(_ parent: ImagePicker) { self.parent = parent }

        func imagePickerController(_ picker: UIImagePickerController,
                                   didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]) {
            parent.image = info[.originalImage] as? UIImage
        }
    }
}
```

---

## Q8: What are SwiftUI Previews and how do you use them effectively?

```swift
#Preview {
    ContentView()
        .environmentObject(AppSettings())
}

#Preview("Dark Mode") {
    ContentView()
        .preferredColorScheme(.dark)
}

#Preview("Large Text") {
    ContentView()
        .environment(\.sizeCategory, .accessibilityExtraLarge)
}
```
