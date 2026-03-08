# 17. Data Structures & Algorithms

## Q1: Arrays and Strings — Common Patterns

```swift
// Two Pointers
func isPalindrome(_ s: String) -> Bool {
    let chars = Array(s.lowercased().filter { $0.isLetter || $0.isNumber })
    var left = 0, right = chars.count - 1
    while left < right {
        if chars[left] != chars[right] { return false }
        left += 1
        right -= 1
    }
    return true
}

// Sliding Window — max sum of subarray of size k
func maxSubarraySum(_ nums: [Int], k: Int) -> Int {
    var windowSum = nums[0..<k].reduce(0, +)
    var maxSum = windowSum

    for i in k..<nums.count {
        windowSum += nums[i] - nums[i - k]
        maxSum = max(maxSum, windowSum)
    }
    return maxSum
}

// HashMap frequency count
func firstNonRepeating(_ s: String) -> Character? {
    var freq: [Character: Int] = [:]
    for char in s { freq[char, default: 0] += 1 }
    return s.first { freq[$0] == 1 }
}
```

---

## Q2: Linked Lists

```swift
class ListNode {
    var val: Int
    var next: ListNode?
    init(_ val: Int, _ next: ListNode? = nil) {
        self.val = val
        self.next = next
    }
}

// Reverse a linked list
func reverseList(_ head: ListNode?) -> ListNode? {
    var prev: ListNode? = nil
    var current = head
    while let node = current {
        let next = node.next
        node.next = prev
        prev = node
        current = next
    }
    return prev
}

// Detect cycle (Floyd's algorithm)
func hasCycle(_ head: ListNode?) -> Bool {
    var slow = head, fast = head
    while fast != nil && fast?.next != nil {
        slow = slow?.next
        fast = fast?.next?.next
        if slow === fast { return true }
    }
    return false
}
```

---

## Q3: Stacks and Queues

```swift
// Valid Parentheses
func isValid(_ s: String) -> Bool {
    var stack: [Character] = []
    let map: [Character: Character] = [")": "(", "]": "[", "}": "{"]

    for char in s {
        if let match = map[char] {
            if stack.isEmpty || stack.removeLast() != match { return false }
        } else {
            stack.append(char)
        }
    }
    return stack.isEmpty
}

// Min Stack
class MinStack {
    private var stack: [(val: Int, min: Int)] = []

    func push(_ val: Int) {
        let currentMin = stack.last?.min ?? Int.max
        stack.append((val, min(val, currentMin)))
    }

    func pop() { stack.removeLast() }
    func top() -> Int { stack.last!.val }
    func getMin() -> Int { stack.last!.min }
}
```

---

## Q4: Trees

```swift
class TreeNode {
    var val: Int
    var left: TreeNode?
    var right: TreeNode?
    init(_ val: Int) { self.val = val }
}

// BFS — Level Order Traversal
func levelOrder(_ root: TreeNode?) -> [[Int]] {
    guard let root else { return [] }
    var result: [[Int]] = []
    var queue: [TreeNode] = [root]

    while !queue.isEmpty {
        var level: [Int] = []
        for _ in 0..<queue.count {
            let node = queue.removeFirst()
            level.append(node.val)
            if let left = node.left { queue.append(left) }
            if let right = node.right { queue.append(right) }
        }
        result.append(level)
    }
    return result
}

// DFS — Max Depth
func maxDepth(_ root: TreeNode?) -> Int {
    guard let root else { return 0 }
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
}

// Invert Binary Tree
func invertTree(_ root: TreeNode?) -> TreeNode? {
    guard let root else { return nil }
    let temp = root.left
    root.left = invertTree(root.right)
    root.right = invertTree(temp)
    return root
}
```

---

## Q5: Hash Maps and Sets

```swift
// Two Sum
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var map: [Int: Int] = [:]
    for (i, num) in nums.enumerated() {
        if let j = map[target - num] { return [j, i] }
        map[num] = i
    }
    return []
}

// Group Anagrams
func groupAnagrams(_ strs: [String]) -> [[String]] {
    var groups: [[Character]: [String]] = [:]
    for str in strs {
        let key = str.sorted()
        groups[key, default: []].append(str)
    }
    return Array(groups.values)
}
```

---

## Q6: Sorting and Searching

```swift
// Binary Search
func binarySearch(_ nums: [Int], _ target: Int) -> Int {
    var left = 0, right = nums.count - 1
    while left <= right {
        let mid = left + (right - left) / 2
        if nums[mid] == target { return mid }
        else if nums[mid] < target { left = mid + 1 }
        else { right = mid - 1 }
    }
    return -1
}

// Merge Sort
func mergeSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }
    let mid = array.count / 2
    let left = mergeSort(Array(array[..<mid]))
    let right = mergeSort(Array(array[mid...]))
    return merge(left, right)
}

func merge(_ left: [Int], _ right: [Int]) -> [Int] {
    var result: [Int] = []
    var i = 0, j = 0
    while i < left.count && j < right.count {
        if left[i] <= right[j] { result.append(left[i]); i += 1 }
        else { result.append(right[j]); j += 1 }
    }
    result.append(contentsOf: left[i...])
    result.append(contentsOf: right[j...])
    return result
}
```

---

## Q7: Dynamic Programming

```swift
// Fibonacci (bottom-up)
func fib(_ n: Int) -> Int {
    guard n > 1 else { return n }
    var a = 0, b = 1
    for _ in 2...n {
        let temp = a + b
        a = b
        b = temp
    }
    return b
}

// Coin Change
func coinChange(_ coins: [Int], _ amount: Int) -> Int {
    var dp = Array(repeating: amount + 1, count: amount + 1)
    dp[0] = 0
    for i in 1...amount {
        for coin in coins {
            if coin <= i {
                dp[i] = min(dp[i], dp[i - coin] + 1)
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount]
}

// Longest Common Subsequence
func longestCommonSubsequence(_ text1: String, _ text2: String) -> Int {
    let s1 = Array(text1), s2 = Array(text2)
    var dp = Array(repeating: Array(repeating: 0, count: s2.count + 1), count: s1.count + 1)

    for i in 1...s1.count {
        for j in 1...s2.count {
            if s1[i-1] == s2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    return dp[s1.count][s2.count]
}
```

---

## Q8: Big-O Complexity Cheat Sheet

| Algorithm | Time | Space |
|---|---|---|
| Array access | O(1) | — |
| Array search | O(n) | — |
| Binary search | O(log n) | O(1) |
| Hash map lookup | O(1) avg | O(n) |
| Merge sort | O(n log n) | O(n) |
| Quick sort | O(n log n) avg | O(log n) |
| BFS/DFS | O(V + E) | O(V) |
| Dynamic programming | Varies | Varies |

**Swift-specific:** `Array.contains` is O(n), `Set.contains` is O(1). Use `Set` for membership checks.
