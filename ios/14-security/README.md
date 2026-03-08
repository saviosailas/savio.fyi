# 14. Security

## Q1: How do you store sensitive data securely?

| Data Type | Storage | Why |
|---|---|---|
| Auth tokens | Keychain | Encrypted, persists across reinstalls |
| API keys | Keychain or obfuscation | Never hardcode in source |
| User preferences | UserDefaults | Not sensitive |
| Passwords | Keychain | Never store plaintext |
| Encryption keys | Keychain with `.whenUnlockedThisDeviceOnly` | Hardware-backed security |

**Never store sensitive data in:**
- UserDefaults (plist, easily readable)
- Files without encryption
- Source code / Info.plist

---

## Q2: What is App Transport Security (ATS)?

ATS enforces HTTPS for all network connections by default.

```xml
<!-- Info.plist — only if you MUST allow HTTP (not recommended) -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

App Review may reject apps that disable ATS without justification.

---

## Q3: What is Keychain Access Control?

```swift
// Require biometric authentication to access
let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly,
    .biometryCurrentSet,
    nil
)

let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "authToken",
    kSecValueData as String: tokenData,
    kSecAttrAccessControl as String: accessControl as Any
]

SecItemAdd(query as CFDictionary, nil)
```

**Accessibility levels:**
- `kSecAttrAccessibleWhenUnlocked` — default, available when device unlocked.
- `kSecAttrAccessibleAfterFirstUnlock` — available after first unlock (background apps).
- `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly` — requires passcode, no backup migration.

---

## Q4: How do you prevent reverse engineering?

1. **Code obfuscation** — rename symbols, encrypt strings.
2. **Jailbreak detection** — check for Cydia, suspicious paths, writable system dirs.
3. **SSL pinning** — prevent MITM proxy tools.
4. **Integrity checks** — detect debugger attachment, code tampering.
5. **Avoid storing secrets client-side** — use server-side validation.

```swift
// Basic jailbreak detection
func isJailbroken() -> Bool {
    #if targetEnvironment(simulator)
    return false
    #else
    let paths = [
        "/Applications/Cydia.app",
        "/usr/sbin/sshd",
        "/etc/apt",
        "/private/var/lib/apt/"
    ]
    for path in paths {
        if FileManager.default.fileExists(atPath: path) { return true }
    }
    // Check if app can write outside sandbox
    let testPath = "/private/test_jailbreak.txt"
    do {
        try "test".write(toFile: testPath, atomically: true, encoding: .utf8)
        try FileManager.default.removeItem(atPath: testPath)
        return true
    } catch {
        return false
    }
    #endif
}
```

---

## Q5: Biometric Authentication (Face ID / Touch ID)

```swift
import LocalAuthentication

func authenticateWithBiometrics() async -> Bool {
    let context = LAContext()
    var error: NSError?

    guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
        return false
    }

    do {
        return try await context.evaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            localizedReason: "Authenticate to access your account"
        )
    } catch {
        return false
    }
}
```

**Info.plist required:** `NSFaceIDUsageDescription`

---

## Q6: Data Encryption

```swift
import CryptoKit

// AES-GCM encryption
func encrypt(data: Data, key: SymmetricKey) throws -> Data {
    let sealedBox = try AES.GCM.seal(data, using: key)
    return sealedBox.combined!
}

func decrypt(data: Data, key: SymmetricKey) throws -> Data {
    let sealedBox = try AES.GCM.SealedBox(combined: data)
    return try AES.GCM.open(sealedBox, using: key)
}

// Generate key
let key = SymmetricKey(size: .bits256)

// Hashing
let hash = SHA256.hash(data: someData)
let hashString = hash.compactMap { String(format: "%02x", $0) }.joined()
```

---

## Q7: Common Security Mistakes

1. Logging sensitive data (tokens, passwords) in production.
2. Hardcoding API keys or secrets in source code.
3. Not validating server certificates (disabling ATS).
4. Storing tokens in UserDefaults.
5. Not using HTTPS.
6. Trusting client-side validation only.
7. Not clearing sensitive data from memory when done.
