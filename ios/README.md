# iOS Developer Interview Preparation Guide

**Experience Level:** Senior iOS Developer (7.6 years)

## Topics

1. [Swift Language Fundamentals](./01-swift-fundamentals/README.md)
2. [Object-Oriented & Protocol-Oriented Programming](./02-oop-and-pop/README.md)
3. [Memory Management](./03-memory-management/README.md)
4. [Concurrency & Multithreading](./04-concurrency/README.md)
5. [UIKit](./05-uikit/README.md)
6. [SwiftUI](./06-swiftui/README.md)
7. [Networking](./07-networking/README.md)
8. [Data Persistence](./08-data-persistence/README.md)
9. [Architecture Patterns](./09-architecture-patterns/README.md)
10. [Design Patterns](./10-design-patterns/README.md)
11. [Testing](./11-testing/README.md)
12. [App Lifecycle & System Frameworks](./12-app-lifecycle/README.md)
13. [Performance & Optimization](./13-performance/README.md)
14. [Security](./14-security/README.md)
15. [CI/CD & App Distribution](./15-cicd/README.md)
16. [System Design](./16-system-design/README.md)
17. [Data Structures & Algorithms](./17-dsa/README.md)
18. [Behavioral & Leadership Questions](./18-behavioral/README.md)
19. [Swift 6 — What's New and What Changed](./19-swift-6/README.md)

---

> Study each folder in order. Each topic contains interview questions, answers, and code examples.

---

# Senior iOS Interview Preparation Plan

## Priority Map (Highest to Lowest)

Given a large-team background where you owned a narrow slice of the codebase, these areas likely have the biggest gaps:

| Priority | Topic | Why It Matters for Senior |
|---|---|---|
| Critical | Architecture (09) | You'll be asked to design systems from scratch |
| Critical | Concurrency (04) | Most senior interviews heavily test async/await + actors |
| Critical | Memory Management (03) | ARC internals, retain cycles, Instruments |
| Critical | DSA (17) | Coding round — 60-90 min live coding |
| High | Networking (07) | URLSession, Combine, error handling at scale |
| High | Testing (11) | Senior = owns quality; unit + UI + async testing |
| High | System Design (16) | Often skipped but asked at senior level |
| Medium | SwiftUI (06) | Modern apps; state management depth |
| Medium | Performance (13) | Instruments, profiling, optimization |
| Baseline | Swift Fundamentals (01) | Review edge cases |

---

## Phase 1: Foundation Review (Week 1)

### Day 1-2: Swift Fundamentals + Memory Management

**What to do:**
- Re-read `01-swift-fundamentals` and `03-memory-management`
- Write every code example by hand in a Playground — do not just read
- For memory management, draw ARC retain graphs on paper

**Key things you must be able to explain without notes:**
- Why `[weak self]` vs `[unowned self]` — when each crashes
- What happens to a strong reference cycle when the object goes out of scope
- Stack vs Heap allocation — why structs can still go on the heap
- `deinit` — when it is called and when it is NOT called (retain cycle)

**Practice question to answer out loud:**
> "Walk me through what happens in memory when this closure captures self strongly."

---

### Day 3-4: Concurrency Deep Dive

**What to do:**
- Re-read `04-concurrency`
- Build a small project: fetch 3 APIs in parallel with `TaskGroup`, update UI with `@MainActor`
- Understand the GCD to async/await migration path

**Must know cold:**
```
GCD:       DispatchQueue, DispatchGroup, DispatchBarrier, Semaphore
Modern:    async/await, Task, TaskGroup, Actor, @MainActor, Sendable
Problems:  Race condition, Deadlock, Priority inversion, Thread explosion
```

**Senior-level question you will get:**
> "How would you migrate a completion-handler-based networking layer to async/await without breaking existing callers?"

Answer: Use `withCheckedContinuation` or `withCheckedThrowingContinuation` as a bridge layer.

---

### Day 5-7: Architecture + Design Patterns

**What to do:**
- Re-read `09-architecture-patterns` and `10-design-patterns`
- Pick one architecture (MVVM + Coordinator recommended) and build a 3-screen mini-app: Login → List → Detail
- Be able to draw each pattern on a whiteboard with arrows

**The question you WILL be asked:**
> "Tell me about a time you refactored architecture in a large team. What did you choose and why?"

Even if you did not own it, you need an answer. Prepare a story about a technical decision you observed or influenced.

---

## Phase 2: Coding Interview Prep (Week 2)

### LeetCode Strategy for iOS Interviews

Senior iOS coding rounds focus on medium-difficulty problems. You do not need hard DP or graph theory — you need clean, readable Swift.

**Daily routine:**
- 2 problems/day on LeetCode in Swift
- Time yourself: 25 min per problem
- After solving, ask: "What is the optimal solution?" and "What is the space complexity?"

**Must-solve problem list by category:**

| Category | Problems |
|---|---|
| Arrays/Strings | Two Sum, Valid Anagram, Longest Substring Without Repeating |
| Two Pointers | 3Sum, Container With Most Water, Move Zeroes |
| Sliding Window | Maximum Subarray, Permutation in String |
| Linked List | Reverse LL, Merge Two Sorted Lists, LRU Cache |
| Stack/Queue | Valid Parentheses, Min Stack, Daily Temperatures |
| Trees | Max Depth, Invert Binary Tree, Validate BST, LCA |
| Binary Search | Search in Rotated Array, Find Min in Rotated Array |
| Dynamic Programming | Climbing Stairs, Coin Change, House Robber |
| HashMap | Group Anagrams, Top K Frequent, Subarray Sum Equals K |

**Swift-specific things to practice:**

```swift
// String manipulation in Swift is verbose — practice this
let chars = Array(s)           // String → [Character]
String(chars)                  // [Character] → String
s.sorted()                     // Sort characters
s.split(separator: " ")        // Split

// Dictionary with default
freq[char, default: 0] += 1

// Sorting custom objects
arr.sorted { $0.val < $1.val }
```

### Live Coding Rules (what interviewers look for)

1. **Think out loud** — narrate your approach before writing code
2. **Start with brute force** — then optimize: "naive solution is O(n²), we can do O(n) with a hashmap"
3. **Name variables clearly** — `left/right` not `l/r`, `current` not `cur`
4. **Handle edge cases verbally** — empty array, single element, nil
5. **Test with an example** — trace through your code with input `[2,7,11,15], target=9`

---

## Phase 3: System Design (Week 3)

### How to Answer iOS System Design Questions

Framework for any design question:

```
1. Clarify requirements (2 min)
   - Scale? 1M users? 10M?
   - Offline support needed?
   - Real-time updates?

2. Define data models (3 min)
   - What structs/entities exist?

3. Architecture decision (5 min)
   - Which pattern? Why for this use case?

4. Key components (10 min)
   - Networking layer
   - Caching/persistence
   - State management
   - Error handling

5. Trade-offs (5 min)
   - What did you optimize for?
   - What would you do differently at 10x scale?
```

**Common system design questions:**
- Design a photo feed (like Instagram)
- Design an offline-capable notes app
- Design a real-time chat feature
- Design an image caching system

**Example answer for image caching:**
> "I'd use a two-level cache: in-memory NSCache (auto-evicts under memory pressure) with a disk cache using FileManager. Keys are URL-derived hashes. Images are loaded on background queues and assigned on MainActor. For preloading, I'd use URLSession's built-in caching with `.reloadIgnoringLocalCacheData` for forced refreshes. NSCache's `countLimit` and `totalCostLimit` prevent memory spikes."

---

## Phase 4: Face-to-Face Behavioral (Week 3-4)

### The STAR Format

Every behavioral answer: **Situation → Task → Action → Result**

**Questions you will be asked at senior level:**

| Question | What They're Testing |
|---|---|
| "Tell me about a technical conflict with a teammate" | Leadership, communication |
| "How did you handle a production bug?" | Incident response, ownership |
| "Describe a time you influenced without authority" | Senior-level impact |
| "How do you mentor junior developers?" | Leadership |
| "Tell me about the most complex feature you shipped" | Technical depth |
| "How do you handle tech debt?" | Engineering judgment |

**Prepare 5-6 stories.** Each story should be reusable for multiple questions. Cover:
1. A technical decision you made (and its trade-offs)
2. A time you disagreed with a direction but executed anyway
3. A production incident you handled or learned from
4. A time you improved team process/quality
5. A complex feature with ambiguous requirements

**For large-team experience — be honest about scope:**
> "I owned the [payments / search / profile] module. I became the domain expert for that area, which gave me deep expertise in [X]. Here's what I learned from working alongside other modules..."

Do not pretend you built everything. Senior interviewers respect focused ownership.

---

## Phase 5: Topics to Deepen

Your READMEs cover the basics. For each topic below, think through a real-world example from your own experience.

### Networking — key senior questions:
- How do you handle token refresh with concurrent requests? (Use actor + continuation)
- How do you implement retry logic with exponential backoff?
- Certificate pinning — how and why?

### Testing — critical gap area:
- Can you write a unit test for an async function?
- How do you mock a network dependency?
- What is the difference between unit, integration, and UI tests?

```swift
// Async testing pattern — know this cold
func testFetchUser() async throws {
    let mockService = MockUserService()
    mockService.stubbedUser = User(name: "Test")
    let vm = UserViewModel(service: mockService)

    await vm.loadUser(id: 1)

    XCTAssertEqual(vm.displayName, "Test")
    XCTAssertFalse(vm.isLoading)
}
```

### Performance:
- Instruments: Time Profiler, Leaks, Allocations — know what each shows
- `os_signpost` for custom performance logging
- View hierarchy debugging with Debug View Hierarchy

---

## Weekly Schedule

```
Week 1:  Swift + Memory + Concurrency + Architecture review
         Read READMEs → Build mini projects → Explain out loud

Week 2:  LeetCode (2/day in Swift) + Networking + Testing
         Medium problems → Practice talking through solutions

Week 3:  System Design + Behavioral prep
         Write out 5 STAR stories → Practice system design on paper

Week 4:  Mock interviews + weak area review
         Timed mock coding sessions → Review anything you fumbled
```

---

## Most Common Senior iOS Interview Mistakes

1. **Can't explain trade-offs** — knowing *what* isn't enough; know *when* and *why*
2. **Silent during coding** — always narrate
3. **Skips edge cases** — always ask "what if the array is empty?"
4. **Over-engineers behavioral answers** — be specific and concise
5. **Can't answer "why did your team choose X?"** — even if you didn't make the decision, have an informed opinion

---

## Daily Checklist

- [ ] 1 LeetCode medium problem (timed, narrated out loud)
- [ ] Read 1 topic README and explain it to yourself without looking
- [ ] Review 1 behavioral story and refine it
- [ ] 1 system design sketch on paper (5 min freehand)
