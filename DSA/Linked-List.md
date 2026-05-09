# 🔗 LINKED LIST MASTERY — 20-Hour Complete Course
### 10 Sessions × 2 Hours | Interview-Focused | Beginner → Expert

---

# TABLE OF CONTENTS

```
SESSION 1  — Linked List Fundamentals
SESSION 2  — Insertions, Deletions, Search
SESSION 3  — Fast & Slow Pointer Pattern
SESSION 4  — Reversal Patterns
SESSION 5  — Two Pointer Interview Patterns
SESSION 6  — Dummy Node Mastery
SESSION 7  — Doubly & Circular Linked Lists
SESSION 8  — Advanced Linked List Problems
SESSION 9  — Linked List + Recursion
SESSION 10 — Interview Mastery & Mixed Problems

FINAL SECTION:
  — Complete Revision Sheet
  — Interview Q&A Bank
  — Mini Projects
  — Advanced Challenges
  — Mastery Checklist
  — Next Learning Path
```

---

---

# ╔══════════════════════════════════════════╗
# ║  SESSION 1 — LINKED LIST FUNDAMENTALS   ║
# ╚══════════════════════════════════════════╝

---

## 1.1 — What is a Linked List?

### Concept Explanation

**What it is:**
A Linked List is a linear data structure where elements (called **nodes**) are stored in memory at non-contiguous locations. Each node holds two things:
1. **Data** — the actual value stored
2. **Next pointer** — a reference/address pointing to the next node in the sequence

Unlike arrays, elements in a linked list are NOT laid out side-by-side in memory. Instead, each node independently lives somewhere in memory and points to the next one, like a chain.

**Why it exists:**
Arrays have a fundamental limitation: their size must be known upfront (in many languages) and insertion/deletion in the middle requires shifting elements — an O(n) operation. Linked lists solve this by:
- Allowing **dynamic resizing** (no preallocated size needed)
- Making insertion and deletion at known positions **O(1)** by simply redirecting pointers
- Trading random access (array's O(1)) for flexibility

**When to use it:**
- When you frequently insert/delete at the beginning or middle
- When the size of the data collection is unknown upfront
- When you're implementing stacks, queues, or adjacency lists
- When you don't need random access by index

**Real-world relevance:**
- Browser history (back/forward navigation)
- Music playlists (next/previous track)
- Undo/redo in text editors
- Memory allocators (free block lists)
- Hash table chaining for collision resolution

---

### Deep Understanding

**Internal working / mechanics:**

```
Array in memory:
[100] [200] [300] [400] [500]
  ↑     ↑     ↑     ↑     ↑
 [0]   [1]   [2]   [3]   [4]   ← contiguous addresses

Linked List in memory:
Node A          Node B          Node C
[100 | addr_B]  [200 | addr_C]  [300 | NULL]
  at 0x001        at 0x047        at 0x102
```

The nodes live at completely random addresses in memory (0x001, 0x047, 0x102 — not sequential). What links them is the pointer stored inside each node.

**Mental Model:**
Think of a treasure hunt. Each clue (node) tells you the value AND the location of the next clue. You cannot jump to clue #5 directly — you must start at clue #1 and follow every pointer until you reach #5.

**Core terminology:**
- **Node**: The basic building block. Contains data + next pointer.
- **Head**: A pointer/reference to the **first node**. This is your entry point. If head is NULL/None, the list is empty.
- **Tail**: The last node. Its next pointer is NULL/None.
- **Traversal**: Starting from head, visiting each node by following next pointers until NULL.
- **NULL/None**: Signals end of list.

**Common misconceptions:**
1. ❌ "Head IS the list" → Head is just a **pointer** to the first node. The list exists in memory; head just tells you where it starts.
2. ❌ "Linked lists use less memory than arrays" → FALSE. Each node stores an extra pointer (8 bytes on 64-bit systems), so memory overhead is HIGHER per element.
3. ❌ "Insertion is always O(1)" → Only if you already have the pointer to the insertion position. Finding that position is O(n).
4. ❌ "Linked lists and arrays have the same cache behavior" → Arrays are cache-friendly (sequential memory). Linked lists cause cache misses (scattered memory) — this matters at scale.

---

### Node Structure

```python
# Python Implementation
class Node:
    def __init__(self, data):
        self.data = data      # The value stored in this node
        self.next = None      # Pointer to the next node (default: None)
```

```java
// Java Implementation
class Node {
    int data;
    Node next;
    
    Node(int data) {
        this.data = data;
        this.next = null;  // Java defaults to null, but explicit is better
    }
}
```

```cpp
// C++ Implementation
struct Node {
    int data;
    Node* next;  // Pointer to the next Node in memory
    
    Node(int data) : data(data), next(nullptr) {}
};
```

**Line-by-line explanation (Python):**
- `class Node:` — defines the blueprint for every element in the list
- `self.data = data` — stores the actual value (int, string, anything)
- `self.next = None` — by default, this node points to nothing (it's alone)

When you create `Node(5)`, you get a standalone node holding value 5, pointing to nothing. It becomes part of a list only when you connect it to other nodes via `next`.

---

### 1.2 — Memory Layout: Dynamic Memory vs Arrays

```
Array (Static/Dynamic):
┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │
└────┴────┴────┴────┴────┘
  ↑ All elements at consecutive addresses
  ↑ Size may need to be declared (C) or dynamically doubles (Python list, Java ArrayList)
  ↑ Random access: arr[3] = O(1) (just add 3 * element_size to base address)

Linked List (Always dynamic):
┌──────────┐      ┌──────────┐      ┌──────────┐
│ 10 │ ───────→  │ 20 │ ───────→  │ 30 │ NULL │
└──────────┘      └──────────┘      └──────────┘
  at 0xFF10         at 0xAB34         at 0x1122
  ↑ Elements scattered in memory
  ↑ Access element 3: must traverse from head — O(n)
```

**Comparison Table:**

```
Operation          | Array  | Linked List
-------------------|--------|------------
Access by index    | O(1)   | O(n)
Search             | O(n)   | O(n)
Insert at start    | O(n)   | O(1)
Insert at end      | O(1)*  | O(n)** / O(1)***
Insert at middle   | O(n)   | O(1)****
Delete at start    | O(n)   | O(1)
Delete at end      | O(1)*  | O(n)
Delete at middle   | O(n)   | O(1)****
Memory overhead    | Low    | High (extra pointer per node)
Cache performance  | Great  | Poor (scattered memory)

* Amortized for dynamic arrays
** Without tail pointer
*** With tail pointer maintained
**** Given pointer to node, not counting search time
```

---

### 1.3 — Singly Linked List: Complete Implementation

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


class SinglyLinkedList:
    def __init__(self):
        self.head = None  # Empty list: head points to nothing


    # ─────────────────────────────────────────
    # INSERT AT BEGINNING — O(1)
    # ─────────────────────────────────────────
    def insert_at_beginning(self, data):
        """
        New node's next = current head
        Then update head to point to new node
        """
        new_node = Node(data)
        new_node.next = self.head   # Step 1: new node → old first node
        self.head = new_node        # Step 2: head → new node
        
        # Why this order? If you do head = new_node FIRST,
        # you lose the reference to the old list!


    # ─────────────────────────────────────────
    # INSERT AT END — O(n)
    # ─────────────────────────────────────────
    def insert_at_end(self, data):
        """
        Traverse to the last node.
        Set last node's next = new node.
        """
        new_node = Node(data)
        
        # Edge case: empty list
        if self.head is None:
            self.head = new_node
            return
        
        # Traverse to the last node
        current = self.head
        while current.next is not None:   # Stop when current IS the last node
            current = current.next
        
        current.next = new_node   # Attach new node after last node


    # ─────────────────────────────────────────
    # PRINT LIST — O(n)
    # ─────────────────────────────────────────
    def print_list(self):
        """
        Traverse from head to tail, printing each value.
        Common mistake: using head directly (modifies head!)
        Always use a separate 'current' pointer.
        """
        if self.head is None:
            print("List is empty")
            return
        
        current = self.head
        elements = []
        
        while current is not None:
            elements.append(str(current.data))
            current = current.next
        
        print(" → ".join(elements) + " → NULL")


    # ─────────────────────────────────────────
    # COUNT NODES — O(n)
    # ─────────────────────────────────────────
    def count_nodes(self):
        count = 0
        current = self.head
        while current is not None:
            count += 1
            current = current.next
        return count


# ─────────────────────────────────────────
# USAGE DEMONSTRATION
# ─────────────────────────────────────────
ll = SinglyLinkedList()
ll.insert_at_end(10)
ll.insert_at_end(20)
ll.insert_at_end(30)
ll.insert_at_beginning(5)
ll.print_list()        # 5 → 10 → 20 → 30 → NULL
print(ll.count_nodes())  # 4
```

---

### 1.4 — Head Pointer: The Most Critical Concept

```
WHAT HAPPENS WHEN YOU LOSE THE HEAD:

Initial list:  head → [A] → [B] → [C] → NULL

# WRONG: Traversing with head itself
current = head
while current != None:
    current = current.next    # head keeps moving...

# After loop: head = NULL ← YOU LOST YOUR ENTIRE LIST!

# CORRECT: Use a separate traversal pointer
current = head               # current is a copy of the reference
while current != None:
    current = current.next   # only current moves, head stays at A
```

**Professional Insight:** This is the #1 rookie mistake. In interviews, always think: "Am I modifying head directly or using a copy?" Create a `current = self.head` variable and traverse with that.

---

### 1.5 — Why Insertion/Deletion is O(1)

This requires deep understanding because the O(1) claim is CONDITIONAL.

```
Inserting C between A and B:

Before: [A] → [B] → [C] → NULL
         ↑
         We have a pointer to A

Steps:
1. new_node.next = A.next   →  [new_node] → [B]
2. A.next = new_node        →  [A] → [new_node] → [B]

That's it. 2 pointer operations. Regardless of list size = O(1)

Compare to array:
Insert 'X' at position 2 in [1, 2, 3, 4, 5]:
→ Shift 3, 4, 5 one position right
→ Place X at position 2
→ n elements to shift = O(n)
```

**The catch:** To get the pointer to position A in the first place, you need O(n) traversal. So "insertion is O(1)" means: **once you have the pointer, the actual insertion is O(1).**

---

### 1.6 — Traversal Deep Dive

```python
def traverse(head):
    """
    The fundamental operation. Master this and everything else becomes easy.
    """
    current = head          # Start at first node
    
    while current is not None:    # Continue until we fall off the end
        print(current.data)       # Process current node
        current = current.next    # Move to next node
    
    # After loop: current = None (we fell off the end)
    # head still points to first node ✓
```

```
Traversal dry run on: head → [1] → [2] → [3] → NULL

Step 1: current = head = Node(1)
        current.data = 1 → print 1
        current = current.next = Node(2)

Step 2: current = Node(2)
        current.data = 2 → print 2
        current = current.next = Node(3)

Step 3: current = Node(3)
        current.data = 3 → print 3
        current = current.next = None

Step 4: current = None → while condition False → exit loop

Output: 1 2 3 ✓
```

---

### Mini Project — Session 1: Menu-Driven Linked List Program

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


class LinkedList:
    def __init__(self):
        self.head = None

    def insert(self, data):
        new_node = Node(data)
        if not self.head:
            self.head = new_node
            return
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node

    def delete(self, data):
        if not self.head:
            print("List is empty")
            return
        # Delete head case
        if self.head.data == data:
            self.head = self.head.next
            print(f"Deleted {data}")
            return
        # Find node before the one to delete
        current = self.head
        while current.next and current.next.data != data:
            current = current.next
        if not current.next:
            print(f"{data} not found")
        else:
            current.next = current.next.next
            print(f"Deleted {data}")

    def search(self, data):
        current = self.head
        position = 0
        while current:
            if current.data == data:
                print(f"Found {data} at position {position}")
                return True
            current = current.next
            position += 1
        print(f"{data} not found")
        return False

    def print_list(self):
        if not self.head:
            print("Empty list")
            return
        current = self.head
        parts = []
        while current:
            parts.append(str(current.data))
            current = current.next
        print(" → ".join(parts) + " → NULL")


def menu():
    ll = LinkedList()
    while True:
        print("\n=== LINKED LIST MENU ===")
        print("1. Insert")
        print("2. Delete")
        print("3. Search")
        print("4. Print")
        print("5. Exit")
        choice = input("Choose: ")

        if choice == "1":
            val = int(input("Value to insert: "))
            ll.insert(val)
        elif choice == "2":
            val = int(input("Value to delete: "))
            ll.delete(val)
        elif choice == "3":
            val = int(input("Value to search: "))
            ll.search(val)
        elif choice == "4":
            ll.print_list()
        elif choice == "5":
            break
        else:
            print("Invalid choice")


menu()
```

---

### ✅ SESSION 1 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Node = data + next pointer
✓ Head = entry point to the list (NEVER lose it!)
✓ Traversal = current = head, while current != None, current = current.next
✓ Insert at beginning: O(1) — redirect two pointers
✓ Insert at end: O(n) without tail pointer
✓ Deletion: O(1) once you have the predecessor node
✓ Arrays give O(1) access; Linked Lists give O(1) insert/delete

CRITICAL RULES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Rule 1: NEVER traverse with head. Use current = head.
Rule 2: Always handle empty list (head is None) as edge case.
Rule 3: When inserting, set new_node.next BEFORE updating head.
Rule 4: Last node's next MUST be None.
Rule 5: Time complexity is O(n) for search/access — no shortcuts.
```

---
---

# ╔══════════════════════════════════════════════╗
# ║  SESSION 2 — INSERTIONS, DELETIONS, SEARCH  ║
# ╚══════════════════════════════════════════════╝

---

## 2.1 — Insert at a Specific Position

### Concept Explanation

**What it is:** Insert a new node at index `k` (0-based) in the list.

**Why tricky:** You need to:
1. Traverse to node at index `k-1` (the predecessor)
2. Connect new node to node at `k`
3. Connect predecessor to new node

Order matters: set `new_node.next` FIRST, then update predecessor's next. Reverse order loses the rest of the list.

```python
def insert_at_position(self, data, position):
    """
    Position 0 = beginning
    Position n = end (appending)
    """
    new_node = Node(data)
    
    # Case 1: Insert at beginning
    if position == 0:
        new_node.next = self.head
        self.head = new_node
        return
    
    # Case 2: Insert in middle/end
    current = self.head
    current_pos = 0
    
    # Traverse to the node BEFORE the target position
    while current is not None and current_pos < position - 1:
        current = current.next
        current_pos += 1
    
    # If position is beyond list length
    if current is None:
        print(f"Position {position} out of bounds")
        return
    
    # Insert: new_node.next must be set BEFORE current.next changes
    new_node.next = current.next   # Step 1: new_node → next node
    current.next = new_node        # Step 2: predecessor → new_node
```

```
Dry run: Insert 99 at position 2 in [10 → 20 → 30 → 40]

Initial:  head → [10] → [20] → [30] → [40] → NULL

Traverse to position 1 (node with value 20):
  current = Node(10), pos = 0 → move
  current = Node(20), pos = 1 → STOP (we're at position - 1 = 1)

new_node = Node(99)
Step 1: new_node.next = current.next = Node(30)
        [99] → [30] → [40] → NULL

Step 2: current.next = new_node
        [20] → [99] → [30] → [40] → NULL

Result: head → [10] → [20] → [99] → [30] → [40] → NULL ✓
```

---

## 2.2 — Deletion Operations

### Delete by Value

```python
def delete_by_value(self, value):
    """
    Find the node with given value and remove it.
    The key insight: we need the PREDECESSOR node.
    """
    # Edge case: empty list
    if self.head is None:
        print("List is empty")
        return
    
    # Edge case: delete head
    if self.head.data == value:
        self.head = self.head.next
        # In Python, garbage collector handles memory
        # In C/C++: free(temp) where temp held old head
        return
    
    # General case: find predecessor
    prev = None
    current = self.head
    
    while current is not None:
        if current.data == value:
            prev.next = current.next   # Bridge: skip current node
            return
        prev = current
        current = current.next
    
    print(f"Value {value} not found")
```

```
Dry run: Delete 30 from [10 → 20 → 30 → 40 → NULL]

prev = None, current = Node(10)
  10 != 30 → prev = Node(10), current = Node(20)
  20 != 30 → prev = Node(20), current = Node(30)
  30 == 30 → prev.next = current.next = Node(40)

Result: [10 → 20 → 40 → NULL] ✓
Node(30) is now unreachable — garbage collected (Python) or freed (C++)
```

---

### Delete Head

```python
def delete_head(self):
    if self.head is None:
        print("List is empty")
        return
    
    removed = self.head.data
    self.head = self.head.next    # Head now points to second node
    # Old head is garbage collected
    print(f"Deleted head: {removed}")
```

### Delete Tail

```python
def delete_tail(self):
    if self.head is None:
        print("List is empty")
        return
    
    # Only one node
    if self.head.next is None:
        removed = self.head.data
        self.head = None
        print(f"Deleted: {removed}")
        return
    
    # Traverse to second-to-last node
    current = self.head
    while current.next.next is not None:   # Stop when current.next IS tail
        current = current.next
    
    removed = current.next.data
    current.next = None    # Disconnect the tail
    print(f"Deleted tail: {removed}")
```

### Delete Nth Node (1-indexed)

```python
def delete_nth(self, n):
    """Delete the nth node (1-indexed)"""
    if self.head is None:
        print("List is empty")
        return
    
    # Delete first node
    if n == 1:
        self.head = self.head.next
        return
    
    current = self.head
    count = 1
    
    # Go to the (n-1)th node
    while current is not None and count < n - 1:
        current = current.next
        count += 1
    
    # n is larger than list length
    if current is None or current.next is None:
        print(f"Position {n} doesn't exist")
        return
    
    current.next = current.next.next   # Skip the nth node
```

---

## 2.3 — Search Operations

### Iterative Search

```python
def search_iterative(self, target):
    """
    Returns: position (0-indexed) if found, -1 if not found
    Time: O(n), Space: O(1)
    """
    current = self.head
    position = 0
    
    while current is not None:
        if current.data == target:
            return position
        current = current.next
        position += 1
    
    return -1  # Not found
```

### Recursive Search

```python
def search_recursive(self, node, target, position=0):
    """
    Time: O(n), Space: O(n) — stack frames
    
    Base cases:
    1. Node is None → not found
    2. Node.data == target → found!
    
    Recursive case: search in rest of list
    """
    # Base case 1: end of list
    if node is None:
        return -1
    
    # Base case 2: found
    if node.data == target:
        return position
    
    # Recursive case: search next node
    return self.search_recursive(node.next, target, position + 1)

# Call with: result = ll.search_recursive(ll.head, 30)
```

---

## 2.4 — Edge Cases: The Interview Killers

```python
"""
Every linked list operation must handle these:

1. EMPTY LIST (head is None)
2. SINGLE NODE LIST (head.next is None)
3. OPERATION ON HEAD (special case — no predecessor)
4. OPERATION ON TAIL (last node, next = None)
5. POSITION OUT OF BOUNDS
6. VALUE NOT FOUND

Template for bulletproof code:
"""

def robust_operation(self, value):
    # Check 1: Empty list
    if self.head is None:
        return "empty list"
    
    # Check 2: Single node
    if self.head.next is None:
        # Handle single node case separately
        pass
    
    # Check 3: Operation involves head
    if self.head.data == value:
        # Handle head case separately
        pass
    
    # General case with traversal...
```

---

## 2.5 — Memory Leaks (C/C++ Critical)

```cpp
// C++ WRONG — memory leak!
void delete_node(Node* &head, int value) {
    if (head == nullptr) return;
    if (head->data == value) {
        head = head->next;    // Old head memory is LEAKED!
        return;
    }
    // ...
}

// C++ CORRECT — properly free memory
void delete_node(Node* &head, int value) {
    if (head == nullptr) return;
    if (head->data == value) {
        Node* temp = head;    // Save reference
        head = head->next;    // Update head
        delete temp;          // Free old head memory
        return;
    }
    
    Node* prev = nullptr;
    Node* current = head;
    while (current != nullptr && current->data != value) {
        prev = current;
        current = current->next;
    }
    if (current == nullptr) return;  // Not found
    
    prev->next = current->next;
    delete current;   // FREE THE NODE!
}
```

In Python and Java, garbage collection handles this automatically. In C/C++, every `new` must have a corresponding `delete`/`free`.

---

## 2.6 — Mini Project: Browser History Simulation

```python
class BrowserHistory:
    """
    Simulates browser back/forward using a linked list.
    Each page = one node.
    Current page = current pointer.
    """
    
    def __init__(self):
        self.head = None
        self.current = None
    
    def visit(self, url):
        """Navigate to a new URL. Clears forward history."""
        new_page = Node(url)
        
        if self.head is None:
            # First page visited
            self.head = new_page
            self.current = new_page
        else:
            # Visiting new page cuts off forward history
            # (In a real browser, forward is gone when you visit new page)
            self.current.next = new_page
            self.current = new_page
        
        print(f"Visiting: {url}")
    
    def back(self, steps):
        """Go back 'steps' pages. Stays at earliest if fewer pages."""
        # Need to traverse from head since it's singly linked
        # Professional note: for real browsers, use doubly linked list
        # For now, rebuild path to current
        
        # Collect all pages up to current
        pages = []
        node = self.head
        while node is not None and node != self.current.next:
            pages.append(node)
            node = node.next
        
        if not pages:
            return self.current.data
        
        # Go back steps positions
        current_index = len(pages) - 1
        target_index = max(0, current_index - steps)
        self.current = pages[target_index]
        print(f"Back {steps}: Now at {self.current.data}")
        return self.current.data
    
    def print_history(self):
        """Print all visited pages."""
        node = self.head
        history = []
        while node is not None:
            marker = " ← [CURRENT]" if node == self.current else ""
            history.append(node.data + marker)
            node = node.next
        print("\nBrowser History:")
        for i, page in enumerate(history):
            print(f"  [{i}] {page}")


# Demo
browser = BrowserHistory()
browser.visit("google.com")
browser.visit("github.com")
browser.visit("stackoverflow.com")
browser.visit("docs.python.org")
browser.print_history()
browser.back(2)
browser.print_history()
```

---

## 2.7 — Common Mistakes Deep Dive

**Mistake 1: Off-by-one in position traversal**
```python
# WRONG: traverses one too many
while current is not None and count < n:
    ...

# CORRECT: stop at predecessor (n-1)
while current is not None and count < n - 1:
    ...
```

**Mistake 2: Forgetting prev pointer**
```python
# WRONG: can't delete without knowing predecessor
current = head
while current.data != target:
    current = current.next
# Now what? You can't update the node before current!

# CORRECT: track prev
prev = None
current = head
while current.data != target:
    prev = current
    current = current.next
prev.next = current.next  # Now you can delete
```

**Mistake 3: Checking current.next instead of current**
```python
# WRONG: crashes when list is empty or current IS last node
while current.next is not None:
    if current.next.data == target:  # Fine but limits behavior

# CORRECT for general traversal:
while current is not None:
    if current.data == target:
        ...
```

---

### ✅ SESSION 2 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Always handle: empty list, single node, head/tail operations
✓ Deletion requires the PREDECESSOR node (prev pointer)
✓ When inserting: set new_node.next FIRST, then update prev.next
✓ Recursive search is elegant but uses O(n) stack space
✓ In C/C++: ALWAYS free deleted nodes to prevent memory leaks
✓ Off-by-one errors are the most common interview mistake

MENTAL MODEL:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
To delete node X:
  1. Find the node BEFORE X (prev)
  2. prev.next = X.next (bridge over X)
  3. Free X (C++) or let GC handle it (Python/Java)

To insert at position k:
  1. Traverse to position k-1
  2. new_node.next = current.next
  3. current.next = new_node
```

---
---

# ╔══════════════════════════════════════════════════╗
# ║  SESSION 3 — FAST & SLOW POINTER PATTERN         ║
# ╚══════════════════════════════════════════════════╝

---

## 3.1 — The Two-Pointer Pattern: Foundation

### Concept Explanation

**What it is:**
The Fast & Slow pointer technique (also called the "Tortoise and Hare" algorithm) uses two pointers that move through the list at different speeds:
- **Slow pointer**: moves 1 node at a time
- **Fast pointer**: moves 2 nodes at a time

**Why it exists:**
Many linked list problems require knowing the relative position of nodes (middle, start of cycle, etc.) without knowing the list length upfront. Using two pointers eliminates the need for a separate count pass.

**When to use it:**
- Finding the middle of a linked list
- Detecting a cycle
- Finding where a cycle starts
- Finding the nth node from the end (variation)

**Real-world relevance:**
- Memory leak detection in circular data structures
- Round-robin scheduling (circular lists)
- Interview standard: appears in 30%+ of linked list problems

---

### Deep Understanding

**Mental model — The Race Track:**

```
Imagine a circular running track. Two runners start at the same point.
One runs at 1 mph (slow), one at 2 mph (fast).

If track is circular → fast runner eventually LAPS slow runner (they meet)
If track is straight → fast runner reaches end, no meeting

Applied to linked lists:
- No cycle → fast pointer reaches NULL
- Cycle exists → fast pointer enters cycle and eventually catches slow pointer
```

**Why they MUST meet (if cycle exists):**

```
Consider: slow moves 1 step, fast moves 2 steps.
At any point, their distance difference changes by 1 per step.
If fast is k steps behind slow inside the cycle of length L:
  After 1 step: gap = k - 1 (fast closes in by 1 each iteration)
  After k steps: gap = 0 → THEY MEET
  
The meeting is guaranteed because the gap decreases by 1 each time,
and since we're in a cycle, they will eventually be at distance 0.
```

---

## 3.2 — Floyd's Cycle Detection Algorithm

### Problem 1: Detect Cycle (LeetCode #141)

```python
def has_cycle(head):
    """
    Returns True if linked list has a cycle, False otherwise.
    
    Time: O(n)  — fast pointer traverses at most 2 full cycles
    Space: O(1) — only two pointers
    
    Floyd's Algorithm: slow moves 1, fast moves 2.
    If cycle exists, they will meet.
    If no cycle, fast reaches None.
    """
    if head is None or head.next is None:
        return False
    
    slow = head
    fast = head
    
    while fast is not None and fast.next is not None:
        slow = slow.next          # Move 1 step
        fast = fast.next.next     # Move 2 steps
        
        if slow == fast:          # They met → cycle detected!
            return True
    
    return False  # Fast reached end → no cycle
```

```
Dry run on: 1 → 2 → 3 → 4 → 5 → (points back to 3)

Positions:  1    2    3    4    5
                      ↑___________↩ (cycle: 5.next = 3)

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=5
Step 3: slow=4, fast=4  ← MEETING POINT! (fast moved from 5 → 3 → 5? No...)

Let me redo with proper tracking:
  Step 1: slow = 2, fast = 3
  Step 2: slow = 3, fast = 5
  Step 3: slow = 4, fast = 4   (fast: 5 → 3 → 5... wait)
  
  Actually fast.next.next: from 5, next=3, next.next=4. Yes, fast=4.
  slow=4, fast=4 → CYCLE DETECTED ✓
```

**Why check `fast.next is not None`?**

```python
# If fast is at second-to-last node:
fast.next = last_node  (not None)
fast.next.next = None  (last node's next)

# If we do fast = fast.next.next WITHOUT checking fast.next first:
# fast.next could be None, and None.next causes NullPointerException!

# So: check BOTH fast is not None AND fast.next is not None
while fast is not None and fast.next is not None:
```

---

### Problem 2: Find Cycle Start (LeetCode #142)

This is one of the most elegant mathematical results in algorithms.

```
Mathematical Proof:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Let:
  F = distance from head to cycle start
  C = cycle length
  a = distance from cycle start to meeting point (within cycle)

When they meet:
  slow has traveled: F + a
  fast has traveled: F + a + C  (fast did one extra loop)
  
Since fast travels 2× as far as slow:
  2(F + a) = F + a + C
  2F + 2a  = F + a + C
  F + a    = C
  F        = C - a

This means:
  F = distance from head to cycle start
  C - a = distance from meeting point to cycle start (going forward)

Therefore: If one pointer starts at HEAD and another at the MEETING POINT,
both moving 1 step at a time, they meet exactly at the CYCLE START!
```

```python
def detect_cycle_start(head):
    """
    Returns the node where the cycle begins.
    Returns None if no cycle.
    
    Time: O(n), Space: O(1)
    """
    if head is None or head.next is None:
        return None
    
    slow = head
    fast = head
    meeting_point = None
    
    # Phase 1: Detect if cycle exists and find meeting point
    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            meeting_point = slow
            break
    
    # No cycle
    if meeting_point is None:
        return None
    
    # Phase 2: Find cycle start
    # One pointer at head, one at meeting point
    # Both move 1 step at a time
    pointer1 = head
    pointer2 = meeting_point
    
    while pointer1 != pointer2:
        pointer1 = pointer1.next
        pointer2 = pointer2.next
    
    # They meet at cycle start!
    return pointer1
```

---

### Problem 3: Middle of Linked List (LeetCode #876)

```python
def find_middle(head):
    """
    Returns the middle node.
    If even length: returns the SECOND middle node.
    
    Example: 1→2→3→4→5 → returns Node(3)
    Example: 1→2→3→4   → returns Node(3) (second middle)
    
    Time: O(n), Space: O(1)
    """
    slow = head
    fast = head
    
    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
    
    # When fast reaches end, slow is at middle
    return slow
```

```
Dry run on: 1 → 2 → 3 → 4 → 5 → NULL

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=5
Step 3: fast.next = None → STOP
Return slow = Node(3) ✓

Dry run on: 1 → 2 → 3 → 4 → NULL

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=NULL (fast.next.next = 4.next = NULL... wait)

  fast=3, fast.next=4, fast.next.next=None
  So: fast = None → loop condition fails → STOP
  Return slow = Node(3) ✓ (second middle of 4-node list)
```

**Why does this work?**

```
When fast moves 2× speed of slow:
  - By the time fast reaches end (node n), slow is at node n/2
  - That's the middle!

This is elegant because you don't need to:
  1. Count total nodes
  2. Divide by 2
  3. Traverse again
All in ONE pass with O(1) space.
```

---

### Problem 4: Happy Number (LeetCode #202)

```python
def is_happy(n):
    """
    A happy number: repeatedly replace n with sum of squares of digits.
    If sequence reaches 1 → happy.
    If sequence loops → not happy.
    
    Connection to linked lists: unhappy numbers form a CYCLE.
    We can use Floyd's cycle detection!
    
    Time: O(log n), Space: O(1)
    """
    def digit_square_sum(num):
        total = 0
        while num > 0:
            digit = num % 10
            total += digit * digit
            num //= 10
        return total
    
    slow = n
    fast = n
    
    while True:
        slow = digit_square_sum(slow)              # 1 step
        fast = digit_square_sum(digit_square_sum(fast))  # 2 steps
        
        if slow == fast:
            break
    
    # If they meet at 1, it's happy. Otherwise it's in a cycle != 1.
    return slow == 1


# Test
print(is_happy(19))   # True: 1² + 9² = 82, 6²+4=100, 1² + 0² + 0² = 1
print(is_happy(2))    # False: eventually cycles
```

**Connection to linked lists:**
The sequence of numbers forms an implicit linked list. Each number "points to" its next (digit square sum). Happy numbers reach the "tail" node 1. Unhappy numbers form a cycle — detectable with Floyd's algorithm!

---

## 3.3 — Implementation Pattern Reference

```python
# TEMPLATE: Fast & Slow Pointer
slow = head
fast = head

while fast is not None and fast.next is not None:
    slow = slow.next
    fast = fast.next.next
    # Add meeting condition if needed

# After loop:
# - If no cycle: fast is None or fast.next is None
# - slow is at middle
# - If cycle: slow == fast at meeting point
```

---

## 3.4 — Common Mistakes

**Mistake 1: Checking only `fast is not None`**
```python
# WRONG — crashes on even-length lists
while fast is not None:
    slow = slow.next
    fast = fast.next.next  # fast.next could be None → CRASH!

# CORRECT:
while fast is not None and fast.next is not None:
```

**Mistake 2: Starting fast one step ahead**
```python
# Some problems need fast to start at head.next
# Others need both at head
# Always understand what the loop condition implies
slow = head
fast = head.next   # This gives FIRST middle for even lists
```

**Mistake 3: Infinite loop if cycle detection check is wrong**
```python
# WRONG: checking data instead of reference
if slow.data == fast.data:  # Two different nodes can have same data!

# CORRECT: check if they're the SAME node (reference equality)
if slow == fast:   # Python: same object
if slow is fast:   # Python: even more explicit
```

---

### ✅ SESSION 3 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Slow moves 1 step, Fast moves 2 steps
✓ Cycle detected when slow == fast (reference equality)
✓ Cycle start: restart one pointer at head, both move at 1x speed
✓ Middle: when fast hits end, slow is at middle
✓ ALWAYS check: fast != None AND fast.next != None

FLOYD'S ALGORITHM STEPS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Phase 1: Find meeting point (slow×1, fast×2)
Phase 2: Move pointer1 to head, pointer2 stays at meeting
         Both move ×1 until they meet → that's the cycle start!

CORE FORMULA:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
F = C - a  (where F = head to cycle start, C = cycle length, a = head to meeting)
This PROVES that head pointer and meeting point pointer walk same distance to cycle start.
```

---
---

# ╔══════════════════════════════════════════╗
# ║  SESSION 4 — REVERSAL PATTERNS           ║
# ╚══════════════════════════════════════════╝

---

## 4.1 — Why Reversal is So Important

Reversal is the #1 most tested linked list operation in interviews. It appears in:
- Palindrome check
- Reverse K groups
- Reorder list
- Adding two numbers
- Many merge/sort problems

Master it in all 3 forms: iterative, recursive, partial.

---

## 4.2 — Iterative Reversal (Most Important)

### Concept

**What happens:**
We redirect every `next` pointer to point BACKWARDS instead of forwards. After reversal, what was the tail becomes the new head.

**Core insight:** We need THREE pointers: `prev`, `current`, `next_node`. Why? Because when you change `current.next = prev`, you lose the reference to `current.next`. So you must save it first.

```python
def reverse_iterative(head):
    """
    Reverse the linked list in-place.
    
    Time: O(n), Space: O(1)
    
    Three pointer technique:
    prev     = what current should point to (starts None)
    current  = node being processed
    next_node = saved reference to current.next (before we destroy it)
    """
    prev = None
    current = head
    
    while current is not None:
        next_node = current.next    # Step 1: SAVE next (before losing it)
        current.next = prev         # Step 2: REVERSE the pointer
        prev = current              # Step 3: Move prev forward
        current = next_node         # Step 4: Move current forward
    
    # After loop: current = None, prev = last node (new head)
    return prev   # New head of reversed list
```

```
Dry run on: head → [1] → [2] → [3] → [4] → NULL

Initial: prev=NULL, current=1

Iteration 1:
  next_node = 2
  1.next = NULL      (was pointing to 2, now points back to nothing)
  prev = 1
  current = 2
  State: NULL ← [1]    [2] → [3] → [4] → NULL
                prev   cur

Iteration 2:
  next_node = 3
  2.next = 1         (now points back to 1)
  prev = 2
  current = 3
  State: NULL ← [1] ← [2]    [3] → [4] → NULL
                       prev   cur

Iteration 3:
  next_node = 4
  3.next = 2         (now points back to 2)
  prev = 3
  current = 4
  State: NULL ← [1] ← [2] ← [3]    [4] → NULL
                              prev   cur

Iteration 4:
  next_node = NULL
  4.next = 3         (now points back to 3)
  prev = 4
  current = NULL
  State: NULL ← [1] ← [2] ← [3] ← [4]
                                    prev   (new head!)

Loop ends. Return prev = Node(4)
Result: 4 → 3 → 2 → 1 → NULL ✓
```

**Memory Model:**
```
Before: head=1, 1→2→3→4→NULL
After:  head=4, 4→3→2→1→NULL
All connections are reversed, no new nodes created.
```

---

## 4.3 — Recursive Reversal

```python
def reverse_recursive(head):
    """
    Time: O(n), Space: O(n) — call stack depth = n
    
    The magic: recursively reverse the REST of the list,
    then fix the connection at the current level.
    """
    # Base case: empty list or single node
    if head is None or head.next is None:
        return head   # This is the new head (last node)
    
    # Recursive call: reverse everything after head
    # This returns the new head (last node of original list)
    new_head = reverse_recursive(head.next)
    
    # At this point: head.next still points to the node AFTER head
    # But that node (head.next) now needs to point BACK to head
    head.next.next = head    # Make next node point back to current
    head.next = None         # Disconnect current from forward direction
    
    return new_head   # Bubble up the new head
```

```
Call stack visualization for [1 → 2 → 3 → NULL]:

reverse(1):
  reverse(2):
    reverse(3):
      Base case: return 3 (new head)
    
    # Back in reverse(2):
    # head = 2, head.next = 3
    # 3.next = 2  →  [3] → [2]
    # 2.next = None
    # Return new_head = 3
  
  # Back in reverse(1):
  # head = 1, head.next = 2
  # 2.next = 1  →  [3] → [2] → [1]
  # 1.next = None
  # Return new_head = 3

Final: 3 → 2 → 1 → NULL ✓
```

**Interview Tip:** The recursive approach is elegant but uses O(n) stack space. For large lists, iterative is preferred. Know both — interviewers often ask for both approaches.

---

## 4.4 — Reverse Sublist (LeetCode #92)

Reverse nodes from position `left` to position `right` (1-indexed).

```python
def reverse_between(head, left, right):
    """
    Reverse nodes from position left to right (1-indexed).
    
    Example: 1→2→3→4→5, left=2, right=4
    Result:  1→4→3→2→5
    
    Time: O(n), Space: O(1)
    
    Key insight: Use a dummy node to handle the edge case
    where left=1 (reversing from the head).
    """
    if not head or left == right:
        return head
    
    # Dummy node simplifies head-reversal case
    dummy = Node(0)
    dummy.next = head
    prev = dummy
    
    # Step 1: Move prev to the node BEFORE position 'left'
    for _ in range(left - 1):
        prev = prev.next
    
    # prev is now at position left-1
    # current is at position left (start of reversal zone)
    current = prev.next
    
    # Step 2: Reverse (right - left) times
    # Uses the "insert at front" technique
    for _ in range(right - left):
        next_node = current.next       # Save next
        current.next = next_node.next  # Disconnect next_node
        next_node.next = prev.next     # next_node → start of sublist
        prev.next = next_node          # prev → next_node (insert at front)
    
    return dummy.next
```

```
Dry run: 1→2→3→4→5, left=2, right=4

dummy→1→2→3→4→5
prev points to: 1 (after moving left-1=1 times)
current = 2

Iteration 1 (i=0):
  next_node = 3
  current(2).next = 4 (skipping 3)
  3.next = prev.next = 2 (3 → 2)
  prev.next = 3 (1 → 3)
  State: dummy→1→3→2→4→5

Iteration 2 (i=1):
  next_node = 4
  current(2).next = 5
  4.next = prev.next = 3 (4 → 3)
  prev.next = 4 (1 → 4)
  State: dummy→1→4→3→2→5

Return dummy.next = 1 → 4 → 3 → 2 → 5 ✓
```

**Why "insert at front" works:**
Each iteration takes the node immediately after `current` and inserts it right after `prev`. This effectively reverses the sublist one node at a time.

---

## 4.5 — Reverse Nodes in K Groups (LeetCode #25) — HARD

```python
def reverse_k_group(head, k):
    """
    Reverse every k nodes. If remaining nodes < k, leave them as-is.
    
    Example: 1→2→3→4→5, k=2 → 2→1→4→3→5
    Example: 1→2→3→4→5, k=3 → 3→2→1→4→5
    
    Time: O(n), Space: O(n/k) for recursion
    """
    # Check if we have at least k nodes remaining
    count = 0
    node = head
    while node and count < k:
        node = node.next
        count += 1
    
    # Not enough nodes — return as-is
    if count < k:
        return head
    
    # Reverse k nodes starting from head
    prev = None
    current = head
    for _ in range(k):
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    
    # After reversing k nodes:
    # prev = new head of this group
    # current = start of next group
    # head = tail of this group (now points to None)
    
    # Recursively reverse the next group and connect
    head.next = reverse_k_group(current, k)
    
    return prev   # New head
```

```
Visualization: 1→2→3→4→5→6, k=3

Group 1: [1,2,3], Group 2: [4,5,6]

reverse_k_group(1, 3):
  Reverse [1,2,3] → [3,2,1], current points to 4
  1.next = reverse_k_group(4, 3)
    Reverse [4,5,6] → [6,5,4], current points to None
    4.next = reverse_k_group(None, 3) = None
    Return 6
  1.next = 6
  Return 3

Final: 3→2→1→6→5→4 ✓
```

---

## 4.6 — Palindrome Linked List (LeetCode #234)

This problem combines MULTIPLE techniques: find middle + reverse + compare.

```python
def is_palindrome(head):
    """
    Check if linked list is a palindrome.
    
    Strategy:
    1. Find middle using slow/fast pointers
    2. Reverse second half
    3. Compare first half and reversed second half
    4. (Optionally restore the list)
    
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return True
    
    # Step 1: Find middle
    slow = head
    fast = head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    
    # slow is now at the last node of first half
    # (for odd: middle; for even: first of two middles)
    
    # Step 2: Reverse second half
    second_half_head = reverse(slow.next)
    
    # Step 3: Compare
    p1 = head
    p2 = second_half_head
    result = True
    
    while p2:   # Second half is shorter or equal
        if p1.data != p2.data:
            result = False
            break
        p1 = p1.next
        p2 = p2.next
    
    # Step 4: Restore (good practice)
    slow.next = reverse(second_half_head)
    
    return result


def reverse(head):
    prev = None
    current = head
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    return prev


# Test: 1→2→3→2→1 → True
# Test: 1→2→3→3→2→1 → True
# Test: 1→2→3 → False
```

```
Dry run on: 1→2→3→2→1

Find middle:
  slow=1,fast=1 → slow=2,fast=3 → slow=3,fast=1(wrapped? no)
  Wait: fast.next.next:
  step1: slow=2, fast=3
  step2: fast.next=2, fast.next.next=1. So slow=3, fast=1
  fast.next = None → stop. slow=3 (middle)

Reverse second half (2→1):
  Returns: 1→2→NULL

Compare: 1==1 ✓, 2==2 ✓, p2 exhausted

Return True ✓
```

---

### ✅ SESSION 4 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Iterative reversal: prev=None, 4 steps per iteration
✓ ALWAYS save next_node BEFORE changing current.next
✓ After reversal: prev is the new head
✓ Recursive reversal: O(n) space, elegant but risky for large inputs
✓ Partial reversal uses "insert at front" pattern
✓ K-group reversal: check if k nodes exist FIRST, then reverse

THE 4-STEP ITERATIVE REVERSAL MANTRA:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. next_node = current.next   (SAVE)
2. current.next = prev        (REVERSE)
3. prev = current             (ADVANCE prev)
4. current = next_node        (ADVANCE current)

PALINDROME = Find Middle + Reverse Half + Compare
```

---
---

# ╔══════════════════════════════════════════════════╗
# ║  SESSION 5 — TWO POINTER INTERVIEW PATTERNS      ║
# ╚══════════════════════════════════════════════════╝

---

## 5.1 — Remove Nth Node From End (LeetCode #19)

### Concept Explanation

**The insight:** We don't know the length upfront. Using two pointers with a `n`-step gap lets us find the nth from end in ONE pass.

**When fast reaches the end, slow is exactly n positions behind — which is the nth from end.**

```python
def remove_nth_from_end(head, n):
    """
    Remove the nth node from the end in ONE pass.
    
    Technique: Two pointers with a gap of n.
    
    Time: O(L) where L = list length, Space: O(1)
    """
    # Dummy node handles edge case where we delete the head
    dummy = Node(0)
    dummy.next = head
    
    fast = dummy
    slow = dummy
    
    # Move fast n+1 steps ahead (one extra for the dummy)
    for _ in range(n + 1):
        fast = fast.next
    
    # Move both until fast reaches end
    while fast is not None:
        slow = slow.next
        fast = fast.next
    
    # slow is now at the node BEFORE the target
    slow.next = slow.next.next   # Delete the nth from end
    
    return dummy.next
```

```
Dry run: 1→2→3→4→5, n=2 (remove 2nd from end = node 4)

dummy→1→2→3→4→5→NULL

Move fast n+1=3 steps from dummy:
fast: dummy→1→2→3 (fast is at 3)
slow: dummy

Now move both until fast=NULL:
  fast=3→4, slow=dummy→1
  fast=4→5, slow=1→2
  fast=5→NULL? No, fast=5, then fast=NULL
  
  Actually: while fast is not None:
  fast=3 (not None): slow=1, fast=4
  fast=4 (not None): slow=2, fast=5
  fast=5 (not None): slow=3, fast=NULL
  fast=NULL: stop

slow is at node 3.
slow.next = node 4.
slow.next = slow.next.next = node 5.

Result: dummy→1→2→3→5→NULL ✓ (4 is removed)
```

---

## 5.2 — Merge Two Sorted Lists (LeetCode #21)

```python
def merge_two_sorted(l1, l2):
    """
    Merge two sorted linked lists into one sorted list.
    
    Time: O(n + m), Space: O(1) — rearranging existing nodes
    
    Dummy node technique: avoids special case for initializing result head.
    """
    dummy = Node(0)
    current = dummy
    
    while l1 is not None and l2 is not None:
        if l1.data <= l2.data:
            current.next = l1     # Attach l1's current node
            l1 = l1.next          # Advance l1
        else:
            current.next = l2     # Attach l2's current node
            l2 = l2.next          # Advance l2
        current = current.next    # Advance result pointer
    
    # Attach remaining nodes (one list is exhausted)
    if l1 is not None:
        current.next = l1
    else:
        current.next = l2
    
    return dummy.next
```

```
Dry run: l1 = 1→3→5, l2 = 2→4→6

dummy→ (current points here)

1 < 2: attach 1, l1=3, current=1
3 > 2: attach 2, l2=4, current=2
3 < 4: attach 3, l1=5, current=3
5 > 4: attach 4, l2=6, current=4
5 < 6: attach 5, l1=None, current=5
l1 exhausted: current.next = l2 = 6

Result: 1→2→3→4→5→6 ✓
```

---

## 5.3 — Intersection of Two Linked Lists (LeetCode #160)

### Concept

Two lists intersect if they share a common node (by reference, not value). Once they intersect, they share all subsequent nodes (same object in memory).

**The elegant two-pointer approach:**

```
Key insight:
If list A has length a+c and list B has length b+c (c = common part):
  Pointer 1 traverses A, then B: total = a + c + b
  Pointer 2 traverses B, then A: total = b + c + a = a + b + c

Both travel the same total distance!
If they intersect: at distance a+b+c - (c-1)... actually they meet at
the intersection node exactly because they've both traveled a+b steps
before the common part.
```

```python
def get_intersection_node(headA, headB):
    """
    Find intersection node (by reference).
    Returns None if no intersection.
    
    Time: O(m+n), Space: O(1)
    """
    if not headA or not headB:
        return None
    
    a = headA
    b = headB
    
    # If they don't intersect, both reach None at the same time
    # and the loop exits with a = b = None
    while a != b:
        # Switch to other list head when reaching end
        a = a.next if a is not None else headB
        b = b.next if b is not None else headA
    
    return a   # Either intersection node or None
```

```
Visualization:
A: 4→1→8→4→5
B:    5→6→1→8→4→5
               ↑ Intersection at 8

Lengths: A = 5, B = 6, common = 3
a = 5 - 3 = 2 nodes before intersection
b = 6 - 3 = 3 nodes before intersection

Both pointers travel: 2 + 3 + 3 = 8 steps before meeting at node 8
(After pointer A reaches end, it switches to B; vice versa)
They synchronize and meet at the intersection. ✓
```

---

## 5.4 — Remove Duplicates (LeetCode #83 — Sorted List)

```python
def remove_duplicates_sorted(head):
    """
    Remove duplicates from a SORTED linked list.
    Since it's sorted, duplicates are adjacent.
    
    Time: O(n), Space: O(1)
    """
    current = head
    
    while current is not None and current.next is not None:
        if current.data == current.next.data:
            current.next = current.next.next   # Skip duplicate
            # Don't advance current! Check again (may have 3+ duplicates)
        else:
            current = current.next   # Move forward only when values differ
    
    return head
```

```
Dry run: 1→1→2→3→3→4

current=1: 1==1 → skip, current.next = 2
           State: 1→2→3→3→4 (current still at first 1)
current=1: 1!=2 → advance, current=2
current=2: 2!=3 → advance, current=3
current=3: 3==3 → skip, current.next=4
           State: 1→2→3→4
current=3: 3!=4 → advance, current=4
current=4: current.next=None → stop

Result: 1→2→3→4 ✓
```

**Remove duplicates from UNSORTED list (LeetCode #82):**

```python
def remove_all_duplicates_unsorted(head):
    """
    Remove ALL nodes with duplicate values (not sorted).
    Uses a hash set.
    
    Time: O(n), Space: O(n)
    """
    seen = set()
    dummy = Node(0)
    dummy.next = head
    prev = dummy
    current = head
    
    while current is not None:
        if current.data in seen:
            prev.next = current.next   # Skip this duplicate
        else:
            seen.add(current.data)
            prev = current
        current = current.next
    
    return dummy.next
```

---

## 5.5 — Mini Project: Sorted Contact List Manager

```python
class Contact:
    def __init__(self, name, phone):
        self.name = name
        self.phone = phone
        self.next = None
    
    def __repr__(self):
        return f"{self.name}: {self.phone}"


class SortedContactList:
    """
    Maintains contacts sorted alphabetically by name.
    Demonstrates: sorted insertion, search, deletion.
    """
    
    def __init__(self):
        self.head = None
    
    def insert_sorted(self, name, phone):
        """Insert maintaining alphabetical order."""
        new_contact = Contact(name, phone)
        
        # Insert before head
        if self.head is None or self.head.name >= name:
            new_contact.next = self.head
            self.head = new_contact
            return
        
        # Find insertion point
        current = self.head
        while current.next and current.next.name < name:
            current = current.next
        
        new_contact.next = current.next
        current.next = new_contact
    
    def delete(self, name):
        if not self.head:
            print("List is empty")
            return
        
        if self.head.name == name:
            self.head = self.head.next
            print(f"Deleted {name}")
            return
        
        current = self.head
        while current.next and current.next.name != name:
            current = current.next
        
        if not current.next:
            print(f"{name} not found")
        else:
            current.next = current.next.next
            print(f"Deleted {name}")
    
    def search(self, name):
        current = self.head
        while current:
            if current.name == name:
                return current
            if current.name > name:   # Sorted list — can stop early!
                break
            current = current.next
        return None
    
    def print_contacts(self):
        if not self.head:
            print("No contacts")
            return
        current = self.head
        print("\n=== CONTACTS ===")
        i = 1
        while current:
            print(f"{i}. {current}")
            current = current.next
            i += 1


# Demo
contacts = SortedContactList()
contacts.insert_sorted("Charlie", "555-0003")
contacts.insert_sorted("Alice", "555-0001")
contacts.insert_sorted("Bob", "555-0002")
contacts.insert_sorted("Diana", "555-0004")
contacts.print_contacts()
# Output sorted: Alice, Bob, Charlie, Diana

result = contacts.search("Bob")
print(f"Found: {result}")

contacts.delete("Charlie")
contacts.print_contacts()
```

---

### ✅ SESSION 5 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Remove Nth from End: gap of n between fast & slow
  → Move fast n+1 steps (from dummy), then both together
  → When fast=None, slow is at predecessor of target

✓ Merge Two Sorted: dummy + two-pointer alternating comparison
  → Attach remaining when one list exhausted

✓ Intersection: traverse A then B / B then A simultaneously
  → Meet at intersection or both reach None together

✓ Remove Duplicates (sorted): only advance when values differ
  → Skip on match without advancing current

TWO POINTER GAP FORMULA:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
For nth from end: gap = n (fast is n ahead of slow)
When fast reaches end, slow is at position (length - n)
Use dummy node to handle head deletion case cleanly.
```

---
---

# ╔══════════════════════════════════════════╗
# ║  SESSION 6 — DUMMY NODE MASTERY          ║
# ╚══════════════════════════════════════════╝

---

## 6.1 — The Dummy Node Pattern

### Concept Explanation

**What it is:**
A dummy (sentinel) node is a temporary node with a throwaway value (usually 0 or -1) added BEFORE the real head. You work with `dummy.next` as your actual list head.

**Why it exists:**
Many linked list operations require special handling for the head node because:
- Head has no predecessor
- When deleting head, you must return a new head
- When inserting before head, you must update what `head` points to

Dummy node gives every node (including head) a predecessor, eliminating ALL head-special-case code.

**When to use it:**
- Any operation that might change the head
- Deletion problems (RemoveElements, partitioning)
- Building new lists by appending
- Merge operations

```python
# Without dummy node: complicated
def remove_elements_no_dummy(head, val):
    # Special case: delete head nodes
    while head and head.data == val:
        head = head.next
    
    # Then handle the rest
    current = head
    while current and current.next:
        if current.next.data == val:
            current.next = current.next.next
        else:
            current = current.next
    
    return head   # Head might have changed


# With dummy node: clean and uniform
def remove_elements_dummy(head, val):
    dummy = Node(0)         # Dummy before head
    dummy.next = head
    current = dummy         # Start traversal from dummy
    
    while current.next:
        if current.next.data == val:
            current.next = current.next.next   # Skip node
        else:
            current = current.next
    
    return dummy.next   # Actual head (unchanged or new)
```

**The elegance:** With a dummy node, the first real node is treated like any other — it has a predecessor (`dummy`). No special cases needed.

---

## 6.2 — Partition List (LeetCode #86)

```python
def partition(head, x):
    """
    Partition list around value x.
    All nodes with value < x come before nodes >= x.
    Preserve relative order in both partitions.
    
    Strategy: Build two separate lists, then connect them.
    
    Time: O(n), Space: O(1)
    """
    # Two dummy heads: one for "less than x", one for "greater or equal"
    less_dummy = Node(0)
    greater_dummy = Node(0)
    
    less = less_dummy         # Tail of "less" partition
    greater = greater_dummy   # Tail of "greater" partition
    
    current = head
    
    while current:
        if current.data < x:
            less.next = current    # Append to less partition
            less = less.next
        else:
            greater.next = current  # Append to greater partition
            greater = greater.next
        current = current.next
    
    # Critical: terminate the greater list to avoid cycles!
    greater.next = None
    
    # Connect: less partition → greater partition
    less.next = greater_dummy.next
    
    return less_dummy.next
```

```
Dry run: 1→4→3→2→5→2, x=3

less_dummy → [placeholder]
greater_dummy → [placeholder]

1 < 3: less chain: 1
4 >= 3: greater chain: 4
3 >= 3: greater chain: 4→3
2 < 3: less chain: 1→2
5 >= 3: greater chain: 4→3→5
2 < 3: less chain: 1→2→2

Connect: 1→2→2 + 4→3→5
Result: 1→2→2→4→3→5 ✓
```

---

## 6.3 — Swap Nodes in Pairs (LeetCode #24)

```python
def swap_pairs(head):
    """
    Swap every two adjacent nodes.
    1→2→3→4 → 2→1→4→3
    
    Time: O(n), Space: O(1)
    """
    dummy = Node(0)
    dummy.next = head
    prev = dummy
    
    while prev.next and prev.next.next:
        # Identify the two nodes to swap
        first = prev.next
        second = prev.next.next
        
        # Perform swap
        prev.next = second          # prev → second
        first.next = second.next    # first → what was after second
        second.next = first         # second → first
        
        # Move prev to first (which is now AFTER second)
        prev = first
    
    return dummy.next
```

```
Dry run: dummy→1→2→3→4

prev=dummy, first=1, second=2:
  dummy.next = 2
  1.next = 3 (was second.next)
  2.next = 1
  State: dummy→2→1→3→4, prev moves to 1

prev=1, first=3, second=4:
  1.next = 4
  3.next = None
  4.next = 3
  State: dummy→2→1→4→3→None, prev=3

Loop ends. Return dummy.next = 2 → 2→1→4→3 ✓
```

---

## 6.4 — Remove Elements (LeetCode #203)

```python
def remove_elements(head, val):
    """
    Remove ALL nodes with value equal to val.
    
    Time: O(n), Space: O(1)
    
    Dummy node: handles case where head itself has val.
    """
    dummy = Node(0)
    dummy.next = head
    current = dummy
    
    while current.next:
        if current.next.data == val:
            current.next = current.next.next   # Skip
            # Don't advance current! Next might also be val.
        else:
            current = current.next
    
    return dummy.next
```

---

## 6.5 — Dummy Node: Advanced Merge Variations

**Merge Sort on Linked List (LeetCode #148):**

```python
def sort_list(head):
    """
    Sort linked list using merge sort.
    
    Time: O(n log n), Space: O(log n) for recursion
    
    Technique:
    1. Find middle (slow/fast pointers)
    2. Split into two halves
    3. Recursively sort each half
    4. Merge sorted halves
    """
    # Base case
    if not head or not head.next:
        return head
    
    # Find middle and split
    slow = head
    fast = head.next   # Note: fast starts at head.next for "first middle"
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    # Split: slow is at end of first half
    second_half = slow.next
    slow.next = None   # Cut the list
    
    # Recursively sort
    left = sort_list(head)
    right = sort_list(second_half)
    
    # Merge using dummy node
    return merge(left, right)


def merge(l1, l2):
    dummy = Node(0)
    current = dummy
    
    while l1 and l2:
        if l1.data <= l2.data:
            current.next = l1
            l1 = l1.next
        else:
            current.next = l2
            l2 = l2.next
        current = current.next
    
    current.next = l1 if l1 else l2
    return dummy.next
```

---

## 6.6 — Mini Project: Task Scheduler Using Linked List

```python
from datetime import datetime


class Task:
    def __init__(self, name, priority, due_date):
        self.name = name
        self.priority = priority   # 1 (high) to 5 (low)
        self.due_date = due_date
        self.next = None
    
    def __repr__(self):
        return f"[P{self.priority}] {self.name} (Due: {self.due_date})"


class TaskScheduler:
    """
    Priority-ordered task list using linked list.
    Lower priority number = higher priority = inserted earlier.
    Uses dummy node for clean insertion logic.
    """
    
    def __init__(self):
        self.dummy = Task("DUMMY", 0, "")
        self.dummy.next = None
    
    def add_task(self, name, priority, due_date):
        """Insert task in priority order."""
        new_task = Task(name, priority, due_date)
        
        # Dummy node: no special head case needed
        prev = self.dummy
        current = self.dummy.next
        
        # Find insertion point (maintain priority order)
        while current and current.priority <= priority:
            prev = current
            current = current.next
        
        new_task.next = current
        prev.next = new_task
        print(f"Added task: {new_task}")
    
    def complete_task(self):
        """Remove and return highest priority task (front of list)."""
        if not self.dummy.next:
            print("No tasks!")
            return None
        
        # Dummy makes head deletion clean
        task = self.dummy.next
        self.dummy.next = task.next
        print(f"Completing: {task}")
        return task
    
    def remove_task(self, name):
        """Remove specific task by name."""
        prev = self.dummy
        current = self.dummy.next
        
        while current:
            if current.name == name:
                prev.next = current.next
                print(f"Removed: {name}")
                return
            prev = current
            current = current.next
        
        print(f"Task '{name}' not found")
    
    def view_tasks(self):
        print("\n=== TASK QUEUE ===")
        current = self.dummy.next
        if not current:
            print("  No tasks pending")
            return
        rank = 1
        while current:
            print(f"  {rank}. {current}")
            current = current.next
            rank += 1


# Demo
scheduler = TaskScheduler()
scheduler.add_task("Fix production bug", 1, "2024-01-15")
scheduler.add_task("Write documentation", 4, "2024-01-20")
scheduler.add_task("Code review", 2, "2024-01-16")
scheduler.add_task("Team meeting", 3, "2024-01-17")
scheduler.view_tasks()
scheduler.complete_task()
scheduler.view_tasks()
```

---

### ✅ SESSION 6 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Dummy node = one-liner fix for all head edge cases
✓ Return dummy.next, not dummy, at the end
✓ Use two dummies (less_dummy, greater_dummy) for partitioning
✓ After building a new list with dummy, always terminate with .next = None
✓ Dummy node works best when: building result list, filtering nodes,
  partitioning, any operation that might delete/replace head

DUMMY NODE TEMPLATE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
dummy = Node(0)          # Create sentinel
dummy.next = head        # Connect to real list
current = dummy          # Start from dummy for traversal

# ... operations ...

return dummy.next        # Return actual head

RULE: If you're writing special-case code for head,
      consider using a dummy node instead.
```

---
---

# ╔══════════════════════════════════════════════════════╗
# ║  SESSION 7 — DOUBLY & CIRCULAR LINKED LISTS          ║
# ╚══════════════════════════════════════════════════════╝

---

## 7.1 — Doubly Linked List (DLL)

### Concept Explanation

**What it is:**
A Doubly Linked List has nodes with TWO pointers:
- `next`: points to the next node (forward)
- `prev`: points to the previous node (backward)

**Why it exists:**
Singly linked lists can only traverse forward. If you need to go backward (or efficiently delete a node when you only have a pointer to it), you need `prev` pointers.

**When to use:**
- LRU Cache (both ends need O(1) access)
- Deque (double-ended queue)
- Browser history (true back/forward navigation)
- Text editors (cursor movement)
- Undo/redo stacks

```
DLL Node structure:
NULL ← [prev|data|next] ↔ [prev|data|next] ↔ [prev|data|next] → NULL
         head                                     tail
```

---

### DLL Node and Class

```python
class DLLNode:
    def __init__(self, data):
        self.data = data
        self.prev = None    # Points to previous node
        self.next = None    # Points to next node


class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None    # Maintaining tail gives O(1) insertion at end


    # ─────────────────────────────────────────
    # INSERT AT BEGINNING — O(1)
    # ─────────────────────────────────────────
    def insert_at_beginning(self, data):
        new_node = DLLNode(data)
        
        if self.head is None:
            self.head = new_node
            self.tail = new_node
            return
        
        new_node.next = self.head     # new → old head
        self.head.prev = new_node     # old head ← new
        self.head = new_node          # head = new
    

    # ─────────────────────────────────────────
    # INSERT AT END — O(1) with tail pointer
    # ─────────────────────────────────────────
    def insert_at_end(self, data):
        new_node = DLLNode(data)
        
        if self.tail is None:
            self.head = new_node
            self.tail = new_node
            return
        
        new_node.prev = self.tail     # new ← old tail
        self.tail.next = new_node     # old tail → new
        self.tail = new_node          # tail = new
    

    # ─────────────────────────────────────────
    # DELETE A NODE — O(1) if you have the node!
    # ─────────────────────────────────────────
    def delete_node(self, node):
        """
        Delete a specific node from DLL.
        This is O(1) because we have BOTH prev and next — no traversal needed!
        
        In SLL: you need to find the predecessor first (O(n)).
        In DLL: node.prev IS the predecessor — O(1) access.
        """
        if node is None:
            return
        
        # Update predecessor's next
        if node.prev:
            node.prev.next = node.next
        else:
            # node is head
            self.head = node.next
        
        # Update successor's prev
        if node.next:
            node.next.prev = node.prev
        else:
            # node is tail
            self.tail = node.prev
        
        # Disconnect the node
        node.prev = None
        node.next = None
    

    # ─────────────────────────────────────────
    # FORWARD TRAVERSAL — O(n)
    # ─────────────────────────────────────────
    def print_forward(self):
        current = self.head
        parts = []
        while current:
            parts.append(str(current.data))
            current = current.next
        print("NULL ← " + " ↔ ".join(parts) + " → NULL")
    

    # ─────────────────────────────────────────
    # BACKWARD TRAVERSAL — O(n) (unique to DLL!)
    # ─────────────────────────────────────────
    def print_backward(self):
        current = self.tail
        parts = []
        while current:
            parts.append(str(current.data))
            current = current.prev
        print("NULL ← " + " ↔ ".join(parts) + " → NULL (reversed)")
    

    # ─────────────────────────────────────────
    # INSERT AFTER A NODE — O(1)
    # ─────────────────────────────────────────
    def insert_after(self, target_data, new_data):
        current = self.head
        while current:
            if current.data == target_data:
                new_node = DLLNode(new_data)
                new_node.next = current.next    # new → next
                new_node.prev = current         # new ← current
                if current.next:
                    current.next.prev = new_node  # next ← new
                else:
                    self.tail = new_node          # new is now tail
                current.next = new_node         # current → new
                return
            current = current.next
        print(f"{target_data} not found")


# Demo
dll = DoublyLinkedList()
dll.insert_at_end(10)
dll.insert_at_end(20)
dll.insert_at_end(30)
dll.insert_at_beginning(5)
dll.print_forward()   # NULL ← 5 ↔ 10 ↔ 20 ↔ 30 → NULL
dll.print_backward()  # NULL ← 30 ↔ 20 ↔ 10 ↔ 5 → NULL
```

---

## 7.2 — Circular Linked List

### Concept Explanation

**What it is:**
A Circular Linked List is a variant where the last node points back to the first node instead of NULL. The list has no "end."

**Types:**
- **Singly Circular**: tail.next = head
- **Doubly Circular**: tail.next = head AND head.prev = tail

**Why it exists:**
- Round-robin scheduling (CPU time slicing)
- Circular buffers
- Game development (player turns cycling)
- Media players (loop mode)

```
Singly Circular:
[A] → [B] → [C] → [D] → (back to A)
 ↑_________________________________↩

No NULL — you go around forever unless you track where you started.
```

```python
class CircularLinkedList:
    def __init__(self):
        self.head = None
    
    def insert_at_end(self, data):
        new_node = Node(data)
        
        if self.head is None:
            self.head = new_node
            new_node.next = self.head   # Points to itself!
            return
        
        # Find current tail (last node)
        current = self.head
        while current.next != self.head:   # Stop at tail (points to head)
            current = current.next
        
        current.next = new_node    # Old tail → new node
        new_node.next = self.head  # New node → head (circular!)
    
    def print_circular(self, max_rounds=1):
        """
        Print circular list. Stop after max_rounds to avoid infinite loop.
        """
        if not self.head:
            print("Empty list")
            return
        
        current = self.head
        count = 0
        elements = []
        
        while True:
            elements.append(str(current.data))
            current = current.next
            if current == self.head:  # Back to start
                count += 1
                if count >= max_rounds:
                    break
        
        print(" → ".join(elements) + " → (back to head)")
    
    def traverse_n_times(self, n):
        """Traverse the circular list n nodes (crossing head boundary fine)."""
        if not self.head:
            return []
        
        result = []
        current = self.head
        for _ in range(n):
            result.append(current.data)
            current = current.next
        return result
    
    def detect_and_length(self):
        """
        Circular lists: they ALWAYS have a cycle.
        This finds the length of the cycle.
        """
        if not self.head:
            return 0
        
        slow = self.head
        fast = self.head
        
        # They will always meet in a CLL
        while True:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                break
        
        # Count cycle length
        length = 1
        current = slow.next
        while current != slow:
            length += 1
            current = current.next
        
        return length


# Demo
cll = CircularLinkedList()
cll.insert_at_end(1)
cll.insert_at_end(2)
cll.insert_at_end(3)
cll.insert_at_end(4)
cll.print_circular()
# Output: 1 → 2 → 3 → 4 → (back to head)

print(cll.traverse_n_times(7))  # [1,2,3,4,1,2,3] — crosses head twice
```

---

## 7.3 — LRU Cache Implementation Using DLL + HashMap

This is one of the most important interview problems (LeetCode #146). It uses DLL fundamentally.

```python
class LRUNode:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None


class LRUCache:
    """
    Least Recently Used Cache.
    
    Operations: O(1) get and put.
    
    Data structures:
    - HashMap: key → LRUNode (for O(1) lookup)
    - DLL: maintains order of usage
      - Most recently used: next to left dummy (head side)
      - Least recently used: next to right dummy (tail side)
    
    Layout: left_dummy ↔ [MRU] ↔ ... ↔ [LRU] ↔ right_dummy
    """
    
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}   # key → LRUNode
        
        # Sentinel nodes — never removed
        self.left = LRUNode(0, 0)    # Most recently used boundary
        self.right = LRUNode(0, 0)   # Least recently used boundary
        self.left.next = self.right
        self.right.prev = self.left
    
    def _remove(self, node):
        """Remove a node from the DLL (O(1) because it's DLL)."""
        prev = node.prev
        nxt = node.next
        prev.next = nxt
        nxt.prev = prev
    
    def _insert_left(self, node):
        """Insert node right after left sentinel (most recently used position)."""
        node.prev = self.left
        node.next = self.left.next
        self.left.next.prev = node
        self.left.next = node
    
    def get(self, key):
        if key in self.cache:
            # Move to most recently used position
            node = self.cache[key]
            self._remove(node)
            self._insert_left(node)
            return node.value
        return -1
    
    def put(self, key, value):
        if key in self.cache:
            # Update existing
            self._remove(self.cache[key])
        
        # Insert new node
        node = LRUNode(key, value)
        self.cache[key] = node
        self._insert_left(node)
        
        # Evict LRU if over capacity
        if len(self.cache) > self.capacity:
            lru = self.right.prev   # Node just before right sentinel
            self._remove(lru)
            del self.cache[lru.key]
    
    def display(self):
        """Show cache contents from MRU to LRU."""
        current = self.left.next
        order = []
        while current != self.right:
            order.append(f"({current.key}:{current.value})")
            current = current.next
        print("MRU →", " → ".join(order), "→ LRU")


# Demo
lru = LRUCache(3)
lru.put(1, 100)
lru.put(2, 200)
lru.put(3, 300)
lru.display()   # MRU → (3:300) → (2:200) → (1:100) → LRU

lru.get(1)      # Access 1 → moves to front
lru.display()   # MRU → (1:100) → (3:300) → (2:200) → LRU

lru.put(4, 400) # Over capacity → evict LRU (2)
lru.display()   # MRU → (4:400) → (1:100) → (3:300) → LRU
```

---

## 7.4 — Mini Project: Music Playlist System

```python
class Song:
    def __init__(self, title, artist, duration):
        self.title = title
        self.artist = artist
        self.duration = duration
        self.prev = None
        self.next = None
    
    def __repr__(self):
        return f"'{self.title}' by {self.artist} ({self.duration}s)"


class MusicPlaylist:
    """
    Doubly circular linked list for music playlist.
    Features: next, prev, shuffle, remove, loop mode.
    """
    
    def __init__(self):
        self.head = None
        self.tail = None
        self.current = None
        self.size = 0
    
    def add_song(self, title, artist, duration):
        song = Song(title, artist, duration)
        
        if not self.head:
            self.head = song
            self.tail = song
            song.next = song   # Circular: points to itself
            song.prev = song
            self.current = song
        else:
            # Insert at end (before head in circular sense)
            song.prev = self.tail
            song.next = self.head
            self.tail.next = song
            self.head.prev = song
            self.tail = song
        
        self.size += 1
        print(f"Added: {song}")
    
    def play_current(self):
        if not self.current:
            print("Playlist is empty")
            return
        print(f"▶ Now playing: {self.current}")
    
    def next_song(self):
        if not self.current:
            return
        self.current = self.current.next   # Circular: wraps around!
        self.play_current()
    
    def prev_song(self):
        if not self.current:
            return
        self.current = self.current.prev   # Doubly: can go back!
        self.play_current()
    
    def remove_song(self, title):
        if not self.head:
            print("Playlist empty")
            return
        
        current = self.head
        for _ in range(self.size):
            if current.title == title:
                if self.size == 1:
                    self.head = None
                    self.tail = None
                    self.current = None
                else:
                    current.prev.next = current.next
                    current.next.prev = current.prev
                    if current == self.head:
                        self.head = current.next
                    if current == self.tail:
                        self.tail = current.prev
                    if current == self.current:
                        self.current = current.next
                self.size -= 1
                print(f"Removed: {title}")
                return
            current = current.next
        print(f"'{title}' not found")
    
    def display_playlist(self):
        if not self.head:
            print("Empty playlist")
            return
        print("\n🎵 PLAYLIST:")
        current = self.head
        for i in range(self.size):
            marker = " ◀ NOW PLAYING" if current == self.current else ""
            print(f"  {i+1}. {current}{marker}")
            current = current.next


# Demo
playlist = MusicPlaylist()
playlist.add_song("Bohemian Rhapsody", "Queen", 354)
playlist.add_song("Stairway to Heaven", "Led Zeppelin", 482)
playlist.add_song("Hotel California", "Eagles", 391)
playlist.display_playlist()
playlist.next_song()
playlist.next_song()
playlist.prev_song()
```

---

### ✅ SESSION 7 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ DLL has BOTH prev and next pointers
✓ DLL deletion is O(1) — no need to find predecessor
✓ DLL operations: always update BOTH prev and next
✓ Circular: last node → head (no NULL terminator)
✓ Circular traversal: stop when current == head again
✓ LRU Cache = DLL + HashMap (industry-critical pattern!)

DLL DELETION CHECKLIST:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. node.prev.next = node.next  (predecessor skip)
2. node.next.prev = node.prev  (successor skip back)
3. Handle if node is head (update self.head)
4. Handle if node is tail (update self.tail)

CIRCULAR TRAVERSAL RULE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Stop condition: current.next == head (not current.next == None)
NEVER use None check in circular lists — you'll loop forever!
```

---
---

# ╔══════════════════════════════════════════════════════╗
# ║  SESSION 8 — ADVANCED LINKED LIST PROBLEMS           ║
# ╚══════════════════════════════════════════════════════╝

---

## 8.1 — Copy List with Random Pointer (LeetCode #138)

### Concept Explanation

**What it is:**
Each node has `next` AND `random` pointer. Random can point to ANY node in the list (or None). Create a DEEP COPY — completely independent duplicate list.

**The challenge:**
When copying, the `random` pointer of the copy must point to the COPY of the original's random target — not the original. But you don't know where that copy is yet when you're building nodes.

**Three approaches:**
1. HashMap O(n) space — cleaner
2. Interweaving O(1) space — tricky but elegant
3. Recursive with memo

---

### Approach 1: HashMap (Recommended for Interviews)

```python
class NodeWithRandom:
    def __init__(self, val):
        self.val = val
        self.next = None
        self.random = None


def copy_random_list_hashmap(head):
    """
    Two passes:
    Pass 1: Create all copy nodes. Store original→copy in hashmap.
    Pass 2: Set next and random pointers using hashmap lookup.
    
    Time: O(n), Space: O(n)
    """
    if not head:
        return None
    
    # hashmap: original node → its copy
    node_map = {}
    
    # Pass 1: Create all copies (no connections yet)
    current = head
    while current:
        node_map[current] = NodeWithRandom(current.val)
        current = current.next
    
    # Pass 2: Set connections
    current = head
    while current:
        copy = node_map[current]
        
        # Set next: look up the copy of current's next
        copy.next = node_map.get(current.next)    # None if no next
        
        # Set random: look up the copy of current's random
        copy.random = node_map.get(current.random)  # None if no random
        
        current = current.next
    
    return node_map[head]   # Return copy of original head
```

### Approach 2: Interweaving Nodes O(1) Space

```python
def copy_random_list_optimized(head):
    """
    Three passes with no extra space:
    
    Pass 1: Interleave copies — insert copy after each original
            1 → 1' → 2 → 2' → 3 → 3' → NULL
    
    Pass 2: Set random pointers
            original.random exists → its copy is original.random.next
    
    Pass 3: Separate the two lists
    
    Time: O(n), Space: O(1)
    """
    if not head:
        return None
    
    # Pass 1: Create interleaved copy nodes
    current = head
    while current:
        copy = NodeWithRandom(current.val)
        copy.next = current.next       # copy → original's next
        current.next = copy            # original → copy
        current = copy.next            # advance to original's next
    
    # After pass 1: head → head_copy → node2 → node2_copy → ...
    
    # Pass 2: Set random pointers for copies
    current = head
    while current:
        copy = current.next
        if current.random:
            copy.random = current.random.next   # random's copy is right after random!
        current = copy.next   # Move to next original
    
    # Pass 3: Separate the lists
    dummy = NodeWithRandom(0)
    copy_current = dummy
    current = head
    
    while current:
        copy = current.next
        next_original = copy.next
        
        # Restore original list
        current.next = next_original
        
        # Build copy list
        copy_current.next = copy
        copy_current = copy
        
        current = next_original
    
    return dummy.next
```

```
Visualization of Pass 1:

Before: 1 → 2 → 3 → NULL
After:  1 → 1' → 2 → 2' → 3 → 3' → NULL

Now if 1.random = 3:
  1'.random = 1.random.next = 3.next = 3'  ✓

The key insight: after interleaving, every original node's copy
is exactly ONE step ahead (original.next = copy).
So: original.random.next = copy_of_random. 
```

---

## 8.2 — Rotate List (LeetCode #61)

```python
def rotate_right(head, k):
    """
    Rotate list right by k positions.
    
    1→2→3→4→5, k=2 → 4→5→1→2→3
    
    Insight: Find the new tail (length - k - 1 from start).
    Make list circular, then break at new head.
    
    Time: O(n), Space: O(1)
    """
    if not head or not head.next or k == 0:
        return head
    
    # Step 1: Find length and tail
    length = 1
    tail = head
    while tail.next:
        tail = tail.next
        length += 1
    
    # Step 2: Handle k > length (rotation wraps around)
    k = k % length
    if k == 0:
        return head   # No rotation needed
    
    # Step 3: Make circular
    tail.next = head
    
    # Step 4: Find new tail (length - k - 1 steps from head)
    new_tail_pos = length - k - 1
    new_tail = head
    for _ in range(new_tail_pos):
        new_tail = new_tail.next
    
    # Step 5: Set new head and break circle
    new_head = new_tail.next
    new_tail.next = None   # Break the circle
    
    return new_head
```

```
Dry run: 1→2→3→4→5, k=2

length = 5, tail = Node(5)
k = 2 % 5 = 2

Make circular: 5.next = 1 (1→2→3→4→5→1→...)

New tail position = 5 - 2 - 1 = 2
Traverse 2 steps from head:
  start at 1, step 1: 2, step 2: 3
new_tail = Node(3)
new_head = Node(4) (new_tail.next)
new_tail.next = None (break circle)

Result: 4→5→1→2→3 ✓
```

---

## 8.3 — Flatten a Multilevel Doubly Linked List (LeetCode #430)

```python
class MultiLevelNode:
    def __init__(self, val):
        self.val = val
        self.prev = None
        self.next = None
        self.child = None   # Points to sublist!


def flatten(head):
    """
    Flatten multilevel DLL. When a node has a child,
    insert the entire child list between node and node.next.
    
    Time: O(n), Space: O(1) iterative
    
    Strategy: When we find a child, insert child list inline.
    """
    if not head:
        return None
    
    current = head
    
    while current:
        if current.child:
            child_head = current.child
            current.child = None   # Remove child pointer
            
            # Find the tail of the child list
            child_tail = child_head
            while child_tail.next:
                child_tail = child_tail.next
            
            # Insert child list between current and current.next
            next_node = current.next
            
            current.next = child_head
            child_head.prev = current
            
            child_tail.next = next_node
            if next_node:
                next_node.prev = child_tail
        
        current = current.next
    
    return head
```

```
Visualization:

1 ↔ 2 ↔ 3 ↔ NULL
        ↓ (child)
        4 ↔ 5 ↔ NULL
                ↓ (child)
                6 ↔ NULL

After flatten: 1 ↔ 2 ↔ 3 ↔ 4 ↔ 5 ↔ 6 ↔ NULL
```

---

## 8.4 — Add Two Numbers (LeetCode #2)

```python
def add_two_numbers(l1, l2):
    """
    Numbers stored in reverse order as linked lists.
    2→4→3 = 342, 5→6→4 = 465
    Sum = 807 = 7→0→8
    
    Time: O(max(m,n)), Space: O(max(m,n)+1)
    """
    dummy = Node(0)
    current = dummy
    carry = 0
    
    while l1 or l2 or carry:
        val1 = l1.data if l1 else 0
        val2 = l2.data if l2 else 0
        
        total = val1 + val2 + carry
        carry = total // 10      # Carry to next digit
        digit = total % 10       # Current digit
        
        current.next = Node(digit)
        current = current.next
        
        if l1: l1 = l1.next
        if l2: l2 = l2.next
    
    return dummy.next
```

```
Dry run: l1 = 2→4→3 (342), l2 = 5→6→4 (465)

Step 1: val1=2, val2=5, carry=0, total=7, digit=7, carry=0 → Node(7)
Step 2: val1=4, val2=6, carry=0, total=10, digit=0, carry=1 → Node(0)
Step 3: val1=3, val2=4, carry=1, total=8, digit=8, carry=0 → Node(8)
Both exhausted, carry=0 → stop

Result: 7→0→8 = 807 ✓
```

---

## 8.5 — Reorder List (LeetCode #143)

This combines MULTIPLE patterns: find middle + reverse + merge.

```python
def reorder_list(head):
    """
    1→2→3→4→5 → 1→5→2→4→3
    
    Pattern: interleave first half with reversed second half.
    
    Steps:
    1. Find middle (slow/fast pointers)
    2. Reverse second half
    3. Merge alternately
    
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return
    
    # Step 1: Find middle
    slow, fast = head, head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    
    # Step 2: Reverse second half
    second = slow.next
    slow.next = None   # Cut list in half
    
    prev = None
    while second:
        temp = second.next
        second.next = prev
        prev = second
        second = temp
    second = prev   # second now = reversed second half head
    
    # Step 3: Merge alternately
    first = head
    while second:
        temp1 = first.next
        temp2 = second.next
        
        first.next = second      # first → second
        second.next = temp1      # second → rest of first
        
        first = temp1            # advance first
        second = temp2           # advance second
```

```
1→2→3→4→5

Step 1: middle = 3, second half = 4→5
Step 2: reverse 4→5 → 5→4
Step 3: merge 1→2→3 with 5→4:

first=1, second=5:
  1.next = 5
  5.next = 2 (temp1)
  first=2, second=4

first=2, second=4:
  2.next = 4
  4.next = 3 (temp1)
  first=3, second=None

Result: 1→5→2→4→3 ✓
```

---

### ✅ SESSION 8 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Copy with Random: HashMap (O(n) space) or Interweaving (O(1))
✓ Rotate: make circular, find new tail at (length-k-1), break
✓ Flatten: handle child inline by finding child's tail and splicing
✓ Add Two Numbers: simulate grade-school addition with carry
✓ Reorder: Middle + Reverse + Merge (three-pattern combo)

PATTERN COMBINATIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Most hard problems = basic patterns combined:
  Palindrome = Middle + Reverse + Two-pointer compare
  Reorder    = Middle + Reverse + Merge
  Sort       = Middle + Recursive split + Merge

SPACE OPTIMIZATION TRICK (Random Pointer Copy):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Interleaving: original's copy is always original.next
So: original.random.next = copy of original.random
No extra space needed!
```

---
---

# ╔══════════════════════════════════════════════════╗
# ║  SESSION 9 — LINKED LIST + RECURSION             ║
# ╚══════════════════════════════════════════════════╝

---

## 9.1 — Recursive Thinking for Linked Lists

### Concept Explanation

**Mental model for recursion on linked lists:**

```
f(head) = some_operation(head) + f(head.next)

This means: solve the problem for the current node,
then trust the recursive call to handle the rest of the list.

Base cases:
1. head is None → empty list
2. head.next is None → single node
```

**Call stack visualization:**

```
For a list 1→2→3→NULL, calling f(1):

f(1)
  f(2)
    f(3)
      f(None) ← base case, returns
    Process node 3, return
  Process node 2, return
Process node 1, return

The RECURSION goes DEEP first, then UNWINDS back up.
This is crucial for understanding recursive list operations.
```

---

## 9.2 — Recursive Traversal

```python
# Forward traversal (same order as iteration)
def print_recursive(node):
    if node is None:
        return
    print(node.data, end=" → ")   # Print BEFORE recursive call
    print_recursive(node.next)

# Backward traversal (reverse order — unique capability!)
def print_reverse_recursive(node):
    if node is None:
        return
    print_reverse_recursive(node.next)  # Go to end FIRST
    print(node.data, end=" → ")         # Print AFTER (on unwind)

# This is powerful: recursion gives you access to the "unwind" phase
# The call stack acts like a reverse stack of nodes!
```

```
Dry run: print_reverse_recursive(1→2→3)

Call stack:
print_rev(1) → calls print_rev(2) → calls print_rev(3) → calls print_rev(None) → returns
                                     prints 3 ← unwind
               prints 2 ← unwind
prints 1 ← unwind

Output: 3 → 2 → 1 ✓
```

---

## 9.3 — Recursive Reverse

```python
def reverse_recursive(head):
    """
    The famous recursive reversal.
    
    Base case: single node (it's already reversed, return it)
    Recursive case: reverse rest of list, then fix current connection
    """
    # Base case
    if not head or not head.next:
        return head
    
    # Trust the recursion: reverse everything from head.next onwards
    new_head = reverse_recursive(head.next)
    
    # When we get here (unwind phase), head.next is already reversed
    # head.next still points to the node after head (which is now the TAIL of reversed sublist)
    head.next.next = head    # Make that tail point BACK to head
    head.next = None         # head becomes the new tail (no forward link)
    
    return new_head   # new_head is always the last node of original list
```

```
Detailed unwind for 1→2→3:

Call: reverse(1)
  Call: reverse(2)
    Call: reverse(3)
      Base case: return Node(3) [this becomes new_head forever]
    
    Unwind in reverse(2):
      head=2, head.next=3
      new_head = Node(3)
      head.next.next = 2   → 3.next = 2    [3 now points to 2]
      head.next = None     → 2.next = None  [2 is now the tail]
      return Node(3)
  
  Unwind in reverse(1):
    head=1, head.next=2
    new_head = Node(3)
    head.next.next = 1   → 2.next = 1    [2 now points to 1]
    head.next = None     → 1.next = None  [1 is now the tail]
    return Node(3)

Final: 3→2→1→None ✓
```

---

## 9.4 — Recursive Merge

```python
def merge_recursive(l1, l2):
    """
    Recursively merge two sorted lists.
    
    Elegance: much simpler than iterative — no dummy needed.
    Drawback: O(m+n) stack space.
    
    Time: O(m+n), Space: O(m+n)
    """
    # Base cases
    if not l1:
        return l2
    if not l2:
        return l1
    
    # Pick smaller head and recurse
    if l1.data <= l2.data:
        l1.next = merge_recursive(l1.next, l2)  # l1 comes first
        return l1
    else:
        l2.next = merge_recursive(l1, l2.next)  # l2 comes first
        return l2
```

```
Dry run: merge(1→3, 2→4)

merge(1→3, 2→4):
  1 < 2: 1.next = merge(3→None, 2→4)
    merge(3→None, 2→4):
      3 > 2: 2.next = merge(3→None, 4→None)
        merge(3→None, 4→None):
          3 < 4: 3.next = merge(None, 4→None)
            merge(None, 4): return 4  [base case]
          3.next = 4
          return 3 (→ 4)
      2.next = 3→4
      return 2 (→ 3 → 4)
  1.next = 2→3→4
  return 1 (→ 2 → 3 → 4)

Result: 1→2→3→4 ✓
```

---

## 9.5 — Swap Pairs Recursively (LeetCode #24)

```python
def swap_pairs_recursive(head):
    """
    Swap every two adjacent nodes recursively.
    
    Time: O(n), Space: O(n/2) stack frames
    """
    # Base cases
    if not head or not head.next:
        return head
    
    first = head
    second = head.next
    
    # Recursively swap the rest of the list
    first.next = swap_pairs_recursive(second.next)
    
    # Swap current pair
    second.next = first
    
    return second   # second is the new head of this pair
```

```
Dry run: 1→2→3→4

swap(1→2→3→4):
  first=1, second=2
  1.next = swap(3→4)
    swap(3→4):
      first=3, second=4
      3.next = swap(None) = None
      4.next = 3
      return 4
  1.next = 4 (result of swap(3→4))
  2.next = 1
  return 2

Result: 2 → 1 → 4 → 3 ✓
(Reading: 2.next=1, 1.next=4, 4.next=3)
```

---

## 9.6 — Common Recursion Mistakes

**Mistake 1: Missing or wrong base case**
```python
# WRONG: misses single-node case
def bad_reverse(head):
    if head is None:
        return head  # Only handles empty, not single node!
    # For single node: bad_reverse(head.next) = bad_reverse(None) = None
    # head.next.next = head  → None.next = head → CRASH!

# CORRECT:
def good_reverse(head):
    if not head or not head.next:  # Both empty AND single node
        return head
```

**Mistake 2: Not returning the recursive result**
```python
# WRONG: loses the new head!
def bad_merge(l1, l2):
    if not l1: return l2
    if not l2: return l1
    if l1.data <= l2.data:
        l1.next = bad_merge(l1.next, l2)
        # forgot to return l1 here!

# CORRECT:
    if l1.data <= l2.data:
        l1.next = merge(l1.next, l2)
        return l1    # ← THIS IS CRITICAL
```

**Mistake 3: Stack overflow risk**
```python
# For 10,000+ node lists, recursive approaches risk:
# Python: RecursionError (default limit ~1000)
# Java:   StackOverflowError

# Solution 1: Use iterative approach for production
# Solution 2: Increase recursion limit (Python only, hacky)
import sys
sys.setrecursionlimit(100000)  # Use with caution!

# Solution 3: Tail recursion optimization (Python doesn't do this automatically)
```

---

## 9.7 — Mini Project: Recursive Polynomial Operations

```python
class PolynomialNode:
    """
    Represents a term in a polynomial.
    Example: 3x^2 = PolynomialNode(3, 2)
    List: 3x^2 + 2x + 1 = Node(3,2) → Node(2,1) → Node(1,0)
    """
    def __init__(self, coefficient, exponent):
        self.coeff = coefficient
        self.exp = exponent
        self.next = None
    
    def __repr__(self):
        if self.exp == 0:
            return str(self.coeff)
        elif self.exp == 1:
            return f"{self.coeff}x"
        return f"{self.coeff}x^{self.exp}"


def poly_to_string(head):
    """Recursive string representation."""
    if not head:
        return ""
    rest = poly_to_string(head.next)
    if rest:
        return f"{head} + {rest}"
    return str(head)


def poly_add(p1, p2):
    """
    Add two polynomials (both sorted by exponent descending).
    Recursive approach.
    """
    if not p1:
        return p2
    if not p2:
        return p1
    
    result = PolynomialNode(0, 0)
    
    if p1.exp > p2.exp:
        result.coeff = p1.coeff
        result.exp = p1.exp
        result.next = poly_add(p1.next, p2)
    elif p1.exp < p2.exp:
        result.coeff = p2.coeff
        result.exp = p2.exp
        result.next = poly_add(p1, p2.next)
    else:   # Same exponent — add coefficients
        result.coeff = p1.coeff + p2.coeff
        result.exp = p1.exp
        result.next = poly_add(p1.next, p2.next)
    
    # Don't include zero terms
    if result.coeff == 0:
        return result.next
    
    return result


def poly_evaluate(head, x):
    """Recursively evaluate polynomial at given x."""
    if not head:
        return 0
    return head.coeff * (x ** head.exp) + poly_evaluate(head.next, x)


# Build polynomial: 3x^2 + 2x + 1
def build_poly(terms):
    """terms = list of (coeff, exp) sorted by exp descending"""
    if not terms:
        return None
    node = PolynomialNode(terms[0][0], terms[0][1])
    node.next = build_poly(terms[1:])
    return node


p1 = build_poly([(3, 2), (2, 1), (1, 0)])   # 3x^2 + 2x + 1
p2 = build_poly([(1, 3), (4, 1), (5, 0)])   # x^3 + 4x + 5

print("P1:", poly_to_string(p1))   # 3x^2 + 2x + 1
print("P2:", poly_to_string(p2))   # x^3 + 4x + 5
print("Sum:", poly_to_string(poly_add(p1, p2)))  # x^3 + 3x^2 + 6x + 6
print("P1 at x=2:", poly_evaluate(p1, 2))  # 3*4 + 2*2 + 1 = 17
```

---

### ✅ SESSION 9 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Recursion = solve for head, trust recursion for rest
✓ Printing in reverse = print AFTER recursive call (unwind phase)
✓ ALWAYS have base case for: None AND single node
✓ ALWAYS return the recursive result
✓ Recursive reverse: head.next.next = head; head.next = None
✓ Stack overflow risk for large lists — know when to use iterative

RECURSION TEMPLATE FOR LINKED LISTS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
def f(head):
    # Base case(s)
    if not head: return [something]
    if not head.next: return [something for single node]
    
    # Recursive call on rest
    result = f(head.next)
    
    # Process current node using result
    # ...
    
    # Return
    return [new result]

SPACE COMPLEXITY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Recursive linked list ops → O(n) stack space (n = list length)
Iterative equivalent → O(1) space
For interviews: mention both, code iterative if space-constrained
```

---
---

# ╔══════════════════════════════════════════════════════╗
# ║  SESSION 10 — INTERVIEW MASTERY & MIXED PROBLEMS     ║
# ╚══════════════════════════════════════════════════════╝

---

## 10.1 — LRU Cache (LeetCode #146)

Already covered in Session 7 (DLL section), but here's the complete interview-ready version with all edge cases:

```python
class LRUNode:
    def __init__(self, key=0, val=0):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None


class LRUCache:
    """
    LRU Cache — MUST KNOW for FAANG interviews.
    
    get(key) → O(1)
    put(key, value) → O(1)
    
    Core data structures:
    1. HashMap (dict): key → node  [for O(1) access]
    2. Doubly Linked List: [MRU...LRU] order [for O(1) eviction]
    
    Convention:
      left sentinel ↔ [MRU node] ↔ ... ↔ [LRU node] ↔ right sentinel
    
    On get: move accessed node to MRU position (after left)
    On put: add new node at MRU position; evict from LRU position if full
    """
    
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = {}
        
        # Sentinels
        self.left = LRUNode()   # MRU boundary
        self.right = LRUNode()  # LRU boundary
        self.left.next = self.right
        self.right.prev = self.left
    
    def _remove(self, node):
        """Remove node from DLL — O(1)"""
        node.prev.next = node.next
        node.next.prev = node.prev
    
    def _insert_mru(self, node):
        """Insert at MRU position (right after left sentinel) — O(1)"""
        node.next = self.left.next
        node.prev = self.left
        self.left.next.prev = node
        self.left.next = node
    
    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._remove(node)       # Remove from current position
        self._insert_mru(node)   # Re-insert at MRU
        return node.val
    
    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self._remove(self.cache[key])
        
        node = LRUNode(key, value)
        self.cache[key] = node
        self._insert_mru(node)
        
        if len(self.cache) > self.capacity:
            # Evict LRU (node just before right sentinel)
            lru = self.right.prev
            self._remove(lru)
            del self.cache[lru.key]
```

---

## 10.2 — Merge K Sorted Lists (LeetCode #23)

```python
import heapq

def merge_k_lists(lists):
    """
    Merge K sorted linked lists into one sorted list.
    
    Approach 1: Min Heap (Priority Queue) — O(N log K)
    N = total nodes, K = number of lists
    
    Approach 2: Divide and conquer — O(N log K) but less heap overhead
    """
    # Approach 1: Min Heap
    # Python heapq is min-heap by default
    # Problem: Nodes need to be comparable
    # Solution: Push (value, index, node) tuples
    
    dummy = Node(0)
    current = dummy
    heap = []
    
    # Initialize heap with heads of all lists
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.data, i, node))
    
    while heap:
        val, i, node = heapq.heappop(heap)
        current.next = node
        current = current.next
        
        if node.next:
            heapq.heappush(heap, (node.next.data, i, node.next))
    
    return dummy.next


def merge_k_lists_divide_conquer(lists):
    """
    Divide and conquer approach.
    Repeatedly merge pairs of lists.
    
    Like merge sort: split → merge → merge merged results
    
    Time: O(N log K), Space: O(log K) recursion
    """
    if not lists:
        return None
    
    def merge_two(l1, l2):
        dummy = Node(0)
        current = dummy
        while l1 and l2:
            if l1.data <= l2.data:
                current.next = l1
                l1 = l1.next
            else:
                current.next = l2
                l2 = l2.next
            current = current.next
        current.next = l1 or l2
        return dummy.next
    
    # Iteratively merge pairs
    while len(lists) > 1:
        merged = []
        for i in range(0, len(lists), 2):
            l1 = lists[i]
            l2 = lists[i + 1] if i + 1 < len(lists) else None
            merged.append(merge_two(l1, l2))
        lists = merged
    
    return lists[0]
```

---

## 10.3 — Sort List (LeetCode #148)

```python
def sort_list(head):
    """
    Sort linked list using Merge Sort.
    
    Why merge sort (not quick sort)?
    - Merge sort is stable and O(n log n) guaranteed
    - No random pivot issues with linked lists
    - Merge is natural for linked lists (no extra array needed)
    
    Time: O(n log n), Space: O(log n) for recursion
    """
    if not head or not head.next:
        return head
    
    # Find middle (split point)
    slow, fast = head, head.next
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    # Split
    mid = slow.next
    slow.next = None
    
    # Recursively sort
    left = sort_list(head)
    right = sort_list(mid)
    
    # Merge
    dummy = Node(0)
    tail = dummy
    while left and right:
        if left.data <= right.data:
            tail.next = left
            left = left.next
        else:
            tail.next = right
            right = right.next
        tail = tail.next
    tail.next = left or right
    
    return dummy.next
```

---

## 10.4 — Pattern Recognition: How to Approach Any Linked List Problem

```
DECISION TREE FOR LINKED LIST PROBLEMS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Q: Does it involve detecting a cycle?
→ YES: Use Fast & Slow pointers (Floyd's)

Q: Does it involve finding middle?
→ YES: Fast & Slow (fast=2x slow, stop when fast at end)

Q: Does it involve the nth from end?
→ YES: Two pointers with n-gap

Q: Does it involve reversal?
→ YES: Learn all 3 reversal types
    → Full reversal: iterative 3-pointer
    → Partial reversal: reverse_between
    → K-group: check k exists, reverse, recurse

Q: Does it involve merging?
→ YES: Dummy + two-pointer alternating comparison

Q: Does it involve building a new list?
→ YES: Dummy node at the front

Q: Does it involve deletion?
→ YES: Dummy node (for head deletion) + prev pointer tracking

Q: Does it involve duplicates?
→ YES: 
    Sorted: advance only when values differ
    Unsorted: HashSet

Q: Does it involve random/arbitrary pointers?
→ YES: HashMap (original → copy) or interweaving

Q: Does it involve multiple complex steps?
→ YES: Combine patterns (Middle + Reverse + Merge = Palindrome/Reorder)
```

---

## 10.5 — Clean Coding Under Pressure: Interview Template

```python
"""
INTERVIEW APPROACH CHECKLIST (for every linked list problem):

1. UNDERSTAND (2 min):
   - What type of list? (singly, doubly, circular)
   - What does "delete", "return", "modify" mean?
   - What's the output? (new head? boolean? value?)

2. EXAMPLES (2 min):
   - Draw 2-3 examples on paper/whiteboard
   - Include edge cases: empty, single node, two nodes

3. EDGE CASES (1 min):
   - head = None
   - head.next = None
   - Operation on first/last node
   - n > length (for position-based problems)
   - k = 0 or k = length (for rotation)

4. PATTERN IDENTIFICATION (1 min):
   - Which of the 8 core patterns applies?
   - Do I need dummy node?

5. COMPLEXITY ANALYSIS (before coding):
   - State expected Time and Space complexity
   - "I'll aim for O(n) time and O(1) space"

6. CODE (10-15 min):
   - Write cleanly
   - Name variables clearly (prev, current, fast, slow, dummy)
   - Add minimal but clear comments

7. TEST (2-3 min):
   - Dry run your code on the example
   - Test edge cases mentally
"""


# CLEAN CODING TEMPLATE — use this structure every time

def solve(head):
    # Edge case handling first
    if not head:
        return None
    if not head.next:
        return head
    
    # Setup: dummy node if needed, pointers
    dummy = Node(0)
    dummy.next = head
    
    # Core logic with descriptive variable names
    slow = dummy
    fast = dummy
    
    # Main loop with clear condition
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    # Post-processing
    result = ...
    
    return dummy.next  # or result
```

---

## 10.6 — Dry Run Skill: Mastering Pointer Tracing

```
This is the #1 separating skill in interviews.

HOW TO DRY RUN (practice this for every problem):

1. Draw boxes for each node: [data|next]
2. Draw arrows for each pointer
3. Label: head, tail, slow, fast, prev, current, dummy
4. For each line of code:
   - Identify which pointer changes
   - Draw the new arrow
   - Cross out the old arrow
5. After each iteration, redraw the full state

EXAMPLE DRY RUN FORMAT:

Problem: Reverse 1→2→3

Before:
  head → [1]→[2]→[3]→NULL
  prev = NULL, current = [1]

Iteration 1:
  next_node = [2]            (saved)
  [1].next = NULL            (reversed)
  prev = [1]
  current = [2]
  
  State: NULL←[1]    [2]→[3]→NULL
              prev    cur

Iteration 2:
  next_node = [3]
  [2].next = [1]             (reversed)
  prev = [2]
  current = [3]
  
  State: NULL←[1]←[2]    [3]→NULL
                  prev    cur

Iteration 3:
  next_node = NULL
  [3].next = [2]             (reversed)
  prev = [3]
  current = NULL

  State: NULL←[1]←[2]←[3]
                          prev (new head!)

Return prev = [3] ✓
```

---

## 10.7 — Speed Training: 25-Minute Medium Problem Protocol

```
0:00 - 2:00   → Read problem carefully, ask clarifying questions
2:00 - 5:00   → Draw examples on paper (3 examples minimum)
5:00 - 7:00   → Identify pattern, plan approach, state complexity
7:00 - 8:00   → Write function signature and handle edge cases
8:00 - 20:00  → Implement solution cleanly
20:00 - 23:00 → Dry run on example
23:00 - 25:00 → Test edge cases, complexity discussion
```

---

## 10.8 — Mini Project: Mini In-Memory Database Using Linked Lists

```python
"""
In-memory key-value database using:
- Hash table with linked list chaining (for collisions)
- Ordered index using sorted linked list
- Simple CRUD operations
"""

class KVNode:
    """Node in hash table chain."""
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.next = None


class IndexNode:
    """Node in sorted index."""
    def __init__(self, key):
        self.key = key
        self.next = None


class MiniDB:
    """
    Mini in-memory database.
    Features:
    - O(1) average insert/lookup (hash table)
    - Collision handling via chaining (linked list)
    - Sorted key index (sorted linked list)
    - Full CRUD
    """
    
    BUCKET_SIZE = 16
    
    def __init__(self):
        # Hash table: array of linked list heads
        self.buckets = [None] * self.BUCKET_SIZE
        
        # Sorted index (sorted linked list)
        self.index_head = None
        
        # Count
        self.size = 0
    
    def _hash(self, key):
        """Simple hash function."""
        return hash(key) % self.BUCKET_SIZE
    
    def put(self, key, value):
        """Insert or update key-value pair."""
        bucket_idx = self._hash(key)
        
        # Check if key exists in chain
        current = self.buckets[bucket_idx]
        while current:
            if current.key == key:
                current.value = value  # Update
                return
            current = current.next
        
        # Key doesn't exist: insert at head of chain
        new_node = KVNode(key, value)
        new_node.next = self.buckets[bucket_idx]
        self.buckets[bucket_idx] = new_node
        
        # Update sorted index
        self._insert_index(key)
        self.size += 1
    
    def get(self, key):
        """Retrieve value for key."""
        bucket_idx = self._hash(key)
        current = self.buckets[bucket_idx]
        
        while current:
            if current.key == key:
                return current.value
            current = current.next
        
        return None  # Key not found
    
    def delete(self, key):
        """Delete a key-value pair."""
        bucket_idx = self._hash(key)
        
        # Handle head of chain
        if self.buckets[bucket_idx] and self.buckets[bucket_idx].key == key:
            self.buckets[bucket_idx] = self.buckets[bucket_idx].next
            self._remove_index(key)
            self.size -= 1
            return True
        
        # Find in chain
        prev = None
        current = self.buckets[bucket_idx]
        while current:
            if current.key == key:
                if prev:
                    prev.next = current.next
                self._remove_index(key)
                self.size -= 1
                return True
            prev = current
            current = current.next
        
        return False  # Not found
    
    def _insert_index(self, key):
        """Insert into sorted index."""
        new_node = IndexNode(key)
        if not self.index_head or self.index_head.key >= key:
            new_node.next = self.index_head
            self.index_head = new_node
            return
        current = self.index_head
        while current.next and current.next.key < key:
            current = current.next
        new_node.next = current.next
        current.next = new_node
    
    def _remove_index(self, key):
        """Remove from sorted index."""
        if not self.index_head:
            return
        if self.index_head.key == key:
            self.index_head = self.index_head.next
            return
        current = self.index_head
        while current.next and current.next.key != key:
            current = current.next
        if current.next:
            current.next = current.next.next
    
    def list_all_sorted(self):
        """List all keys in sorted order using index."""
        print(f"\n=== DATABASE ({self.size} records) ===")
        current = self.index_head
        while current:
            value = self.get(current.key)
            print(f"  {current.key}: {value}")
            current = current.next
    
    def display_buckets(self):
        """Show internal hash table structure."""
        print("\n=== BUCKET STRUCTURE ===")
        for i in range(self.BUCKET_SIZE):
            if self.buckets[i]:
                chain = []
                current = self.buckets[i]
                while current:
                    chain.append(f"({current.key}:{current.value})")
                    current = current.next
                print(f"  Bucket {i:2d}: {' → '.join(chain)}")


# Demo
db = MiniDB()
db.put("alice", 95)
db.put("charlie", 87)
db.put("bob", 92)
db.put("diana", 78)
db.put("alice", 98)  # Update alice
db.list_all_sorted()  # Sorted by name

print(f"\nGet alice: {db.get('alice')}")    # 98 (updated)
print(f"Get eve: {db.get('eve')}")          # None

db.delete("charlie")
db.list_all_sorted()
db.display_buckets()
```

---

### ✅ SESSION 10 — QUICK REVISION SUMMARY

```
KEY TAKEAWAYS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ LRU Cache = DLL (for order) + HashMap (for O(1) access)
✓ Merge K lists: Min Heap O(N log K) OR Divide & Conquer
✓ Sort list: Merge sort preferred (no quicksort for linked lists)
✓ Pattern recognition is the most valuable interview skill
✓ Dry run every single pointer change on paper
✓ 25-minute protocol: 2 min understand, 3 min examples, code, test

PATTERN LOOKUP TABLE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Problem type         → Pattern to use
─────────────────────────────────────────
Detect cycle         → Fast & Slow
Find cycle start     → Fast & Slow + Phase 2
Find middle          → Fast & Slow
Nth from end         → Two pointers (n-gap)
Reverse list         → 3-pointer iterative
Partial reverse      → Insert-at-front technique
K-group reverse      → Check k, reverse, recurse
Merge sorted lists   → Dummy + alternating compare
Merge K lists        → Min heap or divide & conquer
Build result list    → Dummy node at front
Delete head safely   → Dummy node
Deep copy            → HashMap or interweaving
Sort list            → Merge sort
LRU/LFU cache        → DLL + HashMap
```

---
---

# ╔═══════════════════════════════════════════════════════╗
# ║              FINAL MASTERY SECTION                    ║
# ╚═══════════════════════════════════════════════════════╝

---

## 📋 COMPLETE REVISION SHEET — ONE PAGE REFERENCE

---

### Core Operations Complexity

```
Operation                        | Time    | Space  | Notes
─────────────────────────────────────────────────────────────────
Access by index                  | O(n)    | O(1)   | Must traverse
Search (unsorted)                | O(n)    | O(1)   | Linear scan
Search (sorted)                  | O(n)    | O(1)   | Stop early
Insert at beginning              | O(1)    | O(1)   | Update head
Insert at end (no tail)          | O(n)    | O(1)   | Find tail
Insert at end (with tail)        | O(1)    | O(1)   | Direct append
Insert at position               | O(n)    | O(1)   | Find predecessor
Delete head                      | O(1)    | O(1)   | Update head
Delete tail (SLL)                | O(n)    | O(1)   | Find second-to-last
Delete tail (DLL)                | O(1)    | O(1)   | Use tail.prev
Delete middle (given pointer)    | O(1)    | O(1)   | Redirect pointers
Delete by value                  | O(n)    | O(1)   | Find + remove
Reverse                          | O(n)    | O(1)   | Iterative
Reverse (recursive)              | O(n)    | O(n)   | Stack frames
Detect cycle                     | O(n)    | O(1)   | Floyd's
Find middle                      | O(n)    | O(1)   | Fast/slow
Merge two sorted                 | O(m+n)  | O(1)   | Two pointers
```

---

### 8 Core Patterns: Quick Reference

```python
# ─────────────────────────────────────────────────────
# PATTERN 1: FAST & SLOW POINTERS
# ─────────────────────────────────────────────────────
slow = head
fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:  # Cycle detected
        break

# ─────────────────────────────────────────────────────
# PATTERN 2: DUMMY NODE
# ─────────────────────────────────────────────────────
dummy = Node(0)
dummy.next = head
current = dummy
# ... process ...
return dummy.next

# ─────────────────────────────────────────────────────
# PATTERN 3: ITERATIVE REVERSAL
# ─────────────────────────────────────────────────────
prev = None
current = head
while current:
    next_node = current.next
    current.next = prev
    prev = current
    current = next_node
return prev  # new head

# ─────────────────────────────────────────────────────
# PATTERN 4: TWO POINTER GAP (nth from end)
# ─────────────────────────────────────────────────────
dummy = Node(0)
dummy.next = head
fast = slow = dummy
for _ in range(n + 1):
    fast = fast.next
while fast:
    slow = slow.next
    fast = fast.next
slow.next = slow.next.next
return dummy.next

# ─────────────────────────────────────────────────────
# PATTERN 5: MERGE TWO SORTED
# ─────────────────────────────────────────────────────
dummy = Node(0)
current = dummy
while l1 and l2:
    if l1.data <= l2.data:
        current.next = l1; l1 = l1.next
    else:
        current.next = l2; l2 = l2.next
    current = current.next
current.next = l1 or l2
return dummy.next

# ─────────────────────────────────────────────────────
# PATTERN 6: CYCLE DETECTION + START
# ─────────────────────────────────────────────────────
# Phase 1: Detect
slow = fast = head
while fast and fast.next:
    slow = slow.next; fast = fast.next.next
    if slow == fast: break
# Phase 2: Find start
p1 = head; p2 = slow
while p1 != p2:
    p1 = p1.next; p2 = p2.next
return p1  # cycle start

# ─────────────────────────────────────────────────────
# PATTERN 7: DLL REMOVE + INSERT (LRU)
# ─────────────────────────────────────────────────────
def remove(node):
    node.prev.next = node.next
    node.next.prev = node.prev

def insert_after(sentinel, node):
    node.next = sentinel.next
    node.prev = sentinel
    sentinel.next.prev = node
    sentinel.next = node

# ─────────────────────────────────────────────────────
# PATTERN 8: RECURSIVE PROCESSING
# ─────────────────────────────────────────────────────
def process(head):
    if not head or not head.next:
        return head
    result = process(head.next)
    # ... fix current using result ...
    return new_head
```

---

## 📝 COMMON INTERVIEW QUESTIONS WITH DETAILED ANSWERS

---

**Q1: What is the time complexity of inserting a node in a linked list?**

```
A: It depends on WHERE:
   - At the beginning: O(1) — just redirect head
   - At the end without tail pointer: O(n) — must traverse to find tail
   - At the end with tail pointer: O(1) — direct append
   - At position k: O(k) — must traverse k steps to find predecessor
   - Given a pointer to the predecessor: O(1) — just redirect pointers

KEY DISTINCTION: Insertion itself is O(1) (just pointer operations).
Finding the insertion point is O(n). Most interview answers accept O(n)
for "insert at position k" since traversal is included.
```

**Q2: How does Floyd's Cycle Detection work? Why does it work?**

```
A: Two pointers: slow (1 step) and fast (2 steps) from the same start.

WHY it detects cycles: If no cycle, fast reaches None. If cycle exists,
both enter the cycle. Inside the cycle, fast gains 1 step on slow per
iteration. The gap between them decreases by 1 each step. When gap = 0,
they meet. Guaranteed.

WHY meeting point helps find cycle start:
Let F = head to cycle start, C = cycle length, a = cycle start to meeting.
slow traveled: F + a
fast traveled: F + a + C (one full extra loop)
Since fast = 2 × slow: 2(F+a) = F+a+C → F = C-a

This means distance from head to cycle start = distance from meeting
point to cycle start (going forward). Two pointers from each location
at equal speed MUST meet at the cycle start.
```

**Q3: Why use a dummy node in linked list problems?**

```
A: Dummy node eliminates head-special-case handling.

Without dummy: You need separate code for:
  - Deleting the head node
  - Inserting before the head
  - Building a result list starting from the first element

With dummy: Head is treated identically to any other node because
dummy acts as its predecessor. You always have a node before head.

When to use: ANY problem where head might change, or you're building
a new list by appending. Return dummy.next as the actual result head.

Example: Remove all occurrences of value 5 from a list.
With dummy: uniform loop, no head-special-case.
Without dummy: while-loop to skip head nodes, then another loop.
```

**Q4: What is the difference between SLL, DLL, and CLL?**

```
A:
SLL (Singly): Each node has ONLY next pointer.
  + Less memory (no prev pointer)
  + Simpler implementation
  - Can only traverse forward
  - Deletion requires predecessor (O(n) to find)

DLL (Doubly): Each node has both prev and next.
  + Bidirectional traversal
  + O(1) deletion when you have the node (no predecessor search)
  + O(1) insertion/deletion at both ends with head+tail pointers
  - More memory (extra pointer per node)
  - More complex pointer management (update both prev and next)
  Use for: LRU cache, browser history, text editor buffers

CLL (Circular): Last node.next = head (no NULL terminator).
  + Natural for round-robin, cyclic processing
  - Traversal requires cycle detection logic (stop when current == head)
  - Risk of infinite loops if not careful
  Use for: OS scheduling, game turn management, music loops
```

**Q5: How do you find the middle of a linked list in one pass?**

```
A: Fast & Slow pointers.

slow = head (moves 1 step)
fast = head (moves 2 steps)

When fast reaches end, slow is at middle.

For even-length list (say n=4):
  slow ends at position 2 (second middle = n/2 + 1)
  Unless you use: fast = head.next initially, which gives first middle.

Example: 1→2→3→4
  With fast=head: slow ends at 3 (second middle)
  With fast=head.next: slow ends at 2 (first middle)

Interview answer: state both variants and which you're using.
```

**Q6: How do you reverse a linked list? Give all approaches.**

```
A: Three approaches:

1. ITERATIVE (O(n) time, O(1) space — BEST for interviews):
   prev=None, current=head
   while current:
     next_node = current.next
     current.next = prev
     prev = current
     current = next_node
   return prev

2. RECURSIVE (O(n) time, O(n) space — call stack):
   if not head or not head.next: return head
   new_head = reverse(head.next)
   head.next.next = head
   head.next = None
   return new_head

3. STACK (O(n) time, O(n) space — educational):
   Push all nodes onto stack.
   Pop and reconnect.
   (Not preferred — uses extra space AND doesn't save over recursive)

For interviews: implement iterative first, mention recursive as variant.
```

**Q7: Solve LeetCode #206 (Reverse Linked List) in real-time.**

```python
def reverseList(head):
    prev, curr = None, head
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    return prev
# Time: O(n), Space: O(1)
```

**Q8: What are the common mistakes in linked list interview problems?**

```
A:
1. Traversing with head directly → lose the list entry point
   Fix: current = head (never move head)

2. Not handling empty list (head = None) → NullPointerException
   Fix: Always check if head is None first

3. Off-by-one in position traversal → wrong node deleted/accessed
   Fix: Stop at (target - 1), not target

4. Losing next pointer before redirecting current.next
   Fix: Always save next_node = current.next BEFORE current.next = something

5. Forgetting to terminate cycle → broken list or infinite loop
   Fix: After operations like rotation or flattening, ensure last node.next = None

6. Reference vs value equality for cycle detection
   Fix: Use `slow == fast` (same object), NOT `slow.data == fast.data`

7. Not handling the tail separately in DLL operations
   Fix: Always update self.tail when deleting/inserting at end

8. Returning head instead of dummy.next after dummy-node operations
   Fix: Return dummy.next, not dummy or head
```

**Q9: What is the space complexity of recursive linked list solutions?**

```
A: O(n) for most recursive linked list algorithms.

Each function call adds a frame to the call stack. For a list of n nodes,
there are n recursive calls before hitting the base case. So the call
stack holds n frames simultaneously = O(n) space.

Exceptions:
- Tail-recursive functions (theoretical — Python/Java don't optimize)
- Problems where recursion depth is bounded by something other than n

For interview: if asked for O(1) space, use iterative. Mention that
recursive is O(n) space due to call stack — this shows depth of knowledge.
```

**Q10: Design an LRU Cache. Explain every design decision.**

```
A:
REQUIREMENTS: get(key) and put(key, value), both O(1).

DATA STRUCTURES:
1. HashMap (dict): key → node reference for O(1) lookup
2. Doubly Linked List: maintains LRU order
   - Most recently used: near left sentinel
   - Least recently used: near right sentinel
   - Two sentinel nodes prevent edge cases at boundaries

WHY DLL (not SLL)?
- Deletion is O(1) with DLL (have prev pointer, no traversal needed)
- With SLL, deletion requires finding predecessor = O(n)

WHY SENTINELS?
- Left and right dummy nodes mean we never have empty-head/empty-tail
  special cases. Every real node always has prev and next.

OPERATIONS:
get(key):
  1. If key not in cache: return -1
  2. Remove node from current position (O(1) with DLL)
  3. Re-insert at MRU position (O(1))
  4. Return value

put(key, value):
  1. If key exists: remove old node
  2. Create new node, insert at MRU position, add to hashmap
  3. If over capacity: remove LRU node (right.prev), delete from hashmap

All operations: O(1) time, O(capacity) space.
```

---

## 🛠️ MINI PROJECTS — BUILD THESE FOR DEEP MASTERY

---

### Project 1: Complete Linked List Library (Python)

```python
"""
Build a production-grade linked list library with:
- All insertion variants
- All deletion variants  
- Reversal (iterative + recursive)
- Sort (merge sort)
- Cycle detection
- Type hints
- Error handling
- __repr__ and __iter__
"""

from typing import Optional, Any, Iterator


class Node:
    def __init__(self, data: Any):
        self.data = data
        self.next: Optional['Node'] = None


class LinkedList:
    def __init__(self):
        self._head: Optional[Node] = None
        self._size: int = 0
    
    def __len__(self) -> int:
        return self._size
    
    def __iter__(self) -> Iterator:
        current = self._head
        while current:
            yield current.data
            current = current.next
    
    def __repr__(self) -> str:
        elements = list(self)
        return " → ".join(map(str, elements)) + " → NULL"
    
    def __contains__(self, item) -> bool:
        return self.search(item) != -1
    
    @property
    def head(self) -> Optional[Node]:
        return self._head
    
    def is_empty(self) -> bool:
        return self._head is None
    
    def push(self, data: Any) -> None:
        """Insert at beginning O(1)"""
        node = Node(data)
        node.next = self._head
        self._head = node
        self._size += 1
    
    def append(self, data: Any) -> None:
        """Insert at end O(n)"""
        node = Node(data)
        if not self._head:
            self._head = node
        else:
            current = self._head
            while current.next:
                current = current.next
            current.next = node
        self._size += 1
    
    def insert_at(self, index: int, data: Any) -> None:
        """Insert at index O(n)"""
        if index < 0 or index > self._size:
            raise IndexError(f"Index {index} out of range [0, {self._size}]")
        if index == 0:
            self.push(data)
            return
        node = Node(data)
        current = self._head
        for _ in range(index - 1):
            current = current.next
        node.next = current.next
        current.next = node
        self._size += 1
    
    def pop_front(self) -> Any:
        """Remove and return head O(1)"""
        if self.is_empty():
            raise IndexError("Pop from empty list")
        data = self._head.data
        self._head = self._head.next
        self._size -= 1
        return data
    
    def pop_back(self) -> Any:
        """Remove and return tail O(n)"""
        if self.is_empty():
            raise IndexError("Pop from empty list")
        if not self._head.next:
            data = self._head.data
            self._head = None
            self._size -= 1
            return data
        current = self._head
        while current.next.next:
            current = current.next
        data = current.next.data
        current.next = None
        self._size -= 1
        return data
    
    def remove(self, data: Any) -> bool:
        """Remove first occurrence of data O(n)"""
        dummy = Node(0)
        dummy.next = self._head
        current = dummy
        while current.next:
            if current.next.data == data:
                current.next = current.next.next
                self._head = dummy.next
                self._size -= 1
                return True
            current = current.next
        return False
    
    def search(self, data: Any) -> int:
        """Return index of data, -1 if not found O(n)"""
        current = self._head
        for i, val in enumerate(self):
            if val == data:
                return i
        return -1
    
    def reverse(self) -> None:
        """Reverse in-place O(n)"""
        prev = None
        current = self._head
        while current:
            next_node = current.next
            current.next = prev
            prev = current
            current = next_node
        self._head = prev
    
    def has_cycle(self) -> bool:
        """Detect cycle O(n)"""
        slow = fast = self._head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                return True
        return False
    
    def sort(self) -> None:
        """Sort using merge sort O(n log n)"""
        self._head = self._merge_sort(self._head)
    
    def _merge_sort(self, head: Optional[Node]) -> Optional[Node]:
        if not head or not head.next:
            return head
        mid = self._get_mid(head)
        right_half = mid.next
        mid.next = None
        left = self._merge_sort(head)
        right = self._merge_sort(right_half)
        return self._merge(left, right)
    
    def _get_mid(self, head: Node) -> Node:
        slow, fast = head, head.next
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        return slow
    
    def _merge(self, l1: Optional[Node], l2: Optional[Node]) -> Optional[Node]:
        dummy = Node(0)
        curr = dummy
        while l1 and l2:
            if l1.data <= l2.data:
                curr.next = l1; l1 = l1.next
            else:
                curr.next = l2; l2 = l2.next
            curr = curr.next
        curr.next = l1 or l2
        return dummy.next
    
    def to_list(self) -> list:
        return list(self)


# Usage
ll = LinkedList()
for val in [3, 1, 4, 1, 5, 9, 2, 6]:
    ll.append(val)
print(ll)              # 3 → 1 → 4 → 1 → 5 → 9 → 2 → 6 → NULL
ll.sort()
print(ll)              # 1 → 1 → 2 → 3 → 4 → 5 → 6 → 9 → NULL
ll.reverse()
print(ll)              # 9 → 6 → 5 → 4 → 3 → 2 → 1 → 1 → NULL
print(5 in ll)         # True
print(len(ll))         # 8
```

---

### Project 2: Polynomial Calculator (Session 9 Extended)

Build on the polynomial example from Session 9. Add multiplication and differentiation.

```python
class Polynomial:
    """
    Polynomial stored as sorted linked list of terms.
    """
    class Term:
        def __init__(self, coeff, exp):
            self.coeff = coeff
            self.exp = exp
            self.next = None
    
    def __init__(self):
        self.head = None
    
    def add_term(self, coeff, exp):
        if coeff == 0:
            return
        new_term = self.Term(coeff, exp)
        # Insert in descending exponent order
        if not self.head or self.head.exp < exp:
            new_term.next = self.head
            self.head = new_term
            return
        current = self.head
        # Find insertion point
        while current.next and current.next.exp > exp:
            current = current.next
        # Same exponent: combine
        if current.next and current.next.exp == exp:
            current.next.coeff += coeff
            if current.next.coeff == 0:
                current.next = current.next.next
        else:
            new_term.next = current.next
            current.next = new_term
    
    def __add__(self, other):
        result = Polynomial()
        a = self.head
        b = other.head
        while a and b:
            if a.exp > b.exp:
                result.add_term(a.coeff, a.exp); a = a.next
            elif a.exp < b.exp:
                result.add_term(b.coeff, b.exp); b = b.next
            else:
                result.add_term(a.coeff + b.coeff, a.exp)
                a = a.next; b = b.next
        while a:
            result.add_term(a.coeff, a.exp); a = a.next
        while b:
            result.add_term(b.coeff, b.exp); b = b.next
        return result
    
    def differentiate(self):
        """d/dx of polynomial."""
        result = Polynomial()
        current = self.head
        while current:
            if current.exp > 0:
                result.add_term(current.coeff * current.exp, current.exp - 1)
            current = current.next
        return result
    
    def evaluate(self, x):
        total = 0
        current = self.head
        while current:
            total += current.coeff * (x ** current.exp)
            current = current.next
        return total
    
    def __repr__(self):
        if not self.head:
            return "0"
        terms = []
        current = self.head
        while current:
            if current.exp == 0:
                terms.append(str(current.coeff))
            elif current.exp == 1:
                terms.append(f"{current.coeff}x")
            else:
                terms.append(f"{current.coeff}x^{current.exp}")
            current = current.next
        return " + ".join(terms)


p1 = Polynomial()
for coeff, exp in [(3,2),(2,1),(1,0)]:
    p1.add_term(coeff, exp)
print(f"P1: {p1}")                    # 3x^2 + 2x + 1

p2 = Polynomial()
for coeff, exp in [(1,3),(4,1),(5,0)]:
    p2.add_term(coeff, exp)
print(f"P2: {p2}")                    # x^3 + 4x + 5

print(f"Sum: {p1 + p2}")              # x^3 + 3x^2 + 6x + 6
print(f"P1': {p1.differentiate()}")  # 6x + 2
print(f"P1(2) = {p1.evaluate(2)}")   # 17
```

---

## ⚔️ ADVANCED CHALLENGES

```
CHALLENGE 1: 
Implement a skip list using linked list concepts.
(A probabilistic data structure with O(log n) average search)

CHALLENGE 2:
Implement a deque (double-ended queue) using doubly linked list.
Support: push_front, push_back, pop_front, pop_back — all O(1).

CHALLENGE 3:
Implement LFU Cache (Least Frequently Used) using linked lists.
put() and get() in O(1).
(Harder than LRU — needs two hash maps and nested doubly linked lists)

CHALLENGE 4:
Given a linked list where each node has a 'random' pointer and a 'down' 
pointer (for multi-level lists), flatten it in level-order.

CHALLENGE 5:
Implement a circular doubly linked list with:
- Reverse (in-place)
- Rotate k positions (O(1))
- Find middle without slow/fast (explain why this is harder in circular)

CHALLENGE 6:
Sort a linked list using quicksort (compare vs merge sort performance).
When would you choose quicksort over merge sort for linked lists?

CHALLENGE 7:
Given k linked lists, each sorted, merge them using ONLY the two-pointer
merge (no heap). Analyze the time complexity and compare to heap approach.

CHALLENGE 8:
Implement a memory allocator (simplified malloc/free) using a free-block
linked list. Support: allocate(size), free(pointer), defragment().
```

---

## ✅ MASTERY CHECKLIST

```
FOUNDATIONAL (must pass before moving on):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ Write Node class and LinkedList from memory in < 2 minutes
□ Implement traversal without looking at reference
□ Implement insert at beginning, end, position — all correct
□ Implement delete by value, by index, head, tail — all correct
□ Handle ALL edge cases: empty, single node, head, tail, out-of-bounds
□ Explain O(1) insertion vs O(n) traversal correctly
□ Explain why losing head is catastrophic

PATTERN MASTERY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ Implement Floyd's cycle detection from memory
□ Explain WHY two pointers meet if cycle exists (mathematical proof)
□ Find cycle start using phase 2 — implement correctly
□ Find middle node — implement, explain even/odd list behavior
□ Reverse list iteratively — implement 4-step mantra without reference
□ Reverse list recursively — trace the call stack for 3-node list
□ Reverse sublist (left to right) — understand insert-at-front technique
□ Use dummy node naturally without thinking about it
□ Explain WHEN dummy node helps (give 3 examples)
□ Two-pointer gap technique for nth from end

INTERMEDIATE PROBLEMS (solve in < 20 mins each):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ Merge Two Sorted Lists (#21)
□ Linked List Cycle (#141)
□ Linked List Cycle II (#142)
□ Middle of Linked List (#876)
□ Reverse Linked List (#206)
□ Remove Nth Node From End (#19)
□ Remove Duplicates from Sorted List (#83)
□ Palindrome Linked List (#234)
□ Intersection of Two Linked Lists (#160)
□ Add Two Numbers (#2)

ADVANCED PROBLEMS (solve in < 30 mins each):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ Reverse Linked List II (#92) — partial reversal
□ Reverse Nodes in K Group (#25) — HARD
□ Copy List with Random Pointer (#138)
□ Rotate List (#61)
□ Reorder List (#143)
□ Flatten Multilevel DLL (#430)
□ Sort List (#148) — merge sort
□ Merge K Sorted Lists (#23) — heap approach
□ LRU Cache (#146) — DLL + HashMap

DEPTH OF UNDERSTANDING:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ Explain SLL vs DLL vs CLL trade-offs with examples
□ Implement LRU Cache from scratch — all edge cases handled
□ Explain why merge sort (not quicksort) for linked lists
□ Implement recursive and iterative reverse — explain space difference
□ Design a hash table with linked list chaining
□ Trace Floyd's meeting point math (F = C - a)
□ Identify pattern for any linked list problem in < 1 minute
□ Dry-run pointer changes step by step on paper for any problem

PROFESSIONAL READINESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ Solve Medium linked list problems in < 25 minutes
□ Solve Easy linked list problems in < 10 minutes
□ Verbally explain approach before coding — always
□ Handle edge cases without being prompted
□ State time and space complexity for every solution
□ Discuss optimization when first approach is suboptimal
□ Redo all problems without notes (true memory retention)
□ Teach a concept to someone else (highest mastery bar)
```

---

## 🗺️ RECOMMENDED NEXT LEARNING PATH

```
After mastering Linked Lists, your DSA journey continues:

IMMEDIATE NEXT (builds directly on linked lists):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. STACKS & QUEUES
   → Stack implemented with SLL (push/pop = insert/delete at head)
   → Queue implemented with SLL (enqueue at tail, dequeue at head)
   → Deque with DLL
   → Monotonic Stack problems

2. TREES (especially binary trees)
   → Trees are non-linear linked lists with two "next" pointers
   → DFS/BFS traversal uses same pointer-following intuition
   → Binary Tree linked node structure: data + left + right

3. HASH TABLES
   → Collision resolution via chaining = linked list per bucket
   → Load factor, rehashing concepts
   → Implement HashMap from scratch

MEDIUM TERM:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
4. GRAPHS
   → Adjacency list = array of linked lists
   → DFS/BFS on graphs extends tree traversal

5. DYNAMIC PROGRAMMING
   → Some DP problems on lists
   → Understanding recursion deeply (Session 9) helps here

6. ADVANCED DATA STRUCTURES
   → Skip Lists (probabilistic, layered linked lists)
   → Fibonacci Heaps (complex pointer manipulation)
   → B-Trees (used in databases)

PRACTICE RESOURCES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LeetCode: Complete Linked List study plan
  → Easy: 83, 21, 141, 206, 203, 876, 234, 160, 237
  → Medium: 2, 92, 142, 143, 19, 24, 138, 82, 61, 86, 148, 328
  → Hard: 25, 23, 146, 432

NeetCode 150: Linked list section (12 problems, curated)
Blind 75: All linked list problems
```

---

## 🎯 FINAL CHEAT SHEET: ONE-LINER PATTERN TRIGGERS

```
"detect cycle"          → Floyd's (slow×1, fast×2)
"find cycle start"      → Floyd's phase 2 (pointer from head + meeting)
"find middle"           → Fast/slow, stop when fast at end
"nth from end"          → Two pointers, gap of n, dummy node
"reverse whole list"    → prev/curr/next, 4-step mantra
"reverse partial"       → Insert-at-front, prev anchors the start
"reverse k groups"      → Check k nodes exist, reverse, recurse
"merge sorted"          → Dummy + compare-and-advance
"merge k sorted"        → Min-heap or divide-and-conquer pairs
"palindrome"            → Find middle + reverse half + compare
"reorder list"          → Find middle + reverse half + interleave merge
"add numbers"           → Simulate with carry, dummy for result
"deep copy + random"    → HashMap (O(n)) or interweave (O(1))
"rotate list"           → Make circular, find new tail, break
"flatten multilevel"    → Splice child list inline
"remove duplicates"     → Sorted: advance only on mismatch
"delete head safely"    → Dummy node
"build result list"     → Dummy node
"O(1) delete by node"   → Use DLL (has prev pointer)
"O(1) access both ends" → DLL with head + tail
"cache with eviction"   → LRU: DLL + HashMap
```

---

```
════════════════════════════════════════════════════════════════
            CONGRATULATIONS ON COMPLETING THE COURSE
════════════════════════════════════════════════════════════════

You have now covered:

Session 1:  Fundamentals — Node, head, traversal, insert, delete
Session 2:  All CRUD operations, edge cases, memory management
Session 3:  Floyd's cycle detection — detect, find start, middle
Session 4:  Full reversal, partial reversal, K-group, palindrome
Session 5:  Two-pointer gap, merge sorted, intersection, duplicates
Session 6:  Dummy node pattern — partitioning, swaps, filters
Session 7:  DLL, circular LL, LRU cache basics, music playlist
Session 8:  Deep copy, rotate, flatten, add numbers, reorder list
Session 9:  Recursive traversal, reverse, merge, swap, backtrack
Session 10: LRU cache full, merge K, sort, pattern recognition

8 Core Patterns mastered:
  ✓ Fast & Slow Pointer
  ✓ Dummy Node
  ✓ Iterative Reversal
  ✓ Two Pointer Gap
  ✓ Merge Technique
  ✓ Cycle Detection
  ✓ DLL Remove/Insert
  ✓ Recursive Processing

The REAL mastery begins now:
  → Solve 40+ problems without notes
  → Time yourself on every problem
  → Explain your solution out loud before coding
  → Teach someone else
  → Redo hard problems a week later from memory

That gap between "understood it" and "can produce it under pressure"
is crossed only through deliberate, timed, repetitive practice.

You have the knowledge. Now build the skill.
════════════════════════════════════════════════════════════════
```