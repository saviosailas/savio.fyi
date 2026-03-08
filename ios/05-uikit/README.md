# 5. UIKit

## Q1: Explain the UIViewController Lifecycle

```
init → loadView → viewDidLoad → viewWillAppear → viewDidAppear
                                                        ↓
                              viewDidDisappear ← viewWillDisappear
```

| Method | When Called | Common Use |
|---|---|---|
| `viewDidLoad` | Once, after view loaded into memory | Initial setup, add subviews |
| `viewWillAppear` | Every time before view becomes visible | Refresh data, start animations |
| `viewDidAppear` | After view is visible on screen | Start timers, analytics |
| `viewWillDisappear` | Before view is removed from screen | Save state, pause tasks |
| `viewDidDisappear` | After view is no longer visible | Stop timers, cleanup |

---

## Q2: What is the difference between `frame` and `bounds`?

- `frame` — position and size relative to the superview's coordinate system.
- `bounds` — position and size relative to its own coordinate system.

```swift
let view = UIView()
view.frame = CGRect(x: 50, y: 100, width: 200, height: 200)
// view.bounds = CGRect(x: 0, y: 0, width: 200, height: 200)

// Changing bounds.origin affects subview positioning (like scrolling)
view.bounds.origin = CGPoint(x: 0, y: 50) // subviews shift up by 50
```

---

## Q3: Auto Layout — How does it work?

Auto Layout uses constraints to define relationships between views.

```swift
// Programmatic constraints
let label = UILabel()
label.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(label)

NSLayoutConstraint.activate([
    label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    label.centerYAnchor.constraint(equalTo: view.centerYAnchor),
    label.widthAnchor.constraint(lessThanOrEqualTo: view.widthAnchor, multiplier: 0.8)
])
```

**Key concepts:**
- `intrinsicContentSize` — natural size of a view (labels, buttons).
- Content Hugging Priority — resistance to growing beyond intrinsic size.
- Compression Resistance Priority — resistance to shrinking below intrinsic size.
- Ambiguous layout — not enough constraints.
- Conflicting constraints — too many constraints.

---

## Q4: UITableView vs UICollectionView

| UITableView | UICollectionView |
|---|---|
| Single column, vertical scroll | Flexible grid, any direction |
| Built-in cell styles | Fully custom layouts |
| Simpler for lists | Compositional layouts (iOS 13+) |

**Modern approach — Diffable Data Source:**
```swift
var dataSource: UITableViewDiffableDataSource<Section, Item>!

dataSource = UITableViewDiffableDataSource(tableView: tableView) { tableView, indexPath, item in
    let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
    cell.textLabel?.text = item.title
    return cell
}

var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
snapshot.appendSections([.main])
snapshot.appendItems(items)
dataSource.apply(snapshot, animatingDifferences: true)
```

---

## Q5: What is the Responder Chain?

The responder chain is a series of `UIResponder` objects that handle events.

```
UIView → UIViewController → UIWindow → UIApplication → AppDelegate
```

When a touch event occurs:
1. Hit testing finds the deepest view (first responder).
2. If it can't handle the event, it passes up the responder chain.
3. Continues until handled or reaches `UIApplication`.

```swift
// Custom event handling
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    // Handle or call super to pass up the chain
    super.touchesBegan(touches, with: event)
}
```

---

## Q6: Explain UIView animation approaches

```swift
// 1. UIView.animate
UIView.animate(withDuration: 0.3, delay: 0, options: .curveEaseInOut) {
    self.myView.alpha = 0
    self.myView.transform = CGAffineTransform(scaleX: 0.5, y: 0.5)
} completion: { _ in
    self.myView.removeFromSuperview()
}

// 2. UIViewPropertyAnimator (interruptible)
let animator = UIViewPropertyAnimator(duration: 0.5, curve: .easeOut) {
    self.myView.center = newCenter
}
animator.startAnimation()
animator.pauseAnimation()       // can pause
animator.fractionComplete = 0.5 // scrub to 50%

// 3. Core Animation (CALayer)
let animation = CABasicAnimation(keyPath: "position.x")
animation.fromValue = 0
animation.toValue = 300
animation.duration = 1.0
layer.add(animation, forKey: "move")
```

---

## Q7: What is `layoutSubviews` vs `setNeedsLayout` vs `layoutIfNeeded`?

- `layoutSubviews()` — called by the system to layout subviews. Override for custom layout.
- `setNeedsLayout()` — marks the view as needing layout (batched, next cycle).
- `layoutIfNeeded()` — forces immediate layout if needed.

```swift
// Animate constraint changes
self.topConstraint.constant = 100
UIView.animate(withDuration: 0.3) {
    self.view.layoutIfNeeded() // Animate the layout change
}
```

---

## Q8: What is the difference between `addSubview` and `insertSubview`?

```swift
view.addSubview(childView)                          // adds on top
view.insertSubview(childView, at: 0)                // adds at bottom
view.insertSubview(childView, belowSubview: other)  // below specific view
view.insertSubview(childView, aboveSubview: other)  // above specific view
view.bringSubviewToFront(childView)                 // move to top
view.sendSubviewToBack(childView)                   // move to bottom
```
