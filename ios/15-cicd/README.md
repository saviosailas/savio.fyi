# 15. CI/CD & App Distribution

## Q1: What is the iOS code signing process?

```
Developer Certificate + Provisioning Profile + App ID + Entitlements = Signed App
```

| Component | Purpose |
|---|---|
| Certificate | Proves your identity (Development or Distribution) |
| App ID | Unique identifier (Bundle ID) |
| Provisioning Profile | Links certificate + App ID + devices |
| Entitlements | App capabilities (push, iCloud, etc.) |

**Types of provisioning profiles:**
- Development — for testing on registered devices.
- Ad Hoc — for distribution to specific devices (up to 100).
- App Store — for App Store / TestFlight submission.
- Enterprise — for in-house distribution (requires Enterprise account).

---

## Q2: What is Fastlane?

Fastlane automates building, testing, and releasing iOS apps.

```ruby
# Fastfile
default_platform(:ios)

platform :ios do
  desc "Run tests"
  lane :test do
    run_tests(scheme: "MyApp")
  end

  desc "Build and upload to TestFlight"
  lane :beta do
    increment_build_number
    build_app(scheme: "MyApp")
    upload_to_testflight
  end

  desc "Release to App Store"
  lane :release do
    build_app(scheme: "MyApp")
    upload_to_app_store(
      skip_metadata: false,
      skip_screenshots: false
    )
  end
end
```

**Common Fastlane tools:**
- `match` — sync certificates and profiles across team.
- `snapshot` — automated screenshots.
- `deliver` — upload metadata and builds.
- `scan` — run tests.
- `gym` — build the app.

---

## Q3: Xcode Cloud vs GitHub Actions vs Other CI

| Feature | Xcode Cloud | GitHub Actions | Bitrise |
|---|---|---|---|
| Setup | Built into Xcode | YAML config | Web UI + YAML |
| macOS runners | Apple-managed | GitHub-hosted or self-hosted | Cloud-hosted |
| Cost | 25 hrs/month free | 2000 min/month free | Limited free tier |
| Integration | Deep Xcode/ASC integration | Flexible, any workflow | iOS-focused |

**GitHub Actions example:**
```yaml
name: iOS CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: macos-14
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: xcodebuild build -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
      - name: Test
        run: xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
```

---

## Q4: What is TestFlight?

TestFlight is Apple's beta testing platform.

- Internal testers: up to 100 App Store Connect users, no review needed.
- External testers: up to 10,000, requires Beta App Review.
- Builds expire after 90 days.
- Testers get automatic update notifications.
- Crash logs and feedback collected automatically.

---

## Q5: App Store Review Guidelines — Common Rejections

1. **Crashes and bugs** — test thoroughly on real devices.
2. **Broken links** — all URLs must work.
3. **Incomplete metadata** — screenshots, descriptions, privacy policy.
4. **Privacy** — must declare data collection in App Privacy.
5. **Login required** — provide demo credentials for review.
6. **In-app purchases** — must use Apple's IAP for digital goods.
7. **Minimum functionality** — app must provide value beyond a website.

---

## Q6: What is Swift Package Manager (SPM)?

```swift
// Package.swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "MyLibrary",
    platforms: [.iOS(.v16)],
    products: [
        .library(name: "MyLibrary", targets: ["MyLibrary"])
    ],
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.8.0")
    ],
    targets: [
        .target(name: "MyLibrary", dependencies: ["Alamofire"]),
        .testTarget(name: "MyLibraryTests", dependencies: ["MyLibrary"])
    ]
)
```

**SPM vs CocoaPods vs Carthage:**
- SPM — Apple-native, integrated in Xcode, preferred for new projects.
- CocoaPods — most libraries supported, centralized spec repo, modifies project.
- Carthage — decentralized, builds frameworks, less intrusive.
