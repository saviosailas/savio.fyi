# 16. System Design (iOS Mobile)

## Q1: How to approach a mobile system design interview?

**Framework (5 steps):**

1. **Clarify Requirements** — functional and non-functional.
2. **High-Level Architecture** — client components, API layer, data flow.
3. **Data Model** — entities, relationships, API contracts.
4. **Deep Dive** — key components in detail.
5. **Trade-offs & Optimizations** — caching, offline, performance.

---

## Q2: Design an Image Feed App (like Instagram)

**Requirements:**
- Scrollable feed of images with captions.
- Pull to refresh, infinite scroll.
- Image caching, offline support.
- Like/comment actions.

**Architecture:**

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   FeedView   │ ←→  │ FeedViewModel │ ←→  │ FeedService  │
└─────────────┘     └──────────────┘     └──────┬──────┘
                                                 │
                          ┌──────────────────────┼──────────────┐
                          │                      │              │
                    ┌─────┴─────┐    ┌──────────┴───┐   ┌─────┴──────┐
                    │ APIClient  │    │ ImageLoader   │   │ LocalCache  │
                    └───────────┘    └──────────────┘   └────────────┘
```

**Key decisions:**
- Image loading: multi-layer cache (memory → disk → network).
- Pagination: cursor-based (not offset) for consistency.
- Diffable data source for efficient UI updates.
- Prefetch images 3-5 cells ahead.
- Downsample images to display size to reduce memory.

**Data model:**
```swift
struct FeedItem: Identifiable, Codable {
    let id: String
    let imageURL: URL
    let caption: String
    let author: User
    let likeCount: Int
    let isLiked: Bool
    let createdAt: Date
}

struct FeedResponse: Codable {
    let items: [FeedItem]
    let nextCursor: String?
}
```

---

## Q3: Design a Chat App (like WhatsApp)

**Key components:**
- Real-time messaging via WebSocket.
- Local message storage (Core Data / SwiftData).
- Message states: sending → sent → delivered → read.
- Offline queue for messages sent without connectivity.

```
┌────────────┐     ┌───────────────┐     ┌──────────────┐
│  ChatView   │ ←→  │ ChatViewModel  │ ←→  │ MessageStore  │ (Core Data)
└────────────┘     └───────┬───────┘     └──────────────┘
                           │
                    ┌──────┴──────┐
                    │ WebSocket    │ ←→ Server
                    │ Manager      │
                    └─────────────┘
```

**Offline strategy:**
1. Save message locally with status `.sending`.
2. Attempt to send via WebSocket.
3. If offline, queue in `PendingMessageStore`.
4. On reconnect, flush the queue.
5. Update status on server acknowledgment.

---

## Q4: Design an Offline-First App

**Strategy: Cache-first with background sync.**

```swift
class OfflineFirstRepository<T: Codable & Identifiable> {
    private let localStore: LocalStore<T>
    private let remoteAPI: RemoteAPI<T>

    func getItems() async throws -> [T] {
        // 1. Return cached data immediately
        let cached = try localStore.fetchAll()
        if !cached.isEmpty {
            // 2. Refresh in background
            Task {
                let remote = try? await remoteAPI.fetchAll()
                if let remote {
                    try? localStore.sync(with: remote)
                }
            }
            return cached
        }

        // 3. No cache — fetch from network
        let remote = try await remoteAPI.fetchAll()
        try localStore.save(remote)
        return remote
    }
}
```

**Conflict resolution strategies:**
- Last-write-wins (simplest).
- Server-wins (safest).
- Client-wins (best offline UX).
- Merge (most complex, best for collaborative apps).

---

## Q5: Design a Video Streaming App

**Key considerations:**
- HLS (HTTP Live Streaming) for adaptive bitrate.
- AVPlayer for playback.
- Thumbnail generation and caching.
- Background downloads for offline viewing.
- Memory management — release players when off-screen.

---

## Q6: Common System Design Topics for Senior iOS

- Image feed with caching and pagination.
- Chat / messaging app.
- Offline-first architecture.
- Search with autocomplete and debouncing.
- Payment flow.
- Push notification system.
- Analytics SDK design.
- App modularization strategy.

**For each, discuss:**
- Architecture pattern choice and why.
- Data flow (unidirectional preferred).
- Error handling and retry strategies.
- Accessibility considerations.
- Testing strategy.
