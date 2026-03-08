# 13. Performance & Optimization

## Q1: How do you profile an iOS app?

Use Xcode Instruments:

| Instrument | Detects |
|---|---|
| Time Profiler | CPU bottlenecks, slow functions |
| Allocations | Memory usage, growth over time |
| Leaks | Memory leaks |
| Core Animation | Rendering issues, offscreen rendering |
| Network | HTTP traffic, latency |
| Energy Log | Battery drain causes |

**Steps:** Product → Profile (⌘I) → Choose instrument → Record.

---

## Q2: UITableView / UICollectionView Performance

```swift
// 1. Reuse cells
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
    // configure cell
    return cell
}

// 2. Prefetching
extension ViewController: UITableViewDataSourcePrefetching {
    func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        for indexPath in indexPaths {
            imageLoader.prefetch(url: items[indexPath.row].imageURL)
        }
    }
}

// 3. Self-sizing cells — cache heights
var heightCache: [IndexPath: CGFloat] = [:]

func tableView(_ tableView: UITableView, estimatedHeightForRowAt indexPath: IndexPath) -> CGFloat {
    heightCache[indexPath] ?? 80
}

func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
    heightCache[indexPath] = cell.frame.height
}
```

**Other tips:**
- Avoid transparency and blending (use opaque backgrounds).
- Don't do work in `cellForRowAt` — prepare data beforehand.
- Use `DiffableDataSource` for efficient updates.

---

## Q3: Image Loading and Caching

```swift
actor ImageCache {
    static let shared = ImageCache()
    private var cache = NSCache<NSString, UIImage>()
    private var inFlightRequests: [URL: Task<UIImage, Error>] = [:]

    func image(for url: URL) async throws -> UIImage {
        let key = url.absoluteString as NSString

        // Check cache
        if let cached = cache.object(forKey: key) {
            return cached
        }

        // Deduplicate in-flight requests
        if let existing = inFlightRequests[url] {
            return try await existing.value
        }

        let task = Task<UIImage, Error> {
            let (data, _) = try await URLSession.shared.data(from: url)
            guard let image = UIImage(data: data) else {
                throw URLError(.cannotDecodeContentData)
            }
            // Downsample for display size
            let downsized = await downsample(image, to: CGSize(width: 200, height: 200))
            cache.setObject(downsized, forKey: key)
            return downsized
        }

        inFlightRequests[url] = task
        defer { inFlightRequests[url] = nil }
        return try await task.value
    }
}
```

---

## Q4: What is offscreen rendering and how to avoid it?

Offscreen rendering happens when the GPU can't render directly to the frame buffer. It's expensive.

**Common causes:**
- `cornerRadius` + `masksToBounds` / `clipsToBounds`
- Shadows without `shadowPath`
- `shouldRasterize`
- Masks and complex layer effects

```swift
// ❌ Slow — triggers offscreen rendering
view.layer.cornerRadius = 10
view.layer.masksToBounds = true

// ✅ Better — pre-render shadow path
view.layer.shadowPath = UIBezierPath(roundedRect: view.bounds, cornerRadius: 10).cgPath

// ✅ Use pre-rounded images instead of clipping
extension UIImage {
    func rounded(radius: CGFloat) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: size)
        return renderer.image { _ in
            let rect = CGRect(origin: .zero, size: size)
            UIBezierPath(roundedRect: rect, cornerRadius: radius).addClip()
            draw(in: rect)
        }
    }
}
```

---

## Q5: App Launch Time Optimization

**Target:** < 400ms for warm launch.

**Strategies:**
1. Reduce work in `didFinishLaunchingWithOptions` — defer non-essential setup.
2. Minimize static initializers and `+load` methods.
3. Reduce dynamic frameworks — merge or use static linking.
4. Use `os_signpost` to measure launch phases.
5. Lazy-load heavy resources.

```swift
// Defer non-critical initialization
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Only critical setup here
    setupCrashReporting()
    setupCoreData()

    // Defer everything else
    DispatchQueue.main.async {
        self.setupAnalytics()
        self.setupPushNotifications()
        self.preloadCache()
    }

    return true
}
```

---

## Q6: Memory Optimization Tips

1. Use `autoreleasepool` in tight loops.
2. Downsample images to display size (don't load full-res into memory).
3. Respond to `didReceiveMemoryWarning` — clear caches.
4. Use `NSCache` (auto-evicts under memory pressure) instead of `Dictionary`.
5. Profile with Allocations instrument to find growth.
6. Watch for retain cycles (Memory Graph Debugger).

```swift
// Downsample image to target size
func downsample(imageAt url: URL, to pointSize: CGSize, scale: CGFloat = UIScreen.main.scale) -> UIImage? {
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    guard let imageSource = CGImageSourceCreateWithURL(url as CFURL, imageSourceOptions) else { return nil }

    let maxDimension = max(pointSize.width, pointSize.height) * scale
    let downsampleOptions = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxDimension
    ] as CFDictionary

    guard let downsampledImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, downsampleOptions) else { return nil }
    return UIImage(cgImage: downsampledImage)
}
```
