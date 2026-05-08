# Complete Mastery Guide: Strings in DSA
### 20-Hour Curriculum — From Fundamentals to Interview Readiness

---

# TABLE OF CONTENTS

```
SESSION 1  — String Fundamentals & Core Operations
SESSION 2  — Frequency Maps & Hashing Patterns
SESSION 3  — Two Pointers & Palindrome Patterns
SESSION 4  — Sliding Window Fundamentals
SESSION 5  — Advanced Sliding Window
SESSION 6  — Pattern Matching Algorithms (KMP)
SESSION 7  — Rolling Hash & Rabin-Karp
SESSION 8  — Z Algorithm & Advanced String Processing
SESSION 9  — Encoding, Decoding & Parsing Problems
SESSION 10 — DP on Strings + Interview Mastery
ADDITIONAL — Easy & Advanced Extras (Trie, Manacher, Suffix Arrays)
FINAL      — Revision Sheet, Interview Q&A, Projects, Checklist
```

---

# SESSION 1 — STRING FUNDAMENTALS & CORE OPERATIONS

---

## 1.1 — String Memory Model

### Concept Explanation

A **String** in Java is an object that represents a sequence of characters stored internally as a `char[]` array. Strings in Java are stored in a special memory area called the **String Pool** (also called the intern pool), which is part of the heap memory.

**Why it exists:** The memory model was designed to allow string reuse and interning. Since strings are immutable (explained next), the JVM can safely share the same string object across multiple references.

**When to understand this:** Any time you're comparing strings, debugging unexpected behavior with `==` vs `.equals()`, or working with memory optimization.

**Real-world relevance:** In large-scale applications, understanding string memory prevents memory leaks and performance issues, especially when dealing with millions of string objects.

### Deep Understanding

```
String Pool (in Heap):
+--------------------+
| "hello"            |  <-- One object shared by s1 and s2
+--------------------+
      ^        ^
      |        |
     s1        s2   (both point to same object)

String s1 = "hello";   // goes into string pool
String s2 = "hello";   // reuses existing "hello" from pool
String s3 = new String("hello");  // creates NEW object on heap (NOT in pool)

s1 == s2   → true   (same pool reference)
s1 == s3   → false  (s3 is a new heap object)
s1.equals(s3) → true (same character content)
```

**Core terminology:**
- **String Literal:** A string defined with double quotes — goes to string pool
- **String Object:** Created with `new String()` — goes to heap (separate object)
- **Interning:** Forcing a heap string into the pool using `.intern()`

**Common misconceptions:**
- Many beginners think `==` compares string content. It compares references (memory addresses).
- `new String("hello")` creates two objects — one in the pool, one on the heap.

### Practical Usage

```java
// String literals - pool
String s1 = "hello";
String s2 = "hello";
System.out.println(s1 == s2);      // true (same reference in pool)
System.out.println(s1.equals(s2)); // true (same content)

// New keyword - heap
String s3 = new String("hello");
System.out.println(s1 == s3);      // false (different memory location)
System.out.println(s1.equals(s3)); // true (same content)

// Interning
String s4 = s3.intern();
System.out.println(s1 == s4);      // true (s4 now points to pool object)
```

---

## 1.2 — String Immutability

### Concept Explanation

**Immutability** means once a String object is created, its content cannot be changed. Any operation that "modifies" a string actually creates a new String object.

**Why it exists:**
1. **Thread safety** — Multiple threads can share strings without synchronization
2. **Security** — Passwords, file paths, network connections can't be altered mid-use
3. **Caching** — Hash code of a string is computed once and cached
4. **String Pool** — Only possible because strings can't change

**When to use it:** Always important to know — every "modification" you do creates a new string.

### Deep Understanding

```
String s = "hello";
s = s + " world";

Internally:
Step 1: "hello" (object A created in heap)
Step 2: " world" (object B created)
Step 3: "hello world" (object C created by concatenation)
Step 4: s now points to object C
Step 5: object A and B are eligible for garbage collection

"hello" itself NEVER changed.
```

**The Immutability Proof:**

```java
String s = "hello";
String ref = s;       // ref points to same "hello"
s = s.toUpperCase();  // creates new "HELLO", s now points to it
System.out.println(ref); // still prints "hello" — original unchanged
```

**Why concatenation in loops is expensive:**

```java
// BAD — O(n^2) time complexity
String result = "";
for (int i = 0; i < n; i++) {
    result += chars[i]; // creates n new string objects
}
// For n=1000, this creates 1000 intermediate string objects
// Total characters copied: 0+1+2+...+(n-1) = O(n^2)

// GOOD — O(n) time complexity
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
    sb.append(chars[i]); // modifies in place
}
String result = sb.toString(); // one final conversion
```

---

## 1.3 — String vs StringBuilder vs StringBuffer

### Concept Explanation

| Feature | String | StringBuilder | StringBuffer |
|---|---|---|---|
| Mutability | Immutable | Mutable | Mutable |
| Thread Safety | Yes | No | Yes |
| Performance | Slow for modification | Fast | Slower than SB |
| Use Case | Read-only, literals | Single-threaded modification | Multi-threaded modification |

### Deep Understanding

**StringBuilder internal mechanics:**
- Backed by a `char[]` with initial capacity 16
- When capacity exceeded, it doubles: `newCapacity = (oldCapacity * 2) + 2`
- Append is O(1) amortized (like ArrayList)

```java
StringBuilder sb = new StringBuilder();  // capacity = 16
sb.append("hello");                       // capacity still 16, length = 5
// after 16 chars, capacity expands to 34 (16*2+2)

// Important StringBuilder methods:
StringBuilder sb = new StringBuilder("hello");

sb.append(" world");       // "hello world"
sb.insert(5, ",");         // "hello, world"
sb.delete(5, 6);           // "hello world"
sb.reverse();              // "dlrow olleh"
sb.replace(0, 5, "Hi");   // "Hi olleh" (after reverse)
sb.charAt(0);              // 'H'
sb.length();               // current length
sb.toString();             // convert back to String

// Chaining
sb.append("a").append("b").append("c"); // method chaining works
```

### Practical Usage

```java
// When to use String
String name = "Alice";       // never needs modification
String key = "user_id";     // used as map key — needs stable hash

// When to use StringBuilder
public String reverseString(String s) {
    StringBuilder sb = new StringBuilder(s);
    return sb.reverse().toString();
}

// Building a result string in a loop
public String buildCSV(int[] nums) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < nums.length; i++) {
        sb.append(nums[i]);
        if (i < nums.length - 1) sb.append(",");
    }
    return sb.toString();
}

// When to use StringBuffer (rare in modern code)
// Only when truly needed in multi-threaded context
StringBuffer syncSb = new StringBuffer("thread-safe");
```

---

## 1.4 — Character Arrays

### Concept Explanation

A `char[]` is the raw underlying representation of characters. Converting between `String` and `char[]` is a common pattern in DSA problems because arrays are mutable.

### Deep Understanding

```java
// String to char array
String s = "hello";
char[] chars = s.toCharArray();  // ['h','e','l','l','o']

// Modify char array
chars[0] = 'H';   // ['H','e','l','l','o']

// Back to String
String modified = new String(chars);  // "Hello"

// Direct char access
char c = s.charAt(2);  // 'l'  — O(1)
int len = s.length();  // 5    — O(1)

// Iterating
for (int i = 0; i < s.length(); i++) {
    char c = s.charAt(i);
}

// Enhanced for loop (less common for strings, use toCharArray)
for (char c : s.toCharArray()) {
    System.out.println(c);
}
```

**When to use char array vs String:**
- Use `char[]` when you need to modify individual characters
- Use `String` when content is read-only
- Use `StringBuilder` when building/modifying iteratively

---

## 1.5 — ASCII and Unicode Basics

### Concept Explanation

**ASCII** (American Standard Code for Information Interchange) maps characters to integer values 0–127. **Unicode** extends this to handle all world languages.

**Why it exists:** Computers only understand numbers. ASCII/Unicode is the universal mapping between characters and numbers.

**Real-world relevance:** Character arithmetic (comparing, shifting, counting) is the foundation of most string algorithms.

### Deep Understanding — ASCII Table (Critical Values)

```
Character | ASCII Value
----------+-----------
'A'       |  65
'B'       |  66
...
'Z'       |  90
'a'       |  97
'b'       |  98
...
'z'       | 122
'0'       |  48
'1'       |  49
...
'9'       |  57
' '       |  32 (space)
'\n'      |  10 (newline)
'\t'      |   9 (tab)
```

**Critical ASCII Tricks for DSA:**

```java
// Index for frequency array (lowercase only)
int index = 'c' - 'a';   // = 2 (c is 3rd letter, 0-indexed)
int index = 'a' - 'a';   // = 0
int index = 'z' - 'a';   // = 25

// Lowercase to uppercase
char upper = (char)('a' - 32);  // 'a' - 32 = 65 = 'A'
char upper = (char)(c - 32);    // works for any lowercase c

// Uppercase to lowercase
char lower = (char)('A' + 32);  // 'A' + 32 = 97 = 'a'
char lower = (char)(c + 32);    // works for any uppercase c

// Better way: Java built-in
char upper = Character.toUpperCase(c);
char lower = Character.toLowerCase(c);

// Check if digit
boolean isDigit = c >= '0' && c <= '9';
boolean isDigit2 = Character.isDigit(c);

// Convert digit char to integer
int val = c - '0';   // '3' - '0' = 51 - 48 = 3

// Check if letter
boolean isLetter = Character.isLetter(c);
boolean isAlpha = (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z');

// Check if vowel
boolean isVowel = "aeiouAEIOU".indexOf(c) != -1;
```

**Unicode in Java:**

```java
// Java chars are 16-bit Unicode (UTF-16)
// Most common characters fit in one char
// Emoji and some symbols require two chars (surrogate pairs)
char c = '\u0041';  // 'A' (Unicode escape)
String emoji = "\uD83D\uDE00";  // 😀 (surrogate pair — 2 chars!)

// Safe length check for Unicode
int codePointCount = s.codePointCount(0, s.length());
```

---

## 1.6 — Time Complexity of Common String Operations

### Concept Explanation

Understanding time complexity of string operations prevents hidden O(n²) bugs in your code.

```
Operation                          | Time Complexity | Notes
-----------------------------------+-----------------+----------------------------
s.length()                         | O(1)            | Stored as field
s.charAt(i)                        | O(1)            | Direct array access
s.substring(i, j)                  | O(j-i)          | Creates new string
s1.equals(s2)                      | O(min(n,m))     | Character comparison
s1.compareTo(s2)                   | O(min(n,m))     | Lexicographic
s1 + s2                            | O(n+m)          | New object created
s.indexOf(ch)                      | O(n)            | Linear scan
s.indexOf(pattern)                 | O(n*m) naive    | KMP makes it O(n+m)
s.contains(sub)                    | O(n*m) naive    | 
s.replace(old, new)                | O(n)            | 
s.split(regex)                     | O(n)            | 
s.toCharArray()                    | O(n)            | Creates new array
new String(charArr)                | O(n)            | Creates new string
sb.append(s)                       | O(k) amortized  | k = len of appended string
sb.toString()                      | O(n)            | Creates new string
String.valueOf(int)                | O(log n)        | Digits = log10(n)
```

**Hidden O(n²) trap — substring in loop:**

```java
// This looks O(n) but is actually O(n^2)
String result = "";
for (int i = 0; i < n; i++) {
    result = result.substring(0, i) + ch; // substring + concat = O(n) each time
}
// Total: O(n) iterations × O(n) per iteration = O(n²)

// Fix: Use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
    sb.append(ch);
}
```

---

## 1.7 — String Traversal Patterns

### Concept Explanation

Traversal patterns are the building blocks for almost all string problems. Master these patterns before solving any problems.

### Pattern 1: Forward Linear Traversal

```java
// Standard left-to-right scan
public int countVowels(String s) {
    int count = 0;
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if ("aeiouAEIOU".indexOf(c) != -1) count++;
    }
    return count;
}
```

### Pattern 2: Character Frequency with Array

```java
// For lowercase only: array size 26
public int[] buildFrequency(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) {
        freq[c - 'a']++;
    }
    return freq;
}

// For all ASCII: array size 128 or 256
public int[] buildFreqASCII(String s) {
    int[] freq = new int[128];
    for (char c : s.toCharArray()) {
        freq[c]++;
    }
    return freq;
}
```

### Pattern 3: Two-Pointer Traversal

```java
// Meet in the middle
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
```

### Pattern 4: Sliding Window Traversal (Preview)

```java
// Fixed window of size k
public int maxVowelsInWindow(String s, int k) {
    int count = 0, maxCount = 0;
    for (int i = 0; i < s.length(); i++) {
        if ("aeiou".indexOf(s.charAt(i)) != -1) count++;
        if (i >= k && "aeiou".indexOf(s.charAt(i - k)) != -1) count--;
        maxCount = Math.max(maxCount, count);
    }
    return maxCount;
}
```

---

## 1.8 — Practical Exercises: Core Operations

### Exercise 1: Reverse a String

```java
// Approach 1: StringBuilder
public String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}

// Approach 2: Two pointer on char array
public String reverse2(String s) {
    char[] chars = s.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new String(chars);
}

// Time: O(n), Space: O(n) for char array
```

### Exercise 2: Toggle Case Using ASCII

```java
public String toggleCase(String s) {
    char[] chars = s.toCharArray();
    for (int i = 0; i < chars.length; i++) {
        if (chars[i] >= 'a' && chars[i] <= 'z') {
            chars[i] = (char)(chars[i] - 32);  // to upper
        } else if (chars[i] >= 'A' && chars[i] <= 'Z') {
            chars[i] = (char)(chars[i] + 32);  // to lower
        }
    }
    return new String(chars);
}
// "Hello World" → "hELLO wORLD"
```

### Exercise 3: Count Vowels and Consonants

```java
public void countVowelConsonant(String s) {
    s = s.toLowerCase();
    int vowels = 0, consonants = 0;
    for (char c : s.toCharArray()) {
        if (c >= 'a' && c <= 'z') {
            if ("aeiou".indexOf(c) != -1) vowels++;
            else consonants++;
        }
    }
    System.out.println("Vowels: " + vowels + ", Consonants: " + consonants);
}
```

### Exercise 4: Remove Spaces

```java
// Using StringBuilder
public String removeSpaces(String s) {
    StringBuilder sb = new StringBuilder();
    for (char c : s.toCharArray()) {
        if (c != ' ') sb.append(c);
    }
    return sb.toString();
}

// Using Java built-in
public String removeSpaces2(String s) {
    return s.replaceAll("\\s+", "");  // removes all whitespace
    // OR
    return s.replace(" ", "");  // removes only spaces
}
```

### Exercise 5: Check Equal Ignoring Case

```java
public boolean equalsIgnoreCase(String s1, String s2) {
    return s1.equalsIgnoreCase(s2);  // Built-in
    
    // Manual approach
    return s1.toLowerCase().equals(s2.toLowerCase());
}
```

### Exercise 6: Length of Last Word (LeetCode 58)

```java
// Naive — trim and split
public int lengthOfLastWord(String s) {
    String[] words = s.trim().split(" ");
    return words[words.length - 1].length();
}

// Optimal — traverse from end
public int lengthOfLastWordOptimal(String s) {
    int length = 0;
    int i = s.length() - 1;
    
    // Skip trailing spaces
    while (i >= 0 && s.charAt(i) == ' ') i--;
    
    // Count last word
    while (i >= 0 && s.charAt(i) != ' ') {
        length++;
        i--;
    }
    return length;
}
// "Hello World  " → 5 ("World")
// Time: O(n), Space: O(1)
```

### Exercise 7: Valid Palindrome (LeetCode 125)

```java
// Clean two-pointer solution
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        // Skip non-alphanumeric
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
        
        // Compare ignoring case
        if (Character.toLowerCase(s.charAt(left)) != 
            Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
// "A man, a plan, a canal: Panama" → true
// "race a car" → false
// Time: O(n), Space: O(1)
```

---

## Session 1 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. String is immutable — every modification creates a new object
2. == compares references, .equals() compares content — ALWAYS use .equals()
3. String + in loop is O(n²) — use StringBuilder for O(n)
4. String pool stores literals; new String() creates heap object
5. char - 'a' gives 0-indexed position for frequency arrays
6. Character arithmetic: lowercase = uppercase + 32
7. charAt(i) is O(1), substring(i,j) is O(j-i)

CRITICAL PATTERNS:
- Frequency array: int[] freq = new int[26]; freq[c - 'a']++;
- Toggle case: c ^= 32 (XOR trick) works for letters
- Two-pointer palindrome: left=0, right=n-1, move inward
- Reverse: StringBuilder.reverse() or two-pointer swap

COMPLEXITY RULES:
- charAt: O(1)
- length: O(1)  
- substring: O(k) where k = substring length
- String concatenation: O(n+m)
- StringBuilder append: O(1) amortized
```

---

# SESSION 2 — FREQUENCY MAPS & HASHING PATTERNS

---

## 2.1 — Frequency Counting

### Concept Explanation

**Frequency counting** is the process of counting how many times each character (or element) appears in a string. It is the single most common pattern in string interview problems.

**Why it exists:** Many string problems reduce to: "Do two strings have the same characters?" or "How many times does each character appear?" Frequency maps answer these questions in O(n).

**When to use:**
- Anagram detection
- Duplicate finding
- Most/least frequent character
- Character count comparison

**Real-world relevance:** Autocorrect, spell-check, search engines, text analysis tools all use frequency counting.

### Deep Understanding

**Two approaches: Array vs HashMap**

```
Array-based (PREFERRED when charset is known):
- Size 26 for lowercase only
- Size 128 for ASCII
- Size 256 for extended ASCII
- Pro: O(1) access, cache-friendly, no hashing overhead
- Con: Only works with bounded charset

HashMap-based (when charset is unknown):
- Works for any Unicode character
- Pro: Flexible, handles any character
- Con: Slightly slower due to hashing, more memory overhead
```

### Practical Usage

```java
// Template 1: Array frequency (lowercase only)
public int[] frequencyArray(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) {
        freq[c - 'a']++;     // increment count for that character
    }
    return freq;
}
// Access: freq['c' - 'a'] gives count of 'c'

// Template 2: HashMap frequency (any character)
public Map<Character, Integer> frequencyMap(String s) {
    Map<Character, Integer> map = new HashMap<>();
    for (char c : s.toCharArray()) {
        map.put(c, map.getOrDefault(c, 0) + 1);
    }
    return map;
}

// Template 3: Frequency with getOrDefault pattern
map.put(c, map.getOrDefault(c, 0) + 1);
// If c not in map → returns 0 → 0+1=1 → puts 1
// If c in map → returns current count → increments → puts back

// Template 4: merge() — cleaner approach
map.merge(c, 1, Integer::sum);
// Equivalent to getOrDefault pattern but cleaner
```

---

## 2.2 — HashMap vs Array Frequency

### Deep Comparison

```java
// SCENARIO: Count frequency of characters in "anagram"

// Array approach (26)
int[] freq = new int[26];
for (char c : "anagram".toCharArray()) freq[c - 'a']++;
// freq[0] = 3  (a appears 3 times, 'a'-'a'=0)
// freq[6] = 1  (g appears 1 time,  'g'-'a'=6)
// freq[13] = 1 (n appears 1 time,  'n'-'a'=13)
// freq[17] = 1 (r appears 1 time,  'r'-'a'=17)
// freq[12] = 1 (m appears 1 time,  'm'-'a'=12)

// Access in O(1): freq['a'-'a'] → 3
// Compare two frequency arrays in O(26) = O(1)

// HashMap approach
Map<Character, Integer> map = new HashMap<>();
for (char c : "anagram".toCharArray())
    map.put(c, map.getOrDefault(c, 0) + 1);
// {a=3, n=1, g=1, r=1, m=1}

// Access: map.get('a') → 3
// Compare two maps: map1.equals(map2)

// RULE OF THUMB:
// If problem guarantees lowercase only → use int[26]
// If problem has any character → use HashMap
// Performance: int[26] is ~5x faster than HashMap for this case
```

---

## 2.3 — Set-Based Duplicate Detection

```java
// Use HashSet to detect duplicates in O(n)
public boolean hasDuplicate(String s) {
    Set<Character> seen = new HashSet<>();
    for (char c : s.toCharArray()) {
        if (!seen.add(c)) return true;  // add returns false if already present
    }
    return false;
}

// All unique characters check
public boolean allUnique(String s) {
    return s.chars().distinct().count() == s.length();
    // Or with set:
    Set<Character> set = new HashSet<>();
    for (char c : s.toCharArray()) set.add(c);
    return set.size() == s.length();
}
```

---

## 2.4 — Problem: Valid Anagram (LeetCode 242)

### Concept Explanation

Two strings are anagrams if they contain the same characters with the same frequencies.
"anagram" and "nagaram" → anagram ✓
"rat" and "car" → not anagram ✗

### Solutions

```java
// Approach 1: Sorting — O(n log n) time, O(n) space
public boolean isAnagramSort(String s, String t) {
    if (s.length() != t.length()) return false;
    char[] sc = s.toCharArray();
    char[] tc = t.toCharArray();
    Arrays.sort(sc);
    Arrays.sort(tc);
    return Arrays.equals(sc, tc);
}

// Approach 2: Frequency array — O(n) time, O(1) space (best)
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    
    int[] freq = new int[26];
    
    // Increment for s, decrement for t
    for (int i = 0; i < s.length(); i++) {
        freq[s.charAt(i) - 'a']++;
        freq[t.charAt(i) - 'a']--;
    }
    
    // If anagram, all counts should be 0
    for (int count : freq) {
        if (count != 0) return false;
    }
    return true;
}

// Approach 3: HashMap (for Unicode characters)
public boolean isAnagramUnicode(String s, String t) {
    if (s.length() != t.length()) return false;
    
    Map<Character, Integer> map = new HashMap<>();
    for (char c : s.toCharArray()) map.merge(c, 1, Integer::sum);
    for (char c : t.toCharArray()) {
        map.merge(c, -1, Integer::sum);
        if (map.get(c) < 0) return false;  // early termination
    }
    return true;
}
```

---

## 2.5 — Problem: Group Anagrams (LeetCode 49)

### Concept Explanation

Given an array of strings, group all anagrams together.
Input: ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]

### Deep Understanding

The key insight: Two strings are anagrams if and only if their sorted versions are equal. OR if their frequency arrays are equal.

Use the sorted string (or encoded frequency) as a **HashMap key**.

```java
// Approach 1: Sorted string as key — O(n * k log k)
// n = number of strings, k = max string length
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);                         // sort characters
        String key = new String(chars);             // sorted string as key
        
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
        // computeIfAbsent: if key not present, create new list; then add s
    }
    
    return new ArrayList<>(map.values());
}

// Approach 2: Frequency array encoded as key — O(n * k) — optimal
public List<List<String>> groupAnagramsOptimal(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    
    for (String s : strs) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;
        
        // Encode frequency array as unique string key
        // "#3#0#1#..." for each of 26 letters
        StringBuilder keyBuilder = new StringBuilder();
        for (int count : freq) {
            keyBuilder.append('#');
            keyBuilder.append(count);
        }
        String key = keyBuilder.toString();
        
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    
    return new ArrayList<>(map.values());
}
```

**Why encode with '#'?**  
Without delimiter, "1","10","100"... could collide.  
`#1#0` vs `#10` are distinct keys when using '#' as separator.

---

## 2.6 — Problem: Ransom Note (LeetCode 383)

```java
// Can ransomNote be constructed from magazine?
// Each letter in magazine can only be used once.

public boolean canConstruct(String ransomNote, String magazine) {
    int[] freq = new int[26];
    
    // Count magazine characters
    for (char c : magazine.toCharArray()) freq[c - 'a']++;
    
    // Check if ransomNote can use magazine's characters
    for (char c : ransomNote.toCharArray()) {
        freq[c - 'a']--;
        if (freq[c - 'a'] < 0) return false;  // not enough of this char
    }
    return true;
}
// Time: O(n+m), Space: O(1) — fixed 26 chars
```

---

## 2.7 — Problem: First Non-Repeating Character

```java
// LeetCode 387: First Unique Character in a String
public int firstUniqChar(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    
    for (int i = 0; i < s.length(); i++) {
        if (freq[s.charAt(i) - 'a'] == 1) return i;
    }
    return -1;  // no unique character
}
// "leetcode" → 0 (l is first non-repeating)
// "aabb" → -1

// Two-pass approach:
// Pass 1: Build frequency
// Pass 2: Find first char with frequency 1
```

---

## 2.8 — Problem: Find Common Characters (LeetCode 1002)

```java
// Find characters that appear in ALL strings (with repetition)
public List<String> commonChars(String[] words) {
    int[] minFreq = new int[26];
    Arrays.fill(minFreq, Integer.MAX_VALUE);  // start with "infinite" count
    
    for (String word : words) {
        int[] freq = new int[26];
        for (char c : word.toCharArray()) freq[c - 'a']++;
        
        // Take minimum frequency for each character across all words
        for (int i = 0; i < 26; i++) {
            minFreq[i] = Math.min(minFreq[i], freq[i]);
        }
    }
    
    List<String> result = new ArrayList<>();
    for (int i = 0; i < 26; i++) {
        for (int j = 0; j < minFreq[i]; j++) {
            result.add(String.valueOf((char)('a' + i)));
        }
    }
    return result;
}
// Input: ["bella","label","roller"]
// Output: ["e","l","l"]
```

---

## 2.9 — Most Frequent Character Pattern

```java
// Find character with highest frequency
public char mostFrequent(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    
    int maxFreq = 0, maxIdx = 0;
    for (int i = 0; i < 26; i++) {
        if (freq[i] > maxFreq) {
            maxFreq = freq[i];
            maxIdx = i;
        }
    }
    return (char)('a' + maxIdx);
}

// Sort characters by frequency
public String sortByFrequency(String s) {
    Map<Character, Integer> map = new HashMap<>();
    for (char c : s.toCharArray()) map.merge(c, 1, Integer::sum);
    
    List<Character> chars = new ArrayList<>(map.keySet());
    chars.sort((a, b) -> map.get(b) - map.get(a));  // sort descending by freq
    
    StringBuilder sb = new StringBuilder();
    for (char c : chars) {
        for (int i = 0; i < map.get(c); i++) sb.append(c);
    }
    return sb.toString();
}
```

---

## Session 2 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. Frequency array (int[26]) is faster than HashMap for lowercase strings
2. getOrDefault(key, 0) is the standard HashMap frequency pattern
3. For anagram grouping: use sorted string OR encoded frequency as key
4. computeIfAbsent() is the cleanest way to build lists in maps
5. To compare two frequency arrays: Arrays.equals(freq1, freq2) in O(26)=O(1)

CRITICAL PATTERNS:
// Array freq
int[] freq = new int[26];
freq[c - 'a']++;

// HashMap freq  
map.put(c, map.getOrDefault(c, 0) + 1);
// OR
map.merge(c, 1, Integer::sum);

// Anagram key (sorted)
String key = new String(Arrays.sort(s.toCharArray()));

// Anagram key (frequency encoded)
// Encode int[26] → "#c1#c2#...#c26"

COMMON MISTAKES:
- Forgetting to check s.length() == t.length() before anagram check
- Using HashMap when array[26] is sufficient (5x slower)
- Not encoding frequency key with delimiter (causes collisions)
- Missing early termination optimization
```

---

# SESSION 3 — TWO POINTERS & PALINDROME PATTERNS

---

## 3.1 — Two-Pointer Technique

### Concept Explanation

**Two pointers** is a technique where you maintain two index variables pointing to different positions in the string, moving them based on conditions. It converts many O(n²) brute-force solutions into O(n).

**Why it exists:** Many string problems involve comparing or processing characters from both ends, or finding pairs/windows. Two pointers encode this naturally.

**When to use:**
- Palindrome checking
- Finding pairs that satisfy a condition
- Removing duplicates from sorted arrays
- Merging sorted arrays
- Matching characters from two strings

### Three Two-Pointer Variants

```
Variant 1: Opposite ends — starts at both ends, moves inward
Variant 2: Same direction — both start at left, one is "fast" one is "slow"
Variant 3: Two strings — one pointer per string, both move forward
```

```java
// Variant 1: Opposite ends (palindrome)
int left = 0, right = n - 1;
while (left < right) {
    // process s[left] and s[right]
    left++;
    right--;
}

// Variant 2: Same direction (remove duplicates)
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (condition) {
        // write s[fast] to slow position
        slow++;
    }
}

// Variant 3: Two strings (compare characters)
int i = 0, j = 0;
while (i < s1.length() && j < s2.length()) {
    if (s1.charAt(i) == s2.charAt(j)) { i++; j++; }
    else { /* handle mismatch */ }
}
```

---

## 3.2 — Palindrome Properties

### Concept Explanation

A **palindrome** reads the same forwards and backwards.
- "racecar" — palindrome
- "abba" — palindrome  
- "aba" — palindrome
- "abc" — not palindrome

**Key properties:**
- Characters at position `i` and `n-1-i` must be equal
- For odd-length palindromes, the middle character can be anything
- For even-length palindromes, all characters appear in pairs
- Every single character is a palindrome
- Empty string is a palindrome

### The Expand-Around-Center Technique

This is the most powerful palindrome technique. Instead of checking a fixed string, you **start from a center and expand outward** as long as the palindrome condition holds.

```
For string "abacaba":

Odd-length centers: each character (7 centers)
Even-length centers: between each adjacent pair (6 centers)

Center at index 3 (character 'c'):
  left=3, right=3 → "c" (palindrome)
  left=2, right=4 → "acaba"? a==a ✓, expand...
  left=1, right=5 → "bacab"? b==b ✓, expand...
  left=0, right=6 → "abacaba"? a==a ✓ — longest palindrome!
```

```java
// Core expand-around-center function
private int[] expandCenter(String s, int left, int right) {
    // Expand while palindrome condition holds
    while (left >= 0 && right < s.length() 
           && s.charAt(left) == s.charAt(right)) {
        left--;
        right++;
    }
    // When we exit, left and right are OUTSIDE the palindrome
    // Last valid palindrome: s[left+1 ... right-1]
    return new int[]{left + 1, right - 1};
}

// Call for odd-length center (single character center)
int[] result = expandCenter(s, i, i);

// Call for even-length center (between i and i+1)
int[] result = expandCenter(s, i, i + 1);
```

---

## 3.3 — Problem: Longest Palindromic Substring (LeetCode 5)

### Concept Explanation

Find the longest substring of a string that is a palindrome.
"babad" → "bab" or "aba"
"cbbd" → "bb"

### Solutions

```java
// Approach 1: Brute Force — O(n^3)
// Check every substring if it's palindrome
// Too slow for interviews

// Approach 2: Expand Around Center — O(n^2) time, O(1) space (optimal for interviews)
public String longestPalindrome(String s) {
    if (s == null || s.length() < 1) return "";
    
    int start = 0, maxLen = 1;
    
    for (int i = 0; i < s.length(); i++) {
        // Check odd-length palindromes (center at i)
        int[] odd = expandCenter(s, i, i);
        int oddLen = odd[1] - odd[0] + 1;
        if (oddLen > maxLen) {
            maxLen = oddLen;
            start = odd[0];
        }
        
        // Check even-length palindromes (center between i and i+1)
        if (i + 1 < s.length()) {
            int[] even = expandCenter(s, i, i + 1);
            int evenLen = even[1] - even[0] + 1;
            if (evenLen > maxLen) {
                maxLen = evenLen;
                start = even[0];
            }
        }
    }
    
    return s.substring(start, start + maxLen);
}

private int[] expandCenter(String s, int left, int right) {
    while (left >= 0 && right < s.length() 
           && s.charAt(left) == s.charAt(right)) {
        left--;
        right++;
    }
    return new int[]{left + 1, right - 1};
}

// "cbbd":
// i=0: odd center 'c' → "c", even 'c'/'b' → not palindrome
// i=1: odd center 'b' → "b", even 'b'/'b' → "bb" ← longest (len=2)
// i=2: odd center 'b' → "b", even 'b'/'d' → not palindrome
// i=3: odd center 'd' → "d"
// Result: "bb"

// Approach 3: Dynamic Programming — O(n^2) time, O(n^2) space
// dp[i][j] = true if s[i..j] is palindrome
public String longestPalindromeDP(String s) {
    int n = s.length();
    boolean[][] dp = new boolean[n][n];
    int start = 0, maxLen = 1;
    
    // Every single character is palindrome
    for (int i = 0; i < n; i++) dp[i][i] = true;
    
    // Check length 2
    for (int i = 0; i < n - 1; i++) {
        if (s.charAt(i) == s.charAt(i + 1)) {
            dp[i][i + 1] = true;
            start = i;
            maxLen = 2;
        }
    }
    
    // Check lengths 3 to n
    for (int len = 3; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            // s[i..j] is palindrome if s[i+1..j-1] is palindrome AND s[i]==s[j]
            if (s.charAt(i) == s.charAt(j) && dp[i + 1][j - 1]) {
                dp[i][j] = true;
                if (len > maxLen) {
                    maxLen = len;
                    start = i;
                }
            }
        }
    }
    
    return s.substring(start, start + maxLen);
}
```

**Note on Manacher's Algorithm:** Exists for O(n) solution but rarely asked in interviews. Covered in Additional Topics.

---

## 3.4 — Problem: Valid Palindrome II (LeetCode 680)

### Concept Explanation

Given a string, can you make it a palindrome by removing **at most one** character?

"aba" → true (already palindrome)
"abca" → true (remove 'c')
"abc" → false

### Deep Understanding

Standard palindrome check with two pointers. When mismatch found, try skipping either the left or right character and check if remaining is palindrome.

```java
public boolean validPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) {
            // Try removing left or removing right
            return isPalindromeRange(s, left + 1, right) 
                || isPalindromeRange(s, left, right - 1);
        }
        left++;
        right--;
    }
    return true;
}

private boolean isPalindromeRange(String s, int left, int right) {
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}

// "abca":
// left=0('a'), right=3('a') → match, left=1, right=2
// left=1('b'), right=2('c') → mismatch!
// Try removing 'b': isPalindrome("ca", 2, 2) → "c" == palindrome? yes, single char
//   left=2, right=2 → left >= right → return true ✓
// Result: true
```

---

## 3.5 — Problem: Palindromic Substrings (LeetCode 647)

### Concept Explanation

Count all substrings that are palindromes.
"abc" → "a","b","c" → 3
"aaa" → "a","a","a","aa","aa","aaa" → 6

```java
public int countSubstrings(String s) {
    int count = 0;
    
    for (int i = 0; i < s.length(); i++) {
        count += countFromCenter(s, i, i);       // odd-length
        count += countFromCenter(s, i, i + 1);   // even-length
    }
    return count;
}

private int countFromCenter(String s, int left, int right) {
    int count = 0;
    while (left >= 0 && right < s.length() 
           && s.charAt(left) == s.charAt(right)) {
        count++;
        left--;
        right++;
    }
    return count;
}

// "aaa":
// i=0: odd from (0,0) → "a"(count=1), "aa"(count=2, going -1,1 fails at left=-1)... 
//      Actually let me trace:
//      center (0,0): left=0,right=0 → 'a'=='a' ✓, count=1, left=-1,right=1 → exit
//      center (0,1): left=0,right=1 → 'a'=='a' ✓, count=1, left=-1,right=2 → exit
// i=1: odd (1,1): left=1,right=1→'a'==a ✓ count=1; left=0,right=2→'a'=='a' ✓ count=2; left=-1→exit
//      even (1,2): left=1,right=2→'a'=='a' ✓ count=1; left=0,right=3→exit (out of bounds)
// i=2: odd (2,2): count=1; even (2,3): out of bounds from start
// Total: 1+1+2+1+1 = 6 ✓
```

---

## 3.6 — Professional Insights for Palindromes

```
INTERVIEW TIPS:
1. Always handle both odd and even length centers
2. The expand function should exit when chars DON'T match
3. After expanding, the palindrome is s[left+1 ... right-1], not s[left...right]
4. For "can we make palindrome with k deletions" → check if len - longestPalindrome <= k

COMMON MISTAKES:
1. Using only odd-center expansion (misses "bb", "abba", etc.)
2. Off-by-one in the returned substring indices
3. Forgetting to check i+1 < s.length() before even expansion
4. In Valid Palindrome II, not trying BOTH options when mismatch found

PATTERN RECOGNITION:
- "Is this a palindrome?" → Two pointers from ends
- "Longest palindromic substring?" → Expand around center
- "Count palindromic substrings?" → Expand around center, sum counts
- "Can become palindrome with k changes?" → LPS + DP
```

---

## Session 3 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. Two pointers: left=0, right=n-1, move inward while left < right
2. Expand-around-center handles BOTH odd and even length palindromes
3. After expand(left, right): palindrome is s[left+1 .. right-1]
4. Valid Palindrome II: on mismatch, try BOTH left+1 and right-1

CRITICAL CODE:
// Expand around center
while (l >= 0 && r < n && s[l] == s[r]) { l--; r++; }
// Palindrome is s[l+1 .. r-1]

// Two pointer palindrome check
int l = 0, r = n - 1;
while (l < r) {
    if (s[l] != s[r]) return false;
    l++; r--;
}
return true;
```

---

# SESSION 4 — SLIDING WINDOW FUNDAMENTALS

---

## 4.1 — Sliding Window Concept

### Concept Explanation

The **sliding window** technique maintains a "window" (contiguous subarray/substring) that slides over the input. Instead of recomputing the entire window each time, we add the new right element and remove the old left element.

**Why it exists:** Many problems ask for optimal contiguous subarrays/substrings. Brute force is O(n²) or O(n³). Sliding window reduces to O(n).

**When to use:**
- "Find the longest/shortest substring with property X"
- "Find all substrings of length k with property X"
- "Find minimum window containing all elements of another set"

**Two types:**
1. **Fixed-size window** — window size k is constant
2. **Variable-size window** — window expands and shrinks based on validity

---

## 4.2 — Fixed-Size Sliding Window

### Concept Explanation

Window size stays constant at k. Remove element leaving from left, add element entering from right.

### Template

```java
// Fixed window template
public int fixedWindow(String s, int k) {
    // Initialize window with first k characters
    int windowValue = 0;
    for (int i = 0; i < k; i++) {
        windowValue += compute(s.charAt(i));  // e.g., count vowels
    }
    
    int maxValue = windowValue;
    
    // Slide the window
    for (int i = k; i < s.length(); i++) {
        // Add new right element
        windowValue += compute(s.charAt(i));
        // Remove old left element (i-k is leaving the window)
        windowValue -= compute(s.charAt(i - k));
        // Update answer
        maxValue = Math.max(maxValue, windowValue);
    }
    
    return maxValue;
}
```

### Example: Maximum Vowels in Substring of Length K

```java
public int maxVowels(String s, int k) {
    // Helper: is this a vowel?
    // Count vowels in first window
    int vowelCount = 0;
    for (int i = 0; i < k; i++) {
        if (isVowel(s.charAt(i))) vowelCount++;
    }
    
    int maxVowels = vowelCount;
    
    // Slide
    for (int i = k; i < s.length(); i++) {
        if (isVowel(s.charAt(i))) vowelCount++;      // entering char
        if (isVowel(s.charAt(i - k))) vowelCount--;  // leaving char
        maxVowels = Math.max(maxVowels, vowelCount);
    }
    
    return maxVowels;
}

private boolean isVowel(char c) {
    return "aeiou".indexOf(c) != -1;
}
// "abciiidef", k=3 → "iii" → 3
// Time: O(n), Space: O(1)
```

---

## 4.3 — Variable-Size Sliding Window

### Concept Explanation

Window size changes dynamically. Expand right pointer to include more elements. Shrink left pointer when window becomes invalid.

### Universal Template

```java
// Variable window template
public int variableWindow(String s) {
    int left = 0;
    int windowState = 0;     // e.g., count of distinct chars, frequency tracker
    int result = 0;
    
    for (int right = 0; right < s.length(); right++) {
        // 1. EXPAND: Add s[right] to window state
        windowState = update(windowState, s.charAt(right), +1);
        
        // 2. SHRINK: While window is INVALID, shrink from left
        while (windowIsInvalid(windowState)) {
            windowState = update(windowState, s.charAt(left), -1);
            left++;
        }
        
        // 3. UPDATE ANSWER: Window is now valid
        result = Math.max(result, right - left + 1);
        // (or += 1, or whatever the problem asks)
    }
    
    return result;
}
```

**Mental model:** 
- Right pointer always advances one step
- Left pointer advances only when window becomes invalid
- Window [left, right] is always valid when we compute the answer

---

## 4.4 — Problem: Longest Substring Without Repeating Characters (LeetCode 3)

### Concept Explanation

Find the length of the longest substring with all unique characters.
"abcabcbb" → 3 ("abc")
"bbbbb" → 1 ("b")
"pwwkew" → 3 ("wke")

### Solutions

```java
// Approach 1: HashSet sliding window — O(n) time, O(min(n,m)) space
public int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        
        // Shrink from left until c is no longer in window
        while (window.contains(c)) {
            window.remove(s.charAt(left));
            left++;
        }
        
        window.add(c);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    
    return maxLen;
}

// Approach 2: HashMap with last-seen index — O(n) optimal
// Instead of shrinking one by one, JUMP left directly to position after duplicate
public int lengthOfLongestSubstringOptimal(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        
        // If c was seen before AND its last position is inside current window
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1;  // jump left past the duplicate
        }
        
        lastSeen.put(c, right);  // update last seen position
        maxLen = Math.max(maxLen, right - left + 1);
    }
    
    return maxLen;
}

// Approach 3: Array instead of HashMap for ASCII strings — fastest
public int lengthOfLongestSubstringArray(String s) {
    int[] lastSeen = new int[128];  // ASCII
    Arrays.fill(lastSeen, -1);       // -1 means not seen
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        
        if (lastSeen[c] >= left) {    // was seen inside current window
            left = lastSeen[c] + 1;
        }
        
        lastSeen[c] = right;
        maxLen = Math.max(maxLen, right - left + 1);
    }
    
    return maxLen;
}

// Trace "abcabcbb":
// right=0(a): lastSeen[a]=-1, left=0, window="a", len=1
// right=1(b): lastSeen[b]=-1, left=0, window="ab", len=2
// right=2(c): lastSeen[c]=-1, left=0, window="abc", len=3
// right=3(a): lastSeen[a]=0 >= left=0, left=0+1=1, window="bca", len=3
// right=4(b): lastSeen[b]=1 >= left=1, left=1+1=2, window="cab", len=3
// right=5(c): lastSeen[c]=2 >= left=2, left=2+1=3, window="abc", len=3
// right=6(b): lastSeen[b]=4 >= left=3, left=4+1=5, window="cb", len=2
// right=7(b): lastSeen[b]=6 >= left=5, left=6+1=7, window="b", len=1
// Result: 3 ✓
```

---

## 4.5 — Problem: Permutation in String (LeetCode 567)

### Concept Explanation

Given strings s1 and s2, return true if any permutation of s1 exists as a substring of s2.
s1="ab", s2="eidbaooo" → true ("ba" is a permutation of "ab")
s1="ab", s2="eidboaoo" → false

### Deep Understanding

A permutation of s1 is a fixed-size window of length s1.length() in s2 that has the same character frequencies as s1.

```java
public boolean checkInclusion(String s1, String s2) {
    if (s1.length() > s2.length()) return false;
    
    int[] need = new int[26];   // frequency of s1
    int[] have = new int[26];   // frequency of current window in s2
    
    for (char c : s1.toCharArray()) need[c - 'a']++;
    
    int k = s1.length();  // fixed window size
    
    // Initialize first window
    for (int i = 0; i < k; i++) have[s2.charAt(i) - 'a']++;
    
    if (Arrays.equals(need, have)) return true;
    
    // Slide the window
    for (int i = k; i < s2.length(); i++) {
        // Add new right character
        have[s2.charAt(i) - 'a']++;
        // Remove old left character
        have[s2.charAt(i - k) - 'a']--;
        
        if (Arrays.equals(need, have)) return true;
    }
    
    return false;
}
// Time: O(n) since Arrays.equals on size-26 array is O(26) = O(1)
// Space: O(1)

// OPTIMIZED: Track "matches" count instead of comparing arrays each time
public boolean checkInclusionOptimized(String s1, String s2) {
    if (s1.length() > s2.length()) return false;
    
    int[] count = new int[26];
    for (char c : s1.toCharArray()) count[c - 'a']++;
    
    // 'matches' = number of characters with correct frequency
    int matches = 0;
    for (int c : count) if (c == 0) matches++;  // chars not in s1 = 26 - distinct(s1)
    // Wait, cleaner: count how many of 26 buckets are "balanced"
    
    // Actually cleaner version using difference array:
    int[] diff = new int[26];
    for (char c : s1.toCharArray()) diff[c - 'a']++;
    
    int required = 0;  // number of characters that still need to be balanced
    for (int d : diff) if (d != 0) required++;
    
    int left = 0;
    for (int right = 0; right < s2.length(); right++) {
        // Add s2[right] to window
        int rc = s2.charAt(right) - 'a';
        diff[rc]--;
        if (diff[rc] == 0) required--;       // this character is now balanced
        else if (diff[rc] == -1) required++; // this character went over
        
        if (right - left + 1 > s1.length()) {
            // Remove s2[left] from window
            int lc = s2.charAt(left) - 'a';
            diff[lc]++;
            if (diff[lc] == 0) required--;
            else if (diff[lc] == 1) required++;
            left++;
        }
        
        if (required == 0) return true;
    }
    
    return false;
}
```

---

## 4.6 — Problem: Find All Anagrams in a String (LeetCode 438)

```java
// Find all starting indices of p's anagrams in s
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    if (s.length() < p.length()) return result;
    
    int[] pFreq = new int[26];
    int[] windowFreq = new int[26];
    
    for (char c : p.toCharArray()) pFreq[c - 'a']++;
    
    // Initialize first window
    for (int i = 0; i < p.length(); i++) windowFreq[s.charAt(i) - 'a']++;
    
    if (Arrays.equals(pFreq, windowFreq)) result.add(0);
    
    // Slide
    for (int i = p.length(); i < s.length(); i++) {
        windowFreq[s.charAt(i) - 'a']++;                    // add right
        windowFreq[s.charAt(i - p.length()) - 'a']--;       // remove left
        
        if (Arrays.equals(pFreq, windowFreq)) {
            result.add(i - p.length() + 1);                  // start index
        }
    }
    
    return result;
}
// s="cbaebabacd", p="abc" → [0, 6]
// Window [0,2]="cba" is anagram of "abc" → start=0
// Window [6,8]="bac" is anagram of "abc" → start=6
```

---

## Session 4 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. Fixed window: add right element, remove element at right-k each iteration
2. Variable window: expand right always, shrink left when invalid
3. Window [left, right] has length = right - left + 1
4. For anagram problems: compare frequency arrays of pattern vs window
5. Arrays.equals on int[26] is effectively O(1) for window validity check

CRITICAL TEMPLATES:
// Fixed window
for i from k to n:
    add s[i], remove s[i-k], update answer

// Variable window  
for right = 0 to n-1:
    add s[right] to window
    while window invalid:
        remove s[left], left++
    update answer

COMMON MISTAKES:
1. Forgetting to remove the outgoing left element
2. Off-by-one: window size = right - left + 1, not right - left
3. Infinite loops: always ensure left advances in the while loop
4. Using HashMap when int[26] suffices
```

---

# SESSION 5 — ADVANCED SLIDING WINDOW

---

## 5.1 — Dynamic Constraint Windows

### Concept Explanation

More complex sliding window problems have **dynamic validity conditions** — the window can be valid under multiple states, and you need to track what makes it valid or invalid in real time.

**Key insight:** Track the number of "satisfied" conditions or a "formed" counter to avoid re-checking the entire window each time.

---

## 5.2 — Problem: Minimum Window Substring (LeetCode 76)

### Concept Explanation

Given strings s and t, find the minimum window in s that contains all characters of t.
s="ADOBECODEBANC", t="ABC" → "BANC"
s="a", t="a" → "a"
s="a", t="aa" → ""

This is the **hardest** of the sliding window problems and a favorite in FAANG interviews.

### Deep Understanding

```
Key variables:
- need[c]: how many times character c is needed (from t)
- have[c]: how many times character c is present in current window
- formed: number of DISTINCT characters whose count is satisfied (have[c] >= need[c])
- required: total number of distinct characters in t that we need to satisfy
- When formed == required: window is valid (contains all of t)
```

```java
public String minWindow(String s, String t) {
    if (s.isEmpty() || t.isEmpty()) return "";
    
    // Build frequency map for t
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
    
    int required = need.size();  // distinct chars needed
    int formed = 0;              // distinct chars whose count is satisfied
    
    Map<Character, Integer> have = new HashMap<>();
    
    int left = 0;
    int minLen = Integer.MAX_VALUE;
    int minLeft = 0;
    
    for (int right = 0; right < s.length(); right++) {
        // Add right character to window
        char c = s.charAt(right);
        have.merge(c, 1, Integer::sum);
        
        // Check if this character's requirement is now satisfied
        if (need.containsKey(c) && have.get(c).intValue() == need.get(c).intValue()) {
            formed++;
        }
        
        // Try to shrink window while it remains valid
        while (formed == required && left <= right) {
            // Update minimum window
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minLeft = left;
            }
            
            // Remove left character
            char lc = s.charAt(left);
            have.merge(lc, -1, Integer::sum);
            
            // Check if removing this breaks the formed count
            if (need.containsKey(lc) && have.get(lc) < need.get(lc)) {
                formed--;
            }
            left++;
        }
    }
    
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
}

// Trace: s="ADOBECODEBANC", t="ABC"
// need={A:1, B:1, C:1}, required=3

// right=0 A: have={A:1}, A satisfied, formed=1
// right=1 D: have={A:1,D:1}, formed=1
// right=2 O: have={A:1,D:1,O:1}, formed=1
// right=3 B: have={...B:1}, B satisfied, formed=2
// right=4 E: formed=2
// right=5 C: have={...C:1}, C satisfied, formed=3
//   formed==3! window="ADOBEC"(len=6), minLen=6
//   shrink: remove A(left=0), have[A]=0 < need[A]=1, formed=2, left=1
// right=6 O: formed=2
// right=7 D: formed=2
// right=8 E: formed=2
// right=9 B: have[B]=2, B was already satisfied, formed still 2
// right=10 A: have[A]=1, A satisfied, formed=3
//   window="DOBECODEBA"... shrink:
//   remove D(left=1): D not in need, keep shrinking
//   remove O: keep, left=3
//   remove B: have[B]=1 still >= need[B]=1, keep shrinking
//   remove E: keep, left=5
//   remove C: have[C]=0 < 1, formed=2, left=6
//   Can't shrink more. Window was "CODEBA" → no, let me recheck start.
//   minLen gets updated at each valid shrink step.
// ... eventually result = "BANC" ✓

// Time: O(|s| + |t|), Space: O(|s| + |t|)
```

### Optimized Version with Array

```java
// When dealing only with letters, use int[128] instead of HashMap
public String minWindowArray(String s, String t) {
    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;
    
    int required = 0;
    for (int n : need) if (n > 0) required++;
    
    int[] have = new int[128];
    int formed = 0, left = 0;
    int minLen = Integer.MAX_VALUE, minLeft = 0;
    
    for (int right = 0; right < s.length(); right++) {
        int c = s.charAt(right);
        have[c]++;
        if (need[c] > 0 && have[c] == need[c]) formed++;
        
        while (formed == required) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minLeft = left;
            }
            int lc = s.charAt(left);
            have[lc]--;
            if (need[lc] > 0 && have[lc] < need[lc]) formed--;
            left++;
        }
    }
    
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
}
```

---

## 5.3 — Problem: Longest Repeating Character Replacement (LeetCode 424)

### Concept Explanation

You can replace at most k characters. Find the longest substring where all characters are the same after at most k replacements.
"AABABBA", k=1 → 4 ("AABA" replace B → A, or "BABB" replace A → B)

### Deep Understanding

**Key insight:** In any window [left, right]:
- Window length = right - left + 1
- Most frequent character count = maxFreq
- Characters that need replacing = windowLen - maxFreq
- Window is valid if: windowLen - maxFreq <= k

```java
public int characterReplacement(String s, int k) {
    int[] freq = new int[26];
    int left = 0;
    int maxFreq = 0;   // highest frequency in current window
    int maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        // Add right character
        freq[s.charAt(right) - 'A']++;
        maxFreq = Math.max(maxFreq, freq[s.charAt(right) - 'A']);
        
        // Window validity: windowLen - maxFreq <= k
        int windowLen = right - left + 1;
        if (windowLen - maxFreq > k) {
            // Shrink from left (by exactly 1, since we only want to find max)
            freq[s.charAt(left) - 'A']--;
            left++;
            // Note: maxFreq might not reflect true max after shrinking
            // But this is intentional — we only want windows >= current maxLen
        }
        
        maxLen = Math.max(maxLen, right - left + 1);
    }
    
    return maxLen;
}

// "AABABBA", k=1:
// right=0 A: freq[A]=1, maxFreq=1, window="A"(1), 1-1=0<=1 ✓, maxLen=1
// right=1 A: freq[A]=2, maxFreq=2, window="AA"(2), 2-2=0<=1 ✓, maxLen=2
// right=2 B: freq[B]=1, maxFreq=2, window="AAB"(3), 3-2=1<=1 ✓, maxLen=3
// right=3 A: freq[A]=3, maxFreq=3, window="AABA"(4), 4-3=1<=1 ✓, maxLen=4
// right=4 B: freq[B]=2, maxFreq=3, window="AABAB"(5), 5-3=2>1 ✗
//   shrink: freq[A]--, left=1, maxFreq stays 3
//   window="ABAB"(4), 4-3=1<=1, but we already shrunk to maintain size
// right=5 B: freq[B]=3, maxFreq=3, window="ABABB"(5), but left=1
//   windowLen=5, 5-3=2>1 ✗, shrink: freq[A]--, left=2, maxFreq stays 3
//   window="BABB"(4)
// right=6 A: freq[A]=1, maxFreq=3, window="BABBA"... 
// maxLen = 4 ✓

// Time: O(n), Space: O(26)=O(1)
```

**Why maxFreq doesn't need to decrease when shrinking:**
We're not looking for a shorter valid window — we only care about the maximum length found. If maxFreq drops, window might become invalid but it can't exceed what we already found, so we just maintain the same size window until we find a better one.

---

## 5.4 — Problem: Minimum Size Subarray Sum (LeetCode 209)

```java
// Find minimum length subarray with sum >= target
public int minSubArrayLen(int target, int[] nums) {
    int left = 0, sum = 0;
    int minLen = Integer.MAX_VALUE;
    
    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];
        
        // Shrink while sum >= target (try to minimize length)
        while (sum >= target) {
            minLen = Math.min(minLen, right - left + 1);
            sum -= nums[left++];
        }
    }
    
    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
// nums=[2,3,1,2,4,3], target=7 → 2 ([4,3])
```

---

## 5.5 — Advanced: Smallest Substring Containing All Characters

This is essentially LeetCode 76 (Minimum Window Substring). Already covered above.

### Production Variant: Multi-Keyword Search

```java
// Real-world: find smallest window containing all keywords
// Keywords = unique words (not characters)
public String smallestWindowWithKeywords(String text, String[] keywords) {
    Map<String, Integer> need = new HashMap<>();
    for (String kw : keywords) need.merge(kw, 1, Integer::sum);
    int required = need.size();
    
    String[] words = text.split(" ");
    Map<String, Integer> have = new HashMap<>();
    int formed = 0, left = 0;
    int minLen = Integer.MAX_VALUE, minLeft = 0;
    
    for (int right = 0; right < words.length; right++) {
        have.merge(words[right], 1, Integer::sum);
        String rw = words[right];
        if (need.containsKey(rw) && have.get(rw).intValue() == need.get(rw).intValue())
            formed++;
        
        while (formed == required) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minLeft = left;
            }
            String lw = words[left];
            have.merge(lw, -1, Integer::sum);
            if (need.containsKey(lw) && have.get(lw) < need.get(lw)) formed--;
            left++;
        }
    }
    
    if (minLen == Integer.MAX_VALUE) return "";
    return String.join(" ", Arrays.copyOfRange(words, minLeft, minLeft + minLen));
}
```

---

## Session 5 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. Minimum Window Substring: track formed/required, shrink from left when formed==required
2. Character Replacement: valid if windowLen - maxFreq <= k
3. For shrinking-to-minimize problems: shrink while valid, update answer BEFORE shrinking
4. For expanding-to-maximize problems: shrink while INVALID, update answer AFTER

VARIABLES TO TRACK:
- formed: count of satisfied requirements
- required: total distinct requirements  
- maxFreq: max frequency in window (for replacement problem)

CRITICAL INSIGHT:
Minimum Window: 
  while formed == required → window is valid, try to shrink
Character Replacement:
  if windowLen - maxFreq > k → window invalid, shrink by exactly 1

KEY MISTAKE:
In Min Window: using have[c] == need[c] not >=
Actually: use == for the formed increment because:
when we add a character and go from (need-1) to need, that's when we increment.
When we go from need to need+1, we don't increment again.
```

---

# SESSION 6 — PATTERN MATCHING ALGORITHMS (KMP)

---

## 6.1 — Why Naive Pattern Matching is Inefficient

### Concept Explanation

**Naive approach:** Try matching the pattern at every position of the text.

```
Text:    "AAAAAAAAAAAB"  (11 A's then B)
Pattern: "AAAAB"         (4 A's then B)

Naive:
  Position 0: A=A, A=A, A=A, A=A, A≠B → 4 comparisons, fail
  Position 1: A=A, A=A, A=A, A=A, A≠B → 4 comparisons, fail
  ...
  Position 7: A=A, A=A, A=A, A=A, B=B → match!

Total comparisons: ~7 × 5 = 35 ≈ O(n*m)

For n=10^6 text, m=10^3 pattern: 10^9 operations → too slow!
```

**KMP's insight:** When a mismatch occurs after matching some prefix, we already KNOW those matched characters. We can skip re-examining them.

---

## 6.2 — Prefix-Suffix Logic (The Foundation of KMP)

### Concept Explanation

**Proper prefix:** A prefix that is not the whole string.
"abab" → proper prefixes: "a", "ab", "aba"

**Proper suffix:** A suffix that is not the whole string.
"abab" → proper suffixes: "b", "ab", "bab"

**LPS (Longest Proper Prefix which is also Suffix):**
For "abab": "ab" is both a prefix and suffix → LPS length = 2

The **LPS array** (also called **failure function** or **pi array**) for a pattern stores, for each position i, the length of the longest proper prefix of pattern[0..i] that is also a suffix.

```
Pattern: a  b  a  b  c  a
Index:   0  1  2  3  4  5

lps[0] = 0  ("a" has no proper prefix that is also suffix)
lps[1] = 0  ("ab": no prefix-suffix match)
lps[2] = 1  ("aba": "a" is both prefix and suffix, length 1)
lps[3] = 2  ("abab": "ab" is both prefix and suffix, length 2)
lps[4] = 0  ("ababc": no prefix-suffix match)
lps[5] = 1  ("ababca": "a" is both prefix and suffix, length 1)
```

**Why does LPS help?**  
When mismatch at position j in pattern, lps[j-1] tells us the longest prefix of pattern that still matches. We can resume comparison from lps[j-1] instead of starting over from 0.

---

## 6.3 — Building the LPS Array

### Step-by-Step Construction

```java
public int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    lps[0] = 0;  // always 0
    
    int len = 0;  // length of previous longest prefix suffix
    int i = 1;
    
    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            // Characters match: extend the prefix
            len++;
            lps[i] = len;
            i++;
        } else {
            // Characters don't match
            if (len != 0) {
                // Try falling back: use lps[len-1] to find next candidate
                len = lps[len - 1];
                // Don't increment i — try matching with shorter prefix
            } else {
                // len == 0: no prefix-suffix match at all
                lps[i] = 0;
                i++;
            }
        }
    }
    
    return lps;
}
```

**Manual trace for "ababaca":**

```
Pattern: a  b  a  b  a  c  a
Index:   0  1  2  3  4  5  6
LPS:     0  0  1  2  3  0  1

Build:
lps[0]=0 (base case)
i=1, len=0: pattern[1]='b' != pattern[0]='a', len==0, lps[1]=0, i=2
i=2, len=0: pattern[2]='a' == pattern[0]='a', len=1, lps[2]=1, i=3
i=3, len=1: pattern[3]='b' == pattern[1]='b', len=2, lps[3]=2, i=4
i=4, len=2: pattern[4]='a' == pattern[2]='a', len=3, lps[4]=3, i=5
i=5, len=3: pattern[5]='c' != pattern[3]='b', len!=0, len=lps[2]=1
i=5, len=1: pattern[5]='c' != pattern[1]='b', len!=0, len=lps[0]=0
i=5, len=0: pattern[5]='c' != pattern[0]='a', len==0, lps[5]=0, i=6
i=6, len=0: pattern[6]='a' == pattern[0]='a', len=1, lps[6]=1, i=7 → done
Result: [0,0,1,2,3,0,1]
```

---

## 6.4 — KMP Search Algorithm

### The Full KMP Implementation

```java
public List<Integer> kmpSearch(String text, String pattern) {
    List<Integer> matches = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    
    if (m == 0) return matches;
    
    int[] lps = buildLPS(pattern);
    
    int i = 0;  // index for text
    int j = 0;  // index for pattern
    
    while (i < n) {
        if (text.charAt(i) == pattern.charAt(j)) {
            i++;
            j++;
        }
        
        if (j == m) {
            // Full match found!
            matches.add(i - j);  // start index = i - m
            j = lps[j - 1];      // use LPS to continue searching for more matches
        } else if (i < n && text.charAt(i) != pattern.charAt(j)) {
            if (j != 0) {
                // Don't move i backward; use LPS to skip ahead
                j = lps[j - 1];
            } else {
                i++;  // pattern[0] didn't match, move text pointer
            }
        }
    }
    
    return matches;
}

// Complete KMP implementation
public int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    int len = 0, i = 1;
    
    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            lps[i++] = ++len;
        } else if (len != 0) {
            len = lps[len - 1];
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}

// Total Time: O(n + m)
// O(m) to build LPS, O(n) to search
// Space: O(m) for LPS array
```

**Trace for text="AABXAAB", pattern="AABX":**

```
LPS for "AABX": [0,1,0,0]

i=0,j=0: A==A, i=1,j=1
i=1,j=1: A==A, i=2,j=2
i=2,j=2: B==B, i=3,j=3
i=3,j=3: X==X, i=4,j=4 → j==m=4! MATCH at index 0
  j = lps[3] = 0

i=4,j=0: A==A, i=5,j=1
i=5,j=1: A==A, i=6,j=2
i=6,j=2: B==B, i=7,j=3 → i==n, exit
No more matches. Result: [0]
```

---

## 6.5 — Problem: Implement strStr() / Find Needle in Haystack (LeetCode 28)

```java
public int strStr(String haystack, String needle) {
    if (needle.isEmpty()) return 0;
    if (needle.length() > haystack.length()) return -1;
    
    int[] lps = buildLPS(needle);
    int i = 0, j = 0;
    
    while (i < haystack.length()) {
        if (haystack.charAt(i) == needle.charAt(j)) {
            i++; j++;
        }
        if (j == needle.length()) {
            return i - j;  // first occurrence found
        } else if (i < haystack.length() && haystack.charAt(i) != needle.charAt(j)) {
            if (j != 0) j = lps[j - 1];
            else i++;
        }
    }
    
    return -1;  // not found
}
```

---

## 6.6 — Problem: Repeated Substring Pattern (LeetCode 459)

### Concept Explanation

Does a string consist of repeating copies of a substring?
"abab" → true (two copies of "ab")
"aba" → false
"abcabcabcabc" → true (four copies of "abc")

### The KMP Trick Solution

```java
// Key insight using LPS:
// If s can be formed by repeating substring p, then:
// lps[n-1] > 0 AND n % (n - lps[n-1]) == 0

public boolean repeatedSubstringPattern(String s) {
    int n = s.length();
    int[] lps = buildLPS(s);
    
    int longestBorder = lps[n - 1];  // longest proper prefix = suffix
    if (longestBorder == 0) return false;
    
    int patternLen = n - longestBorder;
    return n % patternLen == 0;
}

// Example "abab":
// LPS: [0,0,1,2]
// longestBorder = 2
// patternLen = 4 - 2 = 2
// 4 % 2 == 0 → true, pattern is "ab"

// Alternative (elegant string trick):
public boolean repeatedSubstringPatternAlt(String s) {
    String doubled = s + s;
    // Remove first and last character
    String middle = doubled.substring(1, doubled.length() - 1);
    // If s is in middle, it's made of repeating substrings
    return middle.contains(s);
    // "abab" + "abab" = "abababab"
    // middle = "bababa"
    // "bababa".contains("abab") → true ✓
}
// Why? If s has period p, then in doubled string,
// after removing first and last char, s must appear at offset p.
```

---

## Session 6 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. KMP avoids rechecking matched characters on mismatch
2. LPS[i] = length of longest proper prefix of pattern[0..i] that is also a suffix
3. Total KMP time: O(n + m) — O(m) preprocessing, O(n) search
4. On mismatch at j: j = lps[j-1] (fall back, don't reset to 0)
5. On mismatch at j=0: i++ (advance text pointer)

LPS CONSTRUCTION RULE:
len=0, i=1
if pattern[i]==pattern[len]: lps[i]=++len, i++
elif len!=0: len=lps[len-1]  (fall back)
else: lps[i]=0, i++

LPS for "ababaca" = [0,0,1,2,3,0,1]
LPS for "aabxaab" = [0,1,0,0,1,2,3]
LPS for "aaaa"    = [0,1,2,3]
LPS for "abcd"    = [0,0,0,0]

INTERVIEW TIP:
If asked "implement strStr()", know both:
- Naive O(n*m): acceptable for interview if you mention KMP
- KMP O(n+m): shows deeper knowledge, mention LPS array
```

---

# SESSION 7 — ROLLING HASH & RABIN-KARP

---

## 7.1 — Why Hashing for String Matching?

### Concept Explanation

**Rolling hash** is an alternative to KMP for pattern matching. Instead of structural preprocessing (LPS), it uses mathematical hashing to compare substrings in O(1).

**Key idea:** Compute a hash for the pattern. Slide a window of same size over the text, computing hash of each window. If hashes match, verify with character comparison (to handle collisions).

**When to prefer over KMP:**
- Multiple pattern search
- Repeated substring finding
- Competitive programming (faster to code)
- Problems involving multiple substring comparisons

---

## 7.2 — Polynomial Rolling Hash

### Mathematical Foundation

For a string `s` of length `m`, its hash is:

```
hash(s) = s[0]*p^(m-1) + s[1]*p^(m-2) + ... + s[m-1]*p^0

Where:
- p = prime base (typically 31 for lowercase, 131 for mixed)
- Values: 'a'=1, 'b'=2, ..., 'z'=26 (or use char values directly)
- Take modulo MOD to prevent overflow (typically 10^9+7 or 10^9+9)
```

**Rolling update formula:**
```
When sliding window by one position (remove leftmost char, add new rightmost char):

old_hash = s[i]*p^(m-1) + s[i+1]*p^(m-2) + ... + s[i+m-1]*p^0
new_hash = s[i+1]*p^(m-1) + ... + s[i+m]*p^0

Relation:
new_hash = (old_hash - s[i]*p^(m-1)) * p + s[i+m]
```

```java
// Rabin-Karp implementation
public int search(String text, String pattern) {
    int n = text.length(), m = pattern.length();
    if (m > n) return -1;
    
    final long MOD = 1_000_000_007L;
    final long BASE = 31L;
    
    // Compute BASE^(m-1) % MOD
    long power = 1;
    for (int i = 0; i < m - 1; i++) {
        power = (power * BASE) % MOD;
    }
    
    // Compute hash of pattern and first window
    long patHash = 0, winHash = 0;
    for (int i = 0; i < m; i++) {
        patHash = (patHash * BASE + (pattern.charAt(i) - 'a' + 1)) % MOD;
        winHash = (winHash * BASE + (text.charAt(i) - 'a' + 1)) % MOD;
    }
    
    for (int i = 0; i <= n - m; i++) {
        if (winHash == patHash) {
            // Hash match — verify character by character (handle collisions)
            if (text.substring(i, i + m).equals(pattern)) {
                return i;  // found at index i
            }
        }
        
        // Roll the hash: remove leftmost, add new rightmost
        if (i < n - m) {
            winHash = (winHash - (text.charAt(i) - 'a' + 1) * power % MOD + MOD) % MOD;
            winHash = (winHash * BASE + (text.charAt(i + m) - 'a' + 1)) % MOD;
        }
    }
    
    return -1;
}
// Time: O(n+m) average, O(n*m) worst case (hash collision)
// Space: O(1) additional
```

---

## 7.3 — Collision Handling

### Concept Explanation

**Hash collision:** Two different strings with the same hash value. This is rare but must be handled.

**Strategies:**
1. **Verification step:** When hash matches, verify with character comparison (standard)
2. **Double hashing:** Use two different (MOD, BASE) pairs — probability of false positive drops to ~10^-18
3. **Larger MOD:** Use 64-bit arithmetic to reduce collision probability

```java
// Double hashing (production-grade)
final long MOD1 = 1_000_000_007L, BASE1 = 31L;
final long MOD2 = 998_244_353L, BASE2 = 37L;

// Compute two hashes; collision only if BOTH match
// Probability of false positive: ~10^-18
```

---

## 7.4 — Problem: Repeated DNA Sequences (LeetCode 187)

### Concept Explanation

Find all 10-letter sequences that appear more than once in DNA string.
"AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT" → ["AAAAACCCCC","CCCCCAAAAA"]

```java
// Approach 1: HashSet — O(n) time and space
public List<String> findRepeatedDnaSequences(String s) {
    Set<String> seen = new HashSet<>();
    Set<String> repeated = new HashSet<>();
    
    for (int i = 0; i <= s.length() - 10; i++) {
        String sub = s.substring(i, i + 10);
        if (!seen.add(sub)) {     // add returns false if already present
            repeated.add(sub);
        }
    }
    
    return new ArrayList<>(repeated);
}
// Note: substring() is O(10)=O(1) here, so overall O(n)
// Space: O(n) for hash sets

// Approach 2: Rolling hash — O(n) time, better constant
public List<String> findRepeatedDnaSequencesHash(String s) {
    // Encode A=0, C=1, G=2, T=3
    Map<Character, Integer> code = Map.of('A', 0, 'C', 1, 'G', 2, 'T', 3);
    
    // Each character needs 2 bits; 10 chars = 20 bits
    // Use bit manipulation rolling hash
    Map<Integer, Integer> seen = new HashMap<>();
    List<String> result = new ArrayList<>();
    
    int hash = 0, mask = (1 << 20) - 1;  // 20-bit mask
    
    for (int i = 0; i < s.length(); i++) {
        hash = ((hash << 2) | code.get(s.charAt(i))) & mask;
        
        if (i >= 9) {  // window is full (10 chars)
            seen.merge(hash, 1, Integer::sum);
            if (seen.get(hash) == 2) {  // second time seen
                result.add(s.substring(i - 9, i + 1));
            }
        }
    }
    
    return result;
}
// Bit rolling hash: shift left by 2, OR new 2-bit code, AND with 20-bit mask
// When window slides: old leftmost char is automatically removed by the mask
// Time: O(n), Space: O(n)
```

---

## 7.5 — Problem: Longest Duplicate Substring (Advanced)

```java
// Binary search + rolling hash
// Find longest substring that appears at least twice
public String longestDupSubstring(String s) {
    int left = 1, right = s.length() - 1;
    String result = "";
    
    while (left <= right) {
        int mid = (left + right) / 2;
        String dup = findDuplicate(s, mid);
        if (dup != null) {
            result = dup;
            left = mid + 1;  // try longer
        } else {
            right = mid - 1;  // try shorter
        }
    }
    
    return result;
}

private String findDuplicate(String s, int len) {
    final long MOD = 1_000_000_007L, BASE = 31L;
    
    long power = 1;
    for (int i = 0; i < len - 1; i++) power = (power * BASE) % MOD;
    
    long hash = 0;
    for (int i = 0; i < len; i++) hash = (hash * BASE + s.charAt(i)) % MOD;
    
    Map<Long, List<Integer>> seen = new HashMap<>();
    seen.computeIfAbsent(hash, k -> new ArrayList<>()).add(0);
    
    for (int i = 1; i <= s.length() - len; i++) {
        hash = (hash - s.charAt(i - 1) * power % MOD + MOD) % MOD;
        hash = (hash * BASE + s.charAt(i + len - 1)) % MOD;
        
        if (seen.containsKey(hash)) {
            for (int prev : seen.get(hash)) {
                if (s.substring(prev, prev + len).equals(s.substring(i, i + len))) {
                    return s.substring(i, i + len);
                }
            }
        }
        seen.computeIfAbsent(hash, k -> new ArrayList<>()).add(i);
    }
    return null;
}
// Time: O(n log n), Space: O(n)
```

---

## Session 7 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. Rolling hash: compute hash of window, update in O(1) per slide
2. Hash update: newHash = (oldHash - leftmost*power) * base + newRightmost
3. Always verify on hash match (handle collisions)
4. For DNA: use 2-bit encoding, 10 chars = 20 bits, fits in int

ROLLING HASH FORMULA:
hash(s[i+1..i+m]) = (hash(s[i..i+m-1]) - s[i]*BASE^(m-1)) * BASE + s[i+m]
All modulo MOD

MODULAR SUBTRACTION:
When subtracting, add MOD first to prevent negative values:
hash = (hash - val + MOD) % MOD

COMMON MISTAKES:
1. Forgetting +MOD when subtracting (negative hash mod in Java is negative!)
2. Integer overflow: use long for intermediate calculations
3. Not verifying on collision → wrong answers
4. Wrong power calculation (should be BASE^(m-1) not BASE^m)
```

---

# SESSION 8 — Z ALGORITHM & ADVANCED STRING PROCESSING

---

## 8.1 — The Z Array

### Concept Explanation

The **Z array** for a string `s` is defined as: `Z[i]` = length of the longest substring starting from `s[i]` that is also a prefix of `s`.

```
s = "aabxaabxcaabxaabxay"

Z[0] = undefined (or length of whole string by convention)
Z[1] = 1   "abxaabxcaabxaabxay" starts with "a" which matches prefix "a" → Z[1]=1
Z[2] = 0   "bxaabxcaabxaabxay" doesn't start with "a"
Z[3] = 0   "xaabxcaabxaabxay" doesn't start with "a"
Z[4] = 4   "aabxcaabxaabxay" matches "aabx" → Z[4]=4
Z[5] = 1   "abxcaabxaabxay" → "a" matches → Z[5]=1
Z[6] = 0   ...
Z[7] = 0
Z[8] = 0
Z[9] = 4   "aabxaabxay" matches "aabx" → Z[9]=4... wait let me recompute:
           s[9..] = "aabxaabxay"
           prefix: "aabxaabxcaabxaabxay"
           match: a=a,a=a,b=b,x=x,a=a,a=a,b=b,x=x... comparing 9th char onwards
           s[9]='a',s[0]='a'✓; s[10]='a',s[1]='a'✓; s[11]='b',s[2]='b'✓; s[12]='x',s[3]='x'✓
           s[13]='a',s[4]='a'✓; s[14]='a',s[5]='a'✓; s[15]='b',s[6]='b'✓; s[16]='x',s[7]='x'✓
           s[17]='a',s[8]='c' ✗ → Z[9]=8

For pattern matching: concatenate pattern + '$' + text
Z[i] == pattern.length() means a match at position (i - pattern.length() - 1) in text
```

---

## 8.2 — Building the Z Array Efficiently

### O(n) Construction

```java
public int[] buildZArray(String s) {
    int n = s.length();
    int[] z = new int[n];
    z[0] = n;  // by convention, entire string matches itself
    
    // [l, r] = current Z-box (rightmost window where Z is non-zero)
    int l = 0, r = 0;
    
    for (int i = 1; i < n; i++) {
        if (i < r) {
            // i is inside the Z-box [l, r]
            // Use previously computed Z value (relative position: i - l)
            z[i] = Math.min(r - i, z[i - l]);
        }
        
        // Try to extend Z[i] beyond the Z-box
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) {
            z[i]++;
        }
        
        // Update Z-box if this extends further right
        if (i + z[i] > r) {
            l = i;
            r = i + z[i];
        }
    }
    
    return z;
}
```

**Mental model for Z-box:**
```
The Z-box [l, r] represents the window [l, r-1] where:
- s[l..r-1] equals s[0..r-l-1]
- This is the rightmost such window we've found

When computing Z[i]:
- If i < r: we can initialize Z[i] = min(r-i, Z[i-l])
  - Z[i-l] is what we computed at mirror position (i-l) relative to l
  - r-i is how far we can trust the mirror (within Z-box)
  - Take minimum: conservative start
- Then extend naively (the while loop)
- Extending is O(1) amortized because r only moves right
```

---

## 8.3 — Pattern Matching Using Z Algorithm

```java
public List<Integer> zSearch(String text, String pattern) {
    String combined = pattern + "$" + text;
    int[] z = buildZArray(combined);
    
    List<Integer> matches = new ArrayList<>();
    int m = pattern.length();
    
    for (int i = m + 1; i < combined.length(); i++) {
        if (z[i] == m) {
            matches.add(i - m - 1);  // position in text
        }
    }
    
    return matches;
}

// "pattern$text"
// If Z[i] == pattern.length(), then:
// - combined[i .. i+m-1] == combined[0 .. m-1] == pattern
// - Position in text: i - m - 1 (subtract pattern length and '$')
```

---

## 8.4 — KMP vs Z Algorithm Comparison

```
Feature          | KMP                      | Z Algorithm
-----------------+--------------------------+-------------------------
Preprocessing    | LPS/failure function     | Z array
Time             | O(n + m)                 | O(n + m)
Space            | O(m)                     | O(n + m) for combined
Conceptual       | Prefix-suffix overlap    | Prefix matching
Implementation   | Slightly complex         | Cleaner
Use case         | Single pattern search    | Prefix matching problems
Competitive CP   | Common                   | More common
Interview        | Very common              | Less common
```

---

## 8.5 — Problem: String Score Problems

```java
// Score of a String (LeetCode 3110)
// Score = sum of |s[i] - s[i+1]| for all adjacent pairs
public int scoreOfString(String s) {
    int score = 0;
    for (int i = 0; i < s.length() - 1; i++) {
        score += Math.abs(s.charAt(i) - s.charAt(i + 1));
    }
    return score;
}

// "hello" → |h-e|+|e-l|+|l-l|+|l-o| = |104-101|+|101-108|+|108-108|+|108-111|
//          = 3 + 7 + 0 + 3 = 13
```

---

## 8.6 — Building Z Array for "aabxaabxcaabxaabxay"

**Manual trace:**

```
s = "aabxaabxcaabxaabxay"
n = 19

Initialize: z[0] = 19, l = 0, r = 0

i=1: i >= r? yes (1 >= 0), z[1]=0
  extend: s[0]='a', s[1]='a' → match, z[1]=1
         s[1]='a', s[2]='b' → s[0]='a' ✗ → stop, z[1]=1
  r = 1+1 = 2 > 0, update l=1, r=2

i=2: i < r? 2 < 2? no, z[2]=0
  extend: s[0]='a', s[2]='b' → no match, z[2]=0

i=3: z[3]=0, s[0]='a',s[3]='x' → no match

i=4: z[4]=0
  extend: s[0..] = "aabx...", s[4..] = "aabx..."
  s[0]='a'=s[4]='a'✓, z=1
  s[1]='a'=s[5]='a'✓, z=2
  s[2]='b'=s[6]='b'✓, z=3
  s[3]='x'=s[7]='x'✓, z=4
  s[4]='a',s[8]='c'✗, stop → z[4]=4
  update l=4, r=8

i=5: i < r (5 < 8), z[5] = min(r-i, z[i-l]) = min(8-5=3, z[5-4]=z[1]=1) = 1
  extend from z=1: s[1]='a',s[6]='b'? 'a'!='b', stop. z[5]=1
  5+1=6, not > r=8, no update

i=6: i < r (6 < 8), z[6] = min(8-6=2, z[6-4]=z[2]=0) = 0
  extend from z=0: s[0]='a',s[6]='b'? no. z[6]=0

i=7: i < r (7 < 8), z[7] = min(8-7=1, z[7-4]=z[3]=0) = 0
  extend: s[0]='a',s[7]='x'? no. z[7]=0

i=8: i >= r, z[8]=0
  extend: s[0]='a',s[8]='c'? no. z[8]=0

i=9: z[9]=0
  extend: s[0]='a'=s[9]='a'✓,s[1]='a'=s[10]='a'✓,...
  match 8 chars (s[0..7] == s[9..16]), z[9]=8
  update l=9, r=17

... and so on
```

---

## 8.7 — Border Strings / Prefix Function Review

```java
// Prefix function (KMP's LPS) is also called "pi function" in CP
// pi[i] = length of longest border of s[0..i]
// border = string that is both proper prefix and proper suffix

// Useful for:
// 1. Counting occurrences of pattern in text (KMP)
// 2. Finding minimum period of string
// 3. Detecting repeated substrings

// Minimum period of string using LPS:
// period = n - lps[n-1]
// If n % period == 0: string is composed of period-length repeats

public int minPeriod(String s) {
    int[] lps = buildLPS(s);
    int n = s.length();
    int period = n - lps[n - 1];
    return (n % period == 0) ? period : n;
}
// "abcabcabc" → period=3 ("abc")
// "abcabc" → period=3
// "abcd" → period=4 (itself)
```

---

## Session 8 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. Z[i] = length of longest substring starting at i that matches prefix of s
2. Z-box [l,r]: rightmost known matching window; enables O(1) initialization
3. For pattern matching: use "pattern$text", Z[i]==m means match at i-m-1
4. Z algorithm time: O(n) — r pointer only moves right, never backwards
5. KMP and Z are equivalent in power; different in structure

Z ARRAY RULES:
Z[0] = n (whole string matches itself)
If i < r: Z[i] = min(r - i, Z[i - l])  [use Z-box]
Then extend naively (while loop)
Update Z-box if i + Z[i] > r

Z FOR PATTERN MATCHING:
s = pattern + "$" + text
Z[i] == pattern.length() → match at text[i - m - 1]
The '$' ensures pattern and text don't overlap in Z computation

BUILD Z FOR:
"aabxaab" → [7, 1, 0, 0, 4, 1, 0]
"aaaa"    → [4, 3, 2, 1]
"abcabc"  → [6, 0, 0, 3, 0, 0]
```

---

# SESSION 9 — ENCODING, DECODING & PARSING PROBLEMS

---

## 9.1 — Run-Length Encoding

### Concept Explanation

**Run-length encoding (RLE)** compresses consecutive identical characters by storing the character once with its count.
"aaabbbccddddde" → "a3b3c2d5e1" or "3a3b2c5d1e"

**Real-world use:** Image compression, DNA sequence storage, simple text compression.

### Implementation

```java
// Compress: "aaabbbccddddde" → "3a3b2c5d1e"
public String compress(String s) {
    StringBuilder sb = new StringBuilder();
    int i = 0;
    
    while (i < s.length()) {
        char c = s.charAt(i);
        int count = 0;
        
        // Count consecutive same characters
        while (i < s.length() && s.charAt(i) == c) {
            count++;
            i++;
        }
        
        sb.append(count).append(c);  // "3a"
    }
    
    return sb.toString();
}

// Decompress: "3a3b2c" → "aaabbbcc"
public String decompress(String s) {
    StringBuilder sb = new StringBuilder();
    int i = 0;
    
    while (i < s.length()) {
        // Parse number (could be multi-digit: "12a")
        int count = 0;
        while (i < s.length() && Character.isDigit(s.charAt(i))) {
            count = count * 10 + (s.charAt(i) - '0');
            i++;
        }
        char c = s.charAt(i++);
        
        for (int j = 0; j < count; j++) sb.append(c);
    }
    
    return sb.toString();
}
```

---

## 9.2 — Problem: String Compression (LeetCode 443)

```java
// In-place compression: "aabcccccaaa" → "a2b1c5a3"
// Return compressed length; modify input char array in-place
public int compress(char[] chars) {
    int write = 0;  // write pointer
    int i = 0;      // read pointer
    
    while (i < chars.length) {
        char c = chars[i];
        int count = 0;
        
        // Count consecutive same characters
        while (i < chars.length && chars[i] == c) {
            i++;
            count++;
        }
        
        // Write character
        chars[write++] = c;
        
        // Write count (only if > 1)
        if (count > 1) {
            String countStr = String.valueOf(count);
            for (char digit : countStr.toCharArray()) {
                chars[write++] = digit;
            }
        }
    }
    
    return write;  // new length
}
// "aabcccccaaa" → chars becomes "a2b1c5a3...", return 8
// Actually: count=1 for 'b' → don't write count, so "a2bc5a3", return 7
// "a" count=2 → "a2"; "b" count=1 → "b"; "c" count=5 → "c5"; "a" count=3 → "a3"
// Result: ['a','2','b','c','5','a','3'] → 7
```

---

## 9.3 — Problem: Count and Say (LeetCode 38)

### Concept Explanation

Generate the nth term of the "count-and-say" sequence.
- 1: "1"
- 2: "11" (one '1')
- 3: "21" (two '1's)
- 4: "1211" (one '2', one '1')
- 5: "111221" (one '1', one '2', two '1's)

```java
public String countAndSay(int n) {
    String current = "1";
    
    for (int i = 1; i < n; i++) {
        StringBuilder next = new StringBuilder();
        int j = 0;
        
        while (j < current.length()) {
            char c = current.charAt(j);
            int count = 0;
            
            while (j < current.length() && current.charAt(j) == c) {
                j++;
                count++;
            }
            
            next.append(count).append(c);
        }
        
        current = next.toString();
    }
    
    return current;
}
```

---

## 9.4 — Stack-Based Parsing

### Concept Explanation

A **stack** is perfect for parsing nested or sequential structures because:
- Stacks follow LIFO (Last In, First Out)
- Nested expressions resolve from innermost to outermost
- Brackets/parentheses/braces are naturally stack-compatible

**When to use stack in string problems:**
- Parenthesis matching/validation
- Expression evaluation
- Nested structure decoding
- Bracket sequence generation

---

## 9.5 — Problem: Valid Parentheses (LeetCode 20)

```java
// "{[()]}" → true; "{[}" → false; "({)" → false
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    
    for (char c : s.toCharArray()) {
        // Push opening brackets
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            // Closing bracket: check if matches top
            if (stack.isEmpty()) return false;
            
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    
    return stack.isEmpty();  // all brackets must be matched
}

// Cleaner version using Map
public boolean isValidClean(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> map = Map.of(')', '(', ']', '[', '}', '{');
    
    for (char c : s.toCharArray()) {
        if (map.containsValue(c)) {  // opening bracket
            stack.push(c);
        } else {  // closing bracket
            if (stack.isEmpty() || stack.pop() != map.get(c)) return false;
        }
    }
    
    return stack.isEmpty();
}
```

---

## 9.6 — Problem: Decode String (LeetCode 394)

### Concept Explanation

Given encoded string like "3[a]2[bc]", decode to "aaabcbc".
"3[a2[c]]" → "accaccacc"
"2[abc]3[cd]ef" → "abcabccdcdcdef"

### Deep Understanding

Use a stack to handle nested brackets. When we encounter `[`, push current state. When we encounter `]`, pop and repeat.

```java
public String decodeString(String s) {
    Deque<Integer> countStack = new ArrayDeque<>();
    Deque<StringBuilder> strStack = new ArrayDeque<>();
    
    StringBuilder current = new StringBuilder();
    int k = 0;  // current number being built
    
    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            // Build multi-digit number
            k = k * 10 + (c - '0');
        } else if (c == '[') {
            // Save current state, start fresh
            countStack.push(k);
            strStack.push(current);
            k = 0;
            current = new StringBuilder();
        } else if (c == ']') {
            // Pop count and previous string
            int count = countStack.pop();
            StringBuilder prev = strStack.pop();
            
            // Repeat current string 'count' times, append to previous
            for (int i = 0; i < count; i++) {
                prev.append(current);
            }
            current = prev;
        } else {
            // Regular character
            current.append(c);
        }
    }
    
    return current.toString();
}

// Trace "3[a2[c]]":
// '3': k=3
// '[': push k=3, push current="", reset k=0, current=""
// 'a': current="a"
// '2': k=2
// '[': push k=2, push current="a", reset k=0, current=""
// 'c': current="c"
// ']': count=2, prev="a", repeat "c" 2 times → prev="acc", current="acc"
// ']': count=3, prev="", repeat "acc" 3 times → prev="accaccacc", current="accaccacc"
// Result: "accaccacc" ✓
```

---

## 9.7 — Problem: Roman to Integer (LeetCode 13)

### Concept Explanation

Roman numeral rules:
- If current value < next value: subtract current (IV = 4, IX = 9)
- Otherwise: add current (VI = 6, VIII = 8)

```java
public int romanToInt(String s) {
    Map<Character, Integer> map = Map.of(
        'I', 1, 'V', 5, 'X', 10, 'L', 50,
        'C', 100, 'D', 500, 'M', 1000
    );
    
    int result = 0;
    int n = s.length();
    
    for (int i = 0; i < n; i++) {
        int current = map.get(s.charAt(i));
        int next = (i + 1 < n) ? map.get(s.charAt(i + 1)) : 0;
        
        if (current < next) {
            result -= current;  // subtraction case: IV, IX, etc.
        } else {
            result += current;  // normal addition
        }
    }
    
    return result;
}
// "MCMXCIV":
// M=1000, C=100(next=M=1000 > 100, subtract), M=1000, X=10(next=C=100>10, subtract),
// C=100, I=1(next=V=5>1, subtract), V=5
// 1000 - 100 + 1000 - 10 + 100 - 1 + 5 = 1994
```

---

## 9.8 — Problem: String to Integer (atoi) (LeetCode 8)

### Concept Explanation

Implement atoi: convert string to 32-bit signed integer, handling leading whitespace, sign, non-numeric characters, and overflow.

```java
public int myAtoi(String s) {
    int i = 0, n = s.length();
    
    // Step 1: Skip leading whitespace
    while (i < n && s.charAt(i) == ' ') i++;
    
    if (i == n) return 0;
    
    // Step 2: Determine sign
    int sign = 1;
    if (s.charAt(i) == '-') { sign = -1; i++; }
    else if (s.charAt(i) == '+') { i++; }
    
    // Step 3: Parse digits, stop at first non-digit
    long result = 0;
    while (i < n && Character.isDigit(s.charAt(i))) {
        result = result * 10 + (s.charAt(i) - '0');
        i++;
        
        // Early overflow check
        if (result * sign > Integer.MAX_VALUE) return Integer.MAX_VALUE;
        if (result * sign < Integer.MIN_VALUE) return Integer.MIN_VALUE;
    }
    
    return (int)(result * sign);
}
// "  -42" → -42
// "4193 with words" → 4193
// "-91283472332" → -2147483648 (Integer.MIN_VALUE)
```

---

## 9.9 — Problem: Generate Parentheses (LeetCode 22)

### Concept Explanation

Generate all valid combinations of n pairs of parentheses.
n=2 → ["(())", "()()"]
n=3 → ["((()))", "(()())", "(())()", "()(())", "()()()"]

**Rules for valid generation:**
- Can add '(' if open count < n
- Can add ')' if close count < open count

```java
public List<String> generateParenthesis(int n) {
    List<String> result = new ArrayList<>();
    generate(result, new StringBuilder(), 0, 0, n);
    return result;
}

private void generate(List<String> result, StringBuilder current, 
                      int open, int close, int n) {
    // Base case: used all n pairs
    if (current.length() == 2 * n) {
        result.add(current.toString());
        return;
    }
    
    // Add '(' if we haven't used all n open parens
    if (open < n) {
        current.append('(');
        generate(result, current, open + 1, close, n);
        current.deleteCharAt(current.length() - 1);  // backtrack
    }
    
    // Add ')' if close count is less than open count
    if (close < open) {
        current.append(')');
        generate(result, current, open, close + 1, n);
        current.deleteCharAt(current.length() - 1);  // backtrack
    }
}
// All valid strings are generated exactly once.
// Total combinations = Catalan number = C(2n, n) / (n+1)
```

---

## 9.10 — Problem: Decode Ways (LeetCode 91) — Preview

```java
// "226" can be decoded as: "2","2","6"→BBF or "22","6"→VF or "2","26"→BZ
// How many ways? → 3
// Full solution in Session 10 (DP)
```

---

## Session 9 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. Run-length encoding: count consecutive same chars, output count+char
2. For multi-digit numbers in strings: count = count*10 + digit
3. Stack for nested structures: push on '[', pop and repeat on ']'
4. For Roman numerals: subtract if current < next, else add
5. For atoi: skip spaces, parse sign, parse digits, handle overflow

STACK PARSING TEMPLATE:
for c in s:
    if digit: build number
    if '[': push current state
    if ']': pop state, combine  
    if letter: append to current

PARENTHESIS GENERATION:
- Add '(' when open < n
- Add ')' when close < open
- This guarantees validity by construction

COMMON MISTAKES:
1. Multi-digit number parsing: forgetting *10 (treating "12" as "1" and "2")
2. Stack underflow: not checking isEmpty() before pop
3. Not handling '+'/'-' sign in atoi
4. Integer overflow in atoi: use long, check before returning
```

---

# SESSION 10 — DP ON STRINGS + INTERVIEW MASTERY

---

## 10.1 — DP on Strings: Foundation

### Concept Explanation

**Dynamic Programming (DP) on strings** is used when:
- The problem asks for optimal operations on substrings/subsequences
- State depends on prefixes of one or two strings
- Choices involve including/excluding characters

**Key distinction:**
- **Substring:** Contiguous part of string — s[i..j]
- **Subsequence:** Characters in order but not necessarily contiguous — chars from s maintaining relative order

**Common DP string patterns:**
1. Single string DP: `dp[i]` = answer for s[0..i]
2. Two-string DP: `dp[i][j]` = answer for s1[0..i] and s2[0..j]
3. Interval DP: `dp[i][j]` = answer for s[i..j]

---

## 10.2 — Problem: Word Break (LeetCode 139)

### Concept Explanation

Given string s and dictionary wordDict, can s be segmented into dictionary words?
s="leetcode", dict=["leet","code"] → true
s="applepenapple", dict=["apple","pen"] → true
s="catsandog", dict=["cats","dog","sand","and","cat"] → false

### Solutions

```java
// DP approach: dp[i] = true if s[0..i-1] can be segmented
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);  // O(1) lookup
    int n = s.length();
    boolean[] dp = new boolean[n + 1];
    dp[0] = true;  // empty string is always segmentable
    
    for (int i = 1; i <= n; i++) {
        // Check every possible last word ending at position i
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.contains(s.substring(j, i))) {
                dp[i] = true;
                break;  // found a valid segmentation, no need to check more j values
            }
        }
    }
    
    return dp[n];
}

// Trace: s="leetcode", dict={"leet","code"}
// dp[0]=true
// i=1: j=0, dp[0]=true, "l"∈dict? no → dp[1]=false
// i=2: j=0, "le"? no; j=1, dp[1]=false → dp[2]=false
// i=3: j=0, "lee"? no; ... → dp[3]=false
// i=4: j=0, dp[0]=true, "leet"∈dict? YES → dp[4]=true
// i=5: j=0, "leetc"? no; j=1,dp[1]=false;...j=4,dp[4]=true,"c"? no → dp[5]=false
// i=6: j=4,dp[4]=true,"co"? no → dp[6]=false
// i=7: j=4,dp[4]=true,"cod"? no → dp[7]=false
// i=8: j=4,dp[4]=true,"code"∈dict? YES → dp[8]=true ✓

// Time: O(n^2) outer*inner × O(k) for substring/lookup
// Optimize: use Trie for dictionary lookup
// Space: O(n)
```

---

## 10.3 — Problem: Decode Ways (LeetCode 91)

### Concept Explanation

Given a string of digits, count the number of ways to decode it.
Mapping: '1'→A, '2'→B, ..., '26'→Z
"226" → 3 ("2","2","6"=BBF or "22","6"=VF or "2","26"=BZ)
"06" → 0 (leading zeros are invalid)

```java
public int numDecodings(String s) {
    int n = s.length();
    int[] dp = new int[n + 1];
    dp[0] = 1;  // empty string: 1 way to decode (do nothing)
    dp[1] = s.charAt(0) != '0' ? 1 : 0;  // single char: 1 way unless '0'
    
    for (int i = 2; i <= n; i++) {
        int oneDigit = Integer.parseInt(s.substring(i - 1, i));    // s[i-1]
        int twoDigit = Integer.parseInt(s.substring(i - 2, i));    // s[i-2..i-1]
        
        // Single character decode: s[i-1] must be 1-9
        if (oneDigit >= 1) {
            dp[i] += dp[i - 1];
        }
        
        // Two character decode: s[i-2..i-1] must be 10-26
        if (twoDigit >= 10 && twoDigit <= 26) {
            dp[i] += dp[i - 2];
        }
    }
    
    return dp[n];
}

// Trace: s="226"
// dp[0]=1, dp[1]=1 ('2'!=0)
// i=2: oneDigit='2'>=1, dp[2]+=dp[1]=1; twoDigit=22,10<=22<=26, dp[2]+=dp[0]=1 → dp[2]=2
// i=3: oneDigit='6'>=1, dp[3]+=dp[2]=2; twoDigit=26,10<=26<=26, dp[3]+=dp[1]=1 → dp[3]=3
// Result: 3 ✓

// Edge cases:
// "30": dp[1]='3'≠0→1; i=2: one='0'<1→skip; two=30>26→skip → dp[2]=0
// "10": dp[1]='1'≠0→1; i=2: one='0'<1→skip; two=10,10<=10<=26 → dp[2]+=dp[0]=1 → dp[2]=1
// "0": dp[1]='0'==0→0 → dp[1]=0

// Space optimization: only need dp[i-1] and dp[i-2]
public int numDecodingsSpaceOpt(String s) {
    int prev2 = 1;  // dp[i-2]
    int prev1 = s.charAt(0) != '0' ? 1 : 0;  // dp[i-1]
    
    for (int i = 2; i <= s.length(); i++) {
        int curr = 0;
        int one = Integer.parseInt(s.substring(i - 1, i));
        int two = Integer.parseInt(s.substring(i - 2, i));
        if (one >= 1) curr += prev1;
        if (two >= 10 && two <= 26) curr += prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    
    return prev1;
}
```

---

## 10.4 — Problem: Longest Common Subsequence (LeetCode 1143)

### Concept Explanation

LCS is the longest sequence of characters that appears in the same order (not necessarily contiguous) in both strings.
"abcde" and "ace" → LCS = "ace" (length 3)
"abc" and "abc" → LCS = "abc" (length 3)
"abc" and "def" → LCS = "" (length 0)

### Deep Understanding

```
dp[i][j] = LCS of s1[0..i-1] and s2[0..j-1]

Recurrence:
  If s1[i-1] == s2[j-1]:   dp[i][j] = dp[i-1][j-1] + 1
  Else:                      dp[i][j] = max(dp[i-1][j], dp[i][j-1])

Intuition:
  If current chars match: extend LCS by 1 (include both)
  If don't match: take best of (skip char from s1) or (skip char from s2)
```

```java
public int longestCommonSubsequence(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;  // chars match: extend
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);  // take best
            }
        }
    }
    
    return dp[m][n];
}

// Trace: s1="abcde", s2="ace"
//     ""  a  c  e
// ""   0  0  0  0
//  a   0  1  1  1
//  b   0  1  1  1
//  c   0  1  2  2
//  d   0  1  2  2
//  e   0  1  2  3
// Result: dp[5][3] = 3 ✓

// Time: O(m*n), Space: O(m*n)
// Space optimization: O(min(m,n)) using rolling row

public int lcsSpaceOpt(String s1, String s2) {
    if (s1.length() < s2.length()) return lcsSpaceOpt(s2, s1);  // ensure s2 is shorter
    int m = s1.length(), n = s2.length();
    int[] dp = new int[n + 1];
    
    for (int i = 1; i <= m; i++) {
        int prev = 0;  // stores dp[i-1][j-1]
        for (int j = 1; j <= n; j++) {
            int temp = dp[j];  // save dp[i-1][j] before overwriting
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) dp[j] = prev + 1;
            else dp[j] = Math.max(dp[j], dp[j - 1]);
            prev = temp;
        }
    }
    return dp[n];
}
```

---

## 10.5 — Problem: Edit Distance (LeetCode 72)

### Concept Explanation

**Levenshtein distance:** Minimum number of single-character operations (insert, delete, replace) to transform one string to another.
"horse" → "ros": replace h→r, delete r, delete e = 3 operations

### Deep Understanding

```
dp[i][j] = min operations to convert s1[0..i-1] to s2[0..j-1]

Base cases:
  dp[i][0] = i  (delete all i chars of s1)
  dp[0][j] = j  (insert all j chars of s2)

Recurrence:
  If s1[i-1] == s2[j-1]:   dp[i][j] = dp[i-1][j-1]  (no operation needed)
  Else: dp[i][j] = 1 + min(
    dp[i-1][j],    // delete from s1
    dp[i][j-1],    // insert into s1 (= delete from s2)
    dp[i-1][j-1]   // replace
  )
```

```java
public int minDistance(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    // Base cases
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];  // chars match, no op
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],   // replace
                               Math.min(dp[i - 1][j],         // delete from s1
                                        dp[i][j - 1]));        // insert into s1
            }
        }
    }
    
    return dp[m][n];
}

// Trace: "horse" → "ros"
//     ""  r  o  s
// ""   0  1  2  3
//  h   1  1  2  3
//  o   2  2  1  2
//  r   3  2  2  2
//  s   4  3  3  2
//  e   5  4  4  3
// Result: 3 ✓

// Time: O(m*n), Space: O(m*n) → can optimize to O(n)
```

---

## 10.6 — Problem: Longest Palindromic Subsequence

```java
// LCS of s with its reverse = LPS
public int longestPalindromeSubseq(String s) {
    String rev = new StringBuilder(s).reverse().toString();
    return longestCommonSubsequence(s, rev);
}
// "bbbab" → reverse = "babbb"
// LCS of "bbbab" and "babbb" = "bbbb" → length 4

// Or direct DP:
// dp[i][j] = LPS of s[i..j]
public int lpsDirect(String s) {
    int n = s.length();
    int[][] dp = new int[n][n];
    
    // Single characters are palindromes of length 1
    for (int i = 0; i < n; i++) dp[i][i] = 1;
    
    // Fill for increasing lengths
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            if (s.charAt(i) == s.charAt(j)) {
                dp[i][j] = dp[i + 1][j - 1] + 2;
            } else {
                dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[0][n - 1];
}
```

---

## 10.7 — Interview Patterns Master Reference

### The 8 Core Patterns

**Pattern 1: Frequency Map**
```java
// Use when: anagram, duplicate detection, frequency counting
int[] freq = new int[26];
freq[c - 'a']++;
```

**Pattern 2: Sliding Window — Maximum/Minimum**
```java
// Use when: longest/shortest substring with property X
int left = 0;
for (int right = 0; right < n; right++) {
    // add s[right]
    while (invalid) { /* remove s[left]; left++; */ }
    // update answer
}
```

**Pattern 3: Two Pointers**
```java
// Use when: palindrome, symmetry, pair problems
int left = 0, right = n - 1;
while (left < right) {
    // process and move
}
```

**Pattern 4: Stack Parsing**
```java
// Use when: nested structures, brackets, decode
Deque<T> stack = new ArrayDeque<>();
for (char c : s.toCharArray()) {
    if (opening) stack.push(state);
    if (closing) { int k = stack.pop(); /* combine */ }
}
```

**Pattern 5: KMP / Pattern Matching**
```java
// Use when: find pattern in text, prefix-suffix overlap
int[] lps = buildLPS(pattern);
// Use lps to skip redundant comparisons
```

**Pattern 6: DP — Subsequence/Substring**
```java
// Use when: transformation, counting, optimal sequence
int[][] dp = new int[m+1][n+1];
// dp[i][j] represents answer for s1[0..i-1] and s2[0..j-1]
```

**Pattern 7: Expand Around Center**
```java
// Use when: palindrome substring/substrings
for (int i = 0; i < n; i++) {
    expand(s, i, i);     // odd
    expand(s, i, i+1);  // even
}
```

**Pattern 8: Hashing**
```java
// Use when: duplicate substring detection, multiple comparisons
// Rolling hash or encode substring as key
```

---

## 10.8 — Mock Interview Problems

### Problem 1: Valid Anagram — 5 minutes target

```java
// Already covered — key pattern: freq array, both same length
```

### Problem 2: Longest Substring Without Repeating Characters — 8 minutes

```java
// Already covered — key pattern: sliding window + lastSeen HashMap
```

### Problem 3: Minimum Window Substring — 15 minutes

```java
// Already covered — key: formed/required tracking
```

### Problem 4: strStr() / Find Needle — 10 minutes (with KMP)

```java
// Already covered — KMP with LPS
```

### Problem 5: Word Break — 10 minutes

```java
// Already covered — DP with dictionary set
```

---

## 10.9 — Additional Important Topics

### Isomorphic Strings (LeetCode 205)

Two strings are isomorphic if characters can be mapped one-to-one.
"egg" and "add" → isomorphic (e→a, g→d)
"foo" and "bar" → not isomorphic (o→a AND o→r — contradiction)

```java
public boolean isIsomorphic(String s, String t) {
    int[] sMap = new int[256];  // s char → last seen index + 1
    int[] tMap = new int[256];  // t char → last seen index + 1
    
    for (int i = 0; i < s.length(); i++) {
        if (sMap[s.charAt(i)] != tMap[t.charAt(i)]) return false;
        sMap[s.charAt(i)] = i + 1;  // +1 so 0 means "not seen"
        tMap[t.charAt(i)] = i + 1;
    }
    return true;
}
// "egg" "add":
// i=0: s['e']=0, t['a']=0 → equal, set both to 1
// i=1: s['g']=0, t['d']=0 → equal, set both to 2
// i=2: s['g']=2, t['d']=2 → equal ✓
// Result: true

// "foo" "bar":
// i=0: s['f']=0, t['b']=0 → equal, set 1
// i=1: s['o']=0, t['a']=0 → equal, set 2
// i=2: s['o']=2, t['r']=0 → NOT equal → false
```

### Reverse Words in a String (LeetCode 151)

```java
public String reverseWords(String s) {
    String[] words = s.trim().split("\\s+");  // split on any whitespace
    StringBuilder sb = new StringBuilder();
    
    for (int i = words.length - 1; i >= 0; i--) {
        sb.append(words[i]);
        if (i > 0) sb.append(' ');
    }
    return sb.toString();
}
// "  hello world  " → "world hello"

// In-place O(1) space approach:
// 1. Reverse entire string
// 2. Reverse each word individually
// 3. Remove extra spaces
```

### Compare Version Numbers (LeetCode 165)

```java
public int compareVersion(String version1, String version2) {
    String[] v1 = version1.split("\\.");
    String[] v2 = version2.split("\\.");
    int n = Math.max(v1.length, v2.length);
    
    for (int i = 0; i < n; i++) {
        int num1 = i < v1.length ? Integer.parseInt(v1[i]) : 0;
        int num2 = i < v2.length ? Integer.parseInt(v2[i]) : 0;
        
        if (num1 < num2) return -1;
        if (num1 > num2) return 1;
    }
    return 0;
}
// "1.01" vs "1.001": split → ["1","01"] vs ["1","001"]
// Compare 1=1, then 01=1 vs 001=1 → equal → 0
```

### Zigzag Conversion (LeetCode 6)

```java
// "PAYPALISHIRING" with numRows=3:
// P   A   H   N
// A P L S I I G
// Y   I   R
// Read row by row: "PAHNAPLSIIGYIR"

public String convert(String s, int numRows) {
    if (numRows == 1 || numRows >= s.length()) return s;
    
    StringBuilder[] rows = new StringBuilder[numRows];
    for (int i = 0; i < numRows; i++) rows[i] = new StringBuilder();
    
    int curRow = 0;
    boolean goingDown = false;
    
    for (char c : s.toCharArray()) {
        rows[curRow].append(c);
        if (curRow == 0 || curRow == numRows - 1) goingDown = !goingDown;
        curRow += goingDown ? 1 : -1;
    }
    
    StringBuilder sb = new StringBuilder();
    for (StringBuilder row : rows) sb.append(row);
    return sb.toString();
}
// Time: O(n), Space: O(n)
```

---

## 10.10 — Advanced: Trie Basics

### Concept Explanation

A **Trie** (prefix tree) is a tree data structure for storing strings where common prefixes share the same path.

**Why it exists:** O(k) search for string of length k, regardless of number of strings stored. Better than HashMap for prefix-based operations.

**Real-world:** Autocomplete, spell check, IP routing tables, T9 phone keyboards.

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd = false;
}

class Trie {
    TrieNode root = new TrieNode();
    
    // Insert word — O(k) where k = word length
    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEnd = true;
    }
    
    // Search for exact word — O(k)
    public boolean search(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return false;
            node = node.children[idx];
        }
        return node.isEnd;
    }
    
    // Check if any word starts with this prefix — O(k)
    public boolean startsWith(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return false;
            node = node.children[idx];
        }
        return true;  // exists as a prefix (regardless of isEnd)
    }
}

// Usage with Word Break using Trie:
// Insert all dictionary words into Trie
// For each position in s, traverse Trie to check valid words
// Slightly faster than HashSet for long prefix checks
```

---

## 10.11 — Manacher's Algorithm (Optional Advanced)

### Concept Explanation

**Manacher's algorithm** finds all palindromic substrings in O(n) time — faster than the O(n²) expand-around-center.

Rarely asked in interviews but good to know for competitive programming.

```java
public int longestPalindromeManacher(String s) {
    // Transform: "abc" → "^#a#b#c#$"
    // This handles both odd and even palindromes uniformly
    String t = "#";
    for (char c : s.toCharArray()) t += c + "#";
    t = "^" + t + "$";
    
    int n = t.length();
    int[] p = new int[n];  // p[i] = radius of palindrome centered at i
    int center = 0, right = 0;
    
    for (int i = 1; i < n - 1; i++) {
        int mirror = 2 * center - i;
        
        if (i < right) {
            p[i] = Math.min(right - i, p[mirror]);
        }
        
        // Expand around i
        while (t.charAt(i + p[i] + 1) == t.charAt(i - p[i] - 1)) {
            p[i]++;
        }
        
        // Update center/right boundary
        if (i + p[i] > right) {
            center = i;
            right = i + p[i];
        }
    }
    
    // Find maximum
    int maxLen = 0;
    for (int len : p) maxLen = Math.max(maxLen, len);
    return maxLen;
}
// Transform maps palindrome in transformed string to original:
// A palindrome of radius r in transformed = palindrome of length r in original
// The '#' and '^'/'$' sentinels prevent boundary checks
```

---

## 10.12 — Suffix Arrays (Overview)

### Concept Explanation

A **suffix array** is a sorted array of all suffixes of a string.

For "banana":
```
Suffixes:       Sorted:
banana          a         → index 5
anana           ana       → index 3
nana            anana     → index 1
ana             banana    → index 0
na              na        → index 4
a               nana      → index 2

Suffix Array: [5, 3, 1, 0, 4, 2]
```

**Use cases:**
- Longest repeated substring — O(n log n) construction + O(n) query
- Number of distinct substrings
- Pattern matching — O(m log n) with suffix array
- Longest Common Prefix (LCP) array

**Construction:** O(n log²n) naive, O(n log n) prefix doubling, O(n) SA-IS (advanced). Usually beyond interview scope but useful for competitive programming.

---

## 10.13 — Lexicographical Ordering

```java
// Natural string comparison is lexicographic in Java
"abc".compareTo("abd")  // negative (a<d at index 2)
"abc".compareTo("abc")  // 0 (equal)
"abd".compareTo("abc")  // positive

// Sort strings lexicographically
String[] arr = {"banana", "apple", "cherry"};
Arrays.sort(arr);  // ["apple", "banana", "cherry"]

// Lexicographically smallest/largest rotation
// For finding: concatenate with itself and search for minimum/maximum

// Custom comparator for sorting strings by length, then lexicographically
Arrays.sort(arr, (a, b) -> a.length() != b.length() 
    ? a.length() - b.length() 
    : a.compareTo(b));

// Important: "1" < "10" < "2" lexicographically (string comparison)
// But numerically: 1 < 2 < 10
// This matters for version comparison, numeric sort of string numbers
```

---

## Session 10 — Quick Revision Summary

```
KEY TAKEAWAYS:
1. DP on strings: define state as dp[i] or dp[i][j], find recurrence
2. Word Break: dp[i] = OR over all j < i of (dp[j] && word(j,i) in dict)
3. Decode Ways: dp[i] = dp[i-1] if valid 1-digit + dp[i-2] if valid 2-digit
4. LCS: dp[i][j] = dp[i-1][j-1]+1 if match, else max(dp[i-1][j], dp[i][j-1])
5. Edit Distance: dp[i][j] = dp[i-1][j-1] if match, else 1+min(3 operations)

PATTERN LOOKUP:
"can this string be broken into dict words?" → Word Break DP
"how many decodings?" → Decode Ways DP
"longest common subsequence?" → LCS 2D DP
"minimum transformations?" → Edit Distance DP
"longest palindromic subsequence?" → LCS with reverse

MEMORY AIDS:
LCS: match → diagonal+1; no match → max(up, left)
Edit: match → diagonal; no match → 1+min(diagonal, up, left)
LPS: match → diagonal+2; no match → max(down, right)
```

---

# ADDITIONAL IMPORTANT TOPICS

---

## A.1 — String Hashing for Competitive Programming

```java
// Double hashing to minimize collision probability
class StringHash {
    private final long MOD1 = 1_000_000_007L, BASE1 = 131L;
    private final long MOD2 = 998_244_353L, BASE2 = 137L;
    
    long[] h1, h2;  // prefix hash arrays
    long[] p1, p2;  // power arrays
    
    StringHash(String s) {
        int n = s.length();
        h1 = new long[n + 1]; h2 = new long[n + 1];
        p1 = new long[n + 1]; p2 = new long[n + 1];
        p1[0] = p2[0] = 1;
        
        for (int i = 0; i < n; i++) {
            h1[i+1] = (h1[i] * BASE1 + s.charAt(i)) % MOD1;
            h2[i+1] = (h2[i] * BASE2 + s.charAt(i)) % MOD2;
            p1[i+1] = p1[i] * BASE1 % MOD1;
            p2[i+1] = p2[i] * BASE2 % MOD2;
        }
    }
    
    // Get hash of s[l..r] (0-indexed, inclusive)
    long[] getHash(int l, int r) {
        long hash1 = (h1[r+1] - h1[l] * p1[r-l+1] % MOD1 + MOD1) % MOD1;
        long hash2 = (h2[r+1] - h2[l] * p2[r-l+1] % MOD2 + MOD2) % MOD2;
        return new long[]{hash1, hash2};
    }
    
    // Compare s[l1..r1] with s[l2..r2]
    boolean equal(int l1, int r1, int l2, int r2) {
        long[] h1 = getHash(l1, r1);
        long[] h2 = getHash(l2, r2);
        return h1[0] == h2[0] && h1[1] == h2[1];
    }
}
```

---

## A.2 — Complete Problem Solutions for Easy/Frequently Asked

### Reverse Words in String II (In-place)

```java
// Reverse individual words then reverse whole string (or vice versa)
public void reverseWordsInPlace(char[] s) {
    // Step 1: Reverse entire array
    reverse(s, 0, s.length - 1);
    
    // Step 2: Reverse each word
    int start = 0;
    for (int end = 0; end <= s.length; end++) {
        if (end == s.length || s[end] == ' ') {
            reverse(s, start, end - 1);
            start = end + 1;
        }
    }
}

private void reverse(char[] s, int l, int r) {
    while (l < r) {
        char tmp = s[l]; s[l] = s[r]; s[r] = tmp;
        l++; r--;
    }
}
```

### Longest Common Prefix (LeetCode 14)

```java
public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        // Shrink prefix until it matches start of strs[i]
        while (!strs[i].startsWith(prefix)) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.isEmpty()) return "";
        }
    }
    return prefix;
}
// ["flower","flow","flight"] → "fl"
// Vertical scanning approach is more efficient but this is clean
```

---

# FINAL COMPLETE REVISION SHEET

---

## MASTER REFERENCE — ALL PATTERNS

```
PATTERN 1: FREQUENCY MAP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: anagram, duplicate, most/least frequent char
int[] freq = new int[26]; freq[c-'a']++;
Compare: Arrays.equals(freq1, freq2)

PATTERN 2: TWO POINTERS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: palindrome, reverse, meeting-in-middle
int l=0, r=n-1; while(l<r) { process; l++; r--; }

PATTERN 3: SLIDING WINDOW (FIXED)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: fixed-size window maximum/sum
Init first window, then: add[i], remove[i-k], update answer

PATTERN 4: SLIDING WINDOW (VARIABLE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: longest/shortest with constraint
for right: add; while invalid: remove left; update answer

PATTERN 5: EXPAND AROUND CENTER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: palindromic substrings
for each center (odd + even): expand while match

PATTERN 6: KMP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: pattern matching in text
Build LPS, then search using j=lps[j-1] on mismatch

PATTERN 7: ROLLING HASH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: duplicate substrings, multi-pattern matching
hash = (hash - left*power) * base + right; verify on match

PATTERN 8: STACK PARSING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: brackets, nested decoding, expression parsing
Push on open, pop and combine on close

PATTERN 9: DP ON STRINGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: optimal transformations, counting ways
LCS: dp[i][j] based on match/no-match
Edit Distance: 1+min(replace, delete, insert)

PATTERN 10: Z / PREFIX FUNCTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Use: advanced pattern matching, prefix/suffix analysis
Z[i]: length of prefix starting at i
```

---

## COMPLEXITY QUICK REFERENCE

```
Algorithm              | Time        | Space
-----------------------+-------------+--------
String concatenation   | O(n²)       | O(n²)
StringBuilder append   | O(n) total  | O(n)
charAt(i)              | O(1)        | O(1)
substring(i,j)         | O(j-i)      | O(j-i)
Frequency array        | O(n)        | O(1)
Frequency HashMap      | O(n)        | O(n)
Sorting for anagram    | O(n log n)  | O(n)
Two pointers           | O(n)        | O(1)
Sliding window         | O(n)        | O(1) to O(n)
KMP search             | O(n+m)      | O(m)
Rabin-Karp             | O(n+m) avg  | O(1)
Z algorithm            | O(n)        | O(n)
LCS (2D DP)            | O(n²)       | O(n²)
Edit Distance          | O(n²)       | O(n²)
Manacher's             | O(n)        | O(n)
Trie insert/search     | O(k)        | O(total chars)
```

---

## COMMON INTERVIEW QUESTIONS WITH ANSWERS

### Q1: What is the time complexity of string concatenation in a loop?

**Answer:** O(n²). Each concatenation creates a new string of increasing length. Total work: 0+1+2+...+(n-1) = n(n-1)/2 = O(n²). Always use StringBuilder which is O(n).

### Q2: Why is String immutable in Java?

**Answer:** Three reasons: (1) Thread safety — immutable objects can be shared without synchronization. (2) Security — strings used as keys (passwords, file paths) can't be altered. (3) String pool optimization — the JVM can safely intern and reuse string literals.

### Q3: How do you detect all anagrams of a pattern in a text?

**Answer:** Use a fixed sliding window of size pattern.length(). Maintain frequency arrays for pattern and current window. Compare arrays at each position. Time O(n), Space O(1). See LeetCode 438.

### Q4: Explain how KMP achieves O(n+m) time.

**Answer:** KMP preprocesses the pattern to build an LPS array in O(m). The LPS[i] stores the length of the longest proper prefix of pattern[0..i] that is also a suffix. During search, when a mismatch occurs at position j, instead of resetting j to 0, we set j = LPS[j-1]. This allows skipping previously matched characters. The key insight: the text pointer i never goes backward — it only advances. Total comparisons: O(n+m).

### Q5: What's the difference between LCS and Edit Distance?

**Answer:** LCS (Longest Common Subsequence) finds the longest sequence of characters appearing in the same order in both strings — no modifications needed, just inclusion/exclusion. Edit Distance finds the minimum operations (insert, delete, replace) to transform one string to another. LCS is about what's preserved; Edit Distance is about what must change.

### Q6: How do you find the longest palindromic substring in O(n²)?

**Answer:** Expand around center. For each position i (0 to n-1), expand outward checking odd-length centers (expand from i,i) and even-length centers (expand from i, i+1). Keep track of the longest palindrome found. Total O(n) centers × O(n) expansion = O(n²). Space O(1).

### Q7: When would you use rolling hash over KMP?

**Answer:** Rolling hash when: (1) matching multiple patterns at once, (2) in competitive programming (faster to code), (3) finding duplicate substrings (Rabin-Karp with binary search). KMP when: (1) guaranteed O(n+m) worst case (rolling hash is O(n*m) worst case due to collision verification), (2) interview setting (KMP is more "classically" expected).

### Q8: How does Minimum Window Substring work?

**Answer:** Use two pointers (left, right) with a "formed" counter. Expand right until all characters of t are covered (formed == required). Then shrink left to minimize window while keeping formed == required, updating minimum each time. Use have[] and need[] frequency maps, increment formed when have[c] == need[c]. Time O(|s|+|t|).

### Q9: What is the Z-array and how is it different from the LPS array?

**Answer:** Z[i] = length of longest substring starting at index i that is also a prefix of the entire string. LPS[i] = length of longest proper prefix of pattern[0..i] that is also a suffix. Z answers "how long does the substring at position i match the string's own prefix?" LPS answers "for a mismatch at position i, how many characters can we skip?" Both enable O(n) pattern matching but with different conceptual approaches.

### Q10: Explain the DP recurrence for Word Break.

**Answer:** dp[i] = true if s[0..i-1] can be completely segmented using dictionary words. dp[0] = true (empty string). For each i: dp[i] = OR over all j (0 to i-1) of (dp[j] AND s[j..i-1] is in dictionary). Intuition: for each end position i, try every possible last word ending at i. If the string before the last word is also breakable (dp[j]), then dp[i] is true.

---

## MINI PROJECTS

### Project 1: Text Utility Toolkit

```java
public class TextUtilityToolkit {
    // Reverse string
    public String reverse(String s) {
        return new StringBuilder(s).reverse().toString();
    }
    
    // Check palindrome (ignoring non-alphanumeric)
    public boolean isPalindrome(String s) {
        int l = 0, r = s.length() - 1;
        while (l < r) {
            while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
            while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
            if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r))) return false;
            l++; r--;
        }
        return true;
    }
    
    // Count words
    public int wordCount(String s) {
        return s.trim().isEmpty() ? 0 : s.trim().split("\\s+").length;
    }
    
    // Case conversion
    public String toToggleCase(String s) {
        char[] chars = s.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            chars[i] = Character.isUpperCase(chars[i]) 
                ? Character.toLowerCase(chars[i]) 
                : Character.toUpperCase(chars[i]);
        }
        return new String(chars);
    }
    
    // Character frequency
    public Map<Character, Integer> frequency(String s) {
        Map<Character, Integer> map = new LinkedHashMap<>();
        for (char c : s.toCharArray()) map.merge(c, 1, Integer::sum);
        return map;
    }
    
    // Compress (run-length encoding)
    public String compress(String s) {
        StringBuilder sb = new StringBuilder();
        int i = 0;
        while (i < s.length()) {
            char c = s.charAt(i);
            int cnt = 0;
            while (i < s.length() && s.charAt(i) == c) { cnt++; i++; }
            if (cnt > 1) sb.append(cnt);
            sb.append(c);
        }
        return sb.length() < s.length() ? sb.toString() : s;
    }
}
```

### Project 2: Anagram Grouping Engine

```java
public class AnagramEngine {
    // Group all anagrams together
    public Map<String, List<String>> groupAnagrams(String[] words) {
        Map<String, List<String>> groups = new HashMap<>();
        for (String word : words) {
            char[] chars = word.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);
            groups.computeIfAbsent(key, k -> new ArrayList<>()).add(word);
        }
        return groups;
    }
    
    // Find all anagram pairs
    public List<int[]> findAnagramPairs(String[] words) {
        Map<String, Integer> keyToIndex = new HashMap<>();
        List<int[]> pairs = new ArrayList<>();
        
        for (int i = 0; i < words.length; i++) {
            char[] c = words[i].toCharArray(); Arrays.sort(c);
            String key = new String(c);
            if (keyToIndex.containsKey(key)) pairs.add(new int[]{keyToIndex.get(key), i});
            keyToIndex.put(key, i);
        }
        return pairs;
    }
}
```

### Project 3: Mini Search Engine with KMP

```java
public class SearchEngine {
    private String[] documents;
    
    public SearchEngine(String[] docs) { this.documents = docs; }
    
    // Find all documents containing the pattern
    public List<Integer> search(String pattern) {
        List<Integer> results = new ArrayList<>();
        int[] lps = buildLPS(pattern);
        
        for (int docIdx = 0; docIdx < documents.length; docIdx++) {
            if (kmpSearch(documents[docIdx], pattern, lps)) {
                results.add(docIdx);
            }
        }
        return results;
    }
    
    private boolean kmpSearch(String text, String pattern, int[] lps) {
        int i = 0, j = 0;
        while (i < text.length()) {
            if (text.charAt(i) == pattern.charAt(j)) { i++; j++; }
            if (j == pattern.length()) return true;
            else if (i < text.length() && text.charAt(i) != pattern.charAt(j)) {
                if (j != 0) j = lps[j - 1]; else i++;
            }
        }
        return false;
    }
    
    private int[] buildLPS(String p) {
        int[] lps = new int[p.length()]; int len = 0, i = 1;
        while (i < p.length()) {
            if (p.charAt(i) == p.charAt(len)) lps[i++] = ++len;
            else if (len != 0) len = lps[len - 1];
            else lps[i++] = 0;
        }
        return lps;
    }
    
    // Find typo-tolerant matches using Edit Distance
    public List<Integer> fuzzySearch(String query, int maxDistance) {
        List<Integer> results = new ArrayList<>();
        for (int i = 0; i < documents.length; i++) {
            if (editDistance(documents[i], query) <= maxDistance) results.add(i);
        }
        return results;
    }
    
    private int editDistance(String a, String b) {
        int m = a.length(), n = b.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 0; i <= m; i++) dp[i][0] = i;
        for (int j = 0; j <= n; j++) dp[0][j] = j;
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                dp[i][j] = a.charAt(i-1) == b.charAt(j-1) ? dp[i-1][j-1]
                    : 1 + Math.min(dp[i-1][j-1], Math.min(dp[i-1][j], dp[i][j-1]));
        return dp[m][n];
    }
}
```

---

## ADVANCED CHALLENGES

### Challenge 1: Shortest Superstring

Given an array of strings, find the shortest string that contains each string as a substring. (DP on bitmasks + string overlap)

### Challenge 2: Palindrome Partitioning II (LeetCode 132)

Find minimum number of cuts needed for palindrome partitioning. Requires DP + palindrome detection.

### Challenge 3: Word Ladder II (LeetCode 126)

Find all shortest transformation sequences from beginWord to endWord. BFS + string manipulation.

### Challenge 4: Distinct Subsequences (LeetCode 115)

Count how many distinct subsequences of s equal t. 2D DP.

### Challenge 5: Regular Expression Matching (LeetCode 10)

Implement regex with '.' and '*'. Complex 2D DP.

---

## MASTERY CHECKLIST

```
FOUNDATIONS
□ Understand String immutability and memory model
□ Know when to use String vs StringBuilder vs StringBuffer
□ Memorize ASCII values for 'a', 'A', '0'
□ Implement char frequency array from memory
□ Can implement basic string operations without library

CORE PATTERNS
□ Two-pointer palindrome in < 2 minutes
□ Sliding window (fixed) template memorized
□ Sliding window (variable) template memorized
□ Frequency map comparison for anagram check
□ Group anagrams using sorted key

PALINDROMES
□ Expand-around-center for both odd and even
□ Valid Palindrome II with two-branch recursion
□ Count palindromic substrings in O(n²)
□ Longest palindromic subsequence (LCS with reverse)

PATTERN MATCHING
□ Build LPS array manually for any pattern
□ Trace KMP search step by step
□ Implement complete KMP from memory
□ Understand rolling hash formula and collision handling
□ Build Z-array and use for pattern matching

ADVANCED SLIDING WINDOW
□ Minimum Window Substring with formed/required tracking
□ Character Replacement: windowLen - maxFreq <= k
□ Explain why maxFreq doesn't need to decrease on shrink

PARSING
□ Valid Parentheses with stack in < 3 minutes
□ Decode String with two stacks (count stack + string stack)
□ String to Integer with sign/overflow handling
□ Generate Parentheses with backtracking

DP ON STRINGS
□ Word Break: dp[i] = any j where dp[j] && word(j,i) in dict
□ Decode Ways: one-digit + two-digit cases
□ LCS: match → diagonal+1, no match → max(up, left)
□ Edit Distance: match → diagonal, no match → 1+min(3)
□ LPS: match → diagonal+2, no match → max(down, right)

ADVANCED TOPICS
□ Trie insert/search/startsWith implementation
□ KMP for Repeated Substring Pattern detection
□ Rabin-Karp for Repeated DNA Sequences
□ Z-algorithm and comparison with KMP

PROBLEM SOLVING
□ Can solve Valid Anagram in < 3 minutes
□ Can solve Longest Substring WRR in < 5 minutes
□ Can solve Minimum Window Substring in < 15 minutes
□ Can solve Word Break in < 10 minutes
□ Can solve Edit Distance in < 12 minutes
```

---

## RECOMMENDED NEXT LEARNING PATH

```
After mastering this curriculum:

IMMEDIATE NEXT (1-2 weeks):
├── Arrays & Hashing (overlaps heavily — review together)
├── Two Pointers on Arrays (generalize from strings)
└── Stack problems (build on parsing foundations)

SHORT TERM (2-4 weeks):
├── Tree problems (apply DFS/BFS — serialization uses strings)
├── Graph problems (word ladder, word transformation)
└── Binary Search (combined with string hashing for substring problems)

MEDIUM TERM (1-2 months):
├── Trie deep dive (Word Search II, Auto-complete, prefix problems)
├── Suffix Arrays and LCP arrays
├── Advanced DP (Regex matching, Wildcard matching, Distinct Subsequences)
└── Advanced bit manipulation (use with string encoding)

COMPETITIVE PROGRAMMING PATH:
├── Z-function and KMP applications
├── Suffix Automaton (most powerful string structure)
├── Aho-Corasick (multi-pattern matching)
├── Palindromic tree (Eertree)
└── Suffix Array + LCP: all substring problems

INTERVIEW PREPARATION:
├── LeetCode Top 75 (string subset first)
├── Blind 75 string problems
├── NeetCode 150 string problems
└── Company-specific string questions (Google, Facebook, Amazon)

TOOLS & RESOURCES:
├── LeetCode (problems + discuss)
├── NeetCode.io (video explanations)
├── CP-Algorithms.com (KMP, Z, Hashing, Suffix Array)
├── CSES Problem Set (string section for CP)
└── Codeforces (string rounds for competitive practice)

80/20 RULE — IF TIME LIMITED:
Master these 8 and you cover ~80% of string interviews:
1. Frequency maps (anagram, unique chars)
2. Sliding window (longest/shortest substring)
3. Two pointers (palindrome, reverse)
4. Stack parsing (brackets, decode, expression)
5. KMP basics (strStr, repeated pattern)
6. DP on strings (word break, edit distance)
7. Expand-around-center palindrome
8. HashMap/Set for tracking (window, seen chars)
```

---

## THE FINAL 12 PROBLEMS — SOLVE THESE COLD

```
Level 1 (Should solve in < 5 min):
1. Valid Anagram (LeetCode 242)
2. Valid Palindrome (LeetCode 125)
3. Reverse String (LeetCode 344)
4. First Unique Character (LeetCode 387)

Level 2 (Should solve in < 10 min):
5. Group Anagrams (LeetCode 49)
6. Longest Substring Without Repeating Chars (LeetCode 3)
7. Valid Parentheses (LeetCode 20)
8. Longest Palindromic Substring (LeetCode 5)

Level 3 (Should solve in < 15 min):
9. Minimum Window Substring (LeetCode 76)
10. Decode String (LeetCode 394)
11. Implement strStr() with KMP (LeetCode 28)
12. Word Break (LeetCode 139)

Bonus (Advanced):
13. Edit Distance (LeetCode 72)
14. Decode Ways (LeetCode 91)
15. Repeated DNA Sequences (LeetCode 187)

If you can solve all 12 core problems confidently, explain your approach,
discuss time/space tradeoffs, and suggest follow-up optimizations —
you are interview-ready for string rounds at most top companies.
```