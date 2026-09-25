# DSA Pattern Matching & Advanced String Processing: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Pattern matching is the process of finding occurrences of a pattern string within a larger text string. Advanced string processing algorithms optimize this search to achieve linear or near-linear time complexity, often using preprocessing techniques that exploit the structure of the pattern.

**Technical Definition**
Given a text \( T \) of length \( n \) and a pattern \( P \) of length \( m \), pattern matching finds all indices \( i \) such that \( T[i..i+m-1] = P \). Advanced algorithms preprocess \( P \) (or both \( P \) and \( T \)) to avoid redundant comparisons, achieving \( O(n + m) \) or \( O(n \cdot m) \) worst-case time depending on the algorithm.

**Beginner-Friendly Explanation**
Think of searching for a word in a book. The naive way is to check every position, letter by letter, restarting from scratch on every mismatch. Advanced algorithms are like having a smart bookmark that tells you "you already matched these letters, so skip ahead"—saving enormous time on large texts.

### Key Characteristics

- **Preprocessing Overhead**: Advanced algorithms spend time building auxiliary structures (LPS array, Z-array, automaton) before searching
- **Linear-Time Guarantees**: KMP, Z-algorithm, and Manacher's achieve \( O(n + m) \)
- **Multi-Pattern Support**: Aho-Corasick searches for many patterns simultaneously
- **Hash-Based Approaches**: Rabin-Karp uses rolling hashes for average-case linear time
- **Real-World Dominance**: Boyer-Moore is often fastest in practice due to right-to-left scanning and large shifts

### Prerequisites

- String fundamentals (indexing, traversal, comparison)
- Basic algorithm analysis (Big O notation)
- Familiarity with arrays and hash functions

### Related Programming Areas

- **Text Editors**: Search and replace functionality
- **Bioinformatics**: DNA sequence matching
- **Network Security**: Intrusion detection (Deep Packet Inspection)
- **Information Retrieval**: Search engines and spam filters

### Core Concepts / Features

| Algorithm | Time Complexity | Space Complexity | Best Use Case |
|-----------|-----------------|------------------|---------------|
| Brute-Force | \( O(n \cdot m) \) | \( O(1) \) | Very small patterns |
| KMP | \( O(n + m) \) | \( O(m) \) | Binary strings, guaranteed linear |
| Z-Algorithm | \( O(n + m) \) | \( O(n + m) \) | Simple linear-time alternative to KMP |
| Rabin-Karp | \( O(n + m) \) avg | \( O(1) \) | Multiple pattern search, plagiarism detection |
| Boyer-Moore | \( O(n \cdot m) \) worst, sublinear typical | \( O(m + \sigma) \) | Large alphabets, text editors |
| Aho-Corasick | \( O(n + m + z) \) | \( O(m \cdot \sigma) \) | Multi-pattern matching |
| Manacher's | \( O(n) \) | \( O(n) \) | Longest palindromic substring |

---

## 1. Brute-Force Pattern Matching and Its Limitations

### Definitions

**Core Definition**
Brute-force pattern matching checks for the pattern at every possible position in the text, comparing character by character until a mismatch or complete match is found.

**Technical Definition**
For each shift \( s \) from \( 0 \) to \( n - m \), compare \( P[0..m-1] \) with \( T[s..s+m-1] \). If all \( m \) characters match, report a match at position \( s \).

**Beginner-Friendly Explanation**
Checking every position in a book to see if the word "cat" appears there, letter by letter, then moving one position right and starting over.

### Purposes (All begin with "To")

- **To** provide a baseline for comparing advanced algorithms
- **To** solve pattern matching for very small patterns where preprocessing overhead exceeds benefit
- **To** understand the fundamental problem before optimization

### Syntax Rules and Structure

**General Syntax**
```
for s from 0 to n - m:
    j = 0
    while j < m and T[s + j] == P[j]:
        j += 1
    if j == m:
        report match at s
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| Outer loop | Shift position | 0 to n-m |
| Inner loop | Character comparison | 0 to m-1 |
| Match condition | All m characters match | j == m |

**Syntax Rules**

- Shifts range from 0 to n-m (inclusive)
- Each shift restarts comparison from the beginning of the pattern
- Worst case occurs with highly repetitive text and pattern (e.g., "aaaaa" in "aaaaaaaa")

**Constraints and Limitations**

- Worst-case time \( O(n \cdot m) \) for patterns like "aaaa" in "aaaa...a"
- Typical case is often \( O(n) \) for random text, but worst case is catastrophic
- No preprocessing means no ability to skip redundant comparisons

### Annotated Code Examples

**Example 1: Brute-Force Implementation (Python)**
```python
def brute_force_search(text, pattern):
    """
    Naive pattern matching: check every position.
    Returns list of starting indices where pattern occurs.
    """
    n, m = len(text), len(pattern)
    matches = []
    
    # Try every possible shift
    for s in range(n - m + 1):
        j = 0
        # Compare character by character
        while j < m and text[s + j] == pattern[j]:
            j += 1
        
        # All characters matched
        if j == m:
            matches.append(s)
    
    return matches

# Test
text = "AABAACAADAABAABA"
pattern = "AABA"
print(brute_force_search(text, pattern))  # [0, 9, 12]
```

**Expected Output:**
```
[0, 9, 12]
```

**Why This Output Occurs**: The pattern "AABA" appears at indices 0 (AABA), 9 (AABA), and 12 (AABA) in the text. The brute-force algorithm checks each position, comparing up to 4 characters per shift.

### Real-World Cases

- **Small Pattern Search**: When \( m \) is tiny (< 10 characters)
- **One-Time Searches**: When preprocessing cost exceeds search benefit
- **Educational Baseline**: Teaching the fundamentals of pattern matching

### References

- Princeton University - Brute-Force Analysis - https://www.cs.princeton.edu/courses/archive/spring02/cs226/lectures/string.4up.pdf

---

## 2. Knuth-Morris-Pratt (KMP) Algorithm and the LPS Array

### Definitions

**Core Definition**
KMP preprocesses the pattern to build a Longest Prefix Suffix (LPS) array that indicates how many characters can be skipped when a mismatch occurs, avoiding redundant comparisons.

**Technical Definition**
The LPS array for pattern \( P \) stores at each index \( i \) the length of the longest proper prefix of \( P[0..i] \) that is also a suffix. During search, on mismatch at pattern position \( j \), set \( j = \text{LPS}[j-1] \) instead of restarting from 0.

**Beginner-Friendly Explanation**
If you're matching "ABCABD" and fail at the last 'D', you've already matched "ABCAB". Since "AB" appears at both the start and end of "ABCAB", you can skip ahead knowing "AB" is already matched—no need to re-check those characters.

### Purposes (All begin with "To")

- **To** achieve guaranteed \( O(n + m) \) pattern matching without hash collisions
- **To** avoid re-comparing characters that are known to match
- **To** provide deterministic linear-time search for binary strings where Boyer-Moore is inefficient

### Syntax Rules and Structure

**General Syntax (Building LPS)**
```
lps = [0] * m
length = 0
i = 1

while i < m:
    if P[i] == P[length]:
        length += 1
        lps[i] = length
        i += 1
    else:
        if length != 0:
            length = lps[length - 1]
        else:
            lps[i] = 0
            i += 1
```

**General Syntax (Searching)**
```
j = 0
for i in range(n):
    while j > 0 and T[i] != P[j]:
        j = lps[j - 1]
    if T[i] == P[j]:
        j += 1
    if j == m:
        report match at i - m + 1
        j = lps[j - 1]
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `length` | Current LPS length | 0 to m-1 |
| `lps[i]` | LPS value at index i | Length of longest proper prefix = suffix |
| `j` | Pattern index during search | 0 to m-1 |
| Fallback | On mismatch | `j = lps[j-1]` |

**Syntax Rules**

- `lps[0] = 0` always (no proper prefix of length 1)
- On mismatch, fall back to `lps[length-1]` until match or length becomes 0
- During search, after a complete match, fall back to `lps[j-1]` to find overlapping matches

**Constraints and Limitations**

- Requires \( O(m) \) extra space for the LPS array
- Building LPS is \( O(m) \) time; searching is \( O(n) \) time
- For very short patterns, overhead may exceed brute-force

### Annotated Code Examples

**Example 1: KMP Implementation (Python)**
```python
def build_lps(pattern):
    """Build Longest Prefix Suffix array for KMP."""
    m = len(pattern)
    lps = [0] * m
    length = 0  # Length of previous longest prefix suffix
    i = 1
    
    while i < m:
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1
        else:
            if length != 0:
                # Fall back to previous longest prefix
                length = lps[length - 1]
            else:
                lps[i] = 0
                i += 1
    
    return lps

def kmp_search(text, pattern):
    """KMP pattern matching."""
    n, m = len(text), len(pattern)
    if m == 0:
        return []
    
    lps = build_lps(pattern)
    matches = []
    j = 0  # Index in pattern
    
    for i in range(n):
        # Fall back on mismatch
        while j > 0 and text[i] != pattern[j]:
            j = lps[j - 1]
        
        if text[i] == pattern[j]:
            j += 1
        
        # Full match found
        if j == m:
            matches.append(i - m + 1)
            j = lps[j - 1]  # Prepare for overlapping match
    
    return matches

# Test
text = "ABABDABACDABABCABAB"
pattern = "ABABCABAB"
print(f"LPS array: {build_lps(pattern)}")
print(f"Matches: {kmp_search(text, pattern)}")
```

**Expected Output:**
```
LPS array: [0, 0, 1, 2, 0, 1, 2, 3, 4]
Matches: [10]
```

**Why This Output Occurs**: The pattern "ABABCABAB" has LPS values reflecting its internal structure: "AB" at positions 2-3, and "ABAB" at positions 5-8. The match occurs at index 10 where the full pattern appears .

### Real-World Cases

- **Binary String Search**: KMP is recommended for binary alphabets where Boyer-Moore's bad character heuristic is ineffective
- **Intrusion Detection**: Searching for fixed byte patterns in network packets
- **Text Editors**: Guaranteed linear-time find operations

### References

- LeetCode - KMP Algorithm Explained from Prefixes to Pattern Search - https://leetcode.com/discuss/post/6921580/kmp-algorithm-explained-from-prefixes-to-u83t/
- GitHub - KMP Algorithm Documentation - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/fundamental/algorithm/9-string-algorithms/2-kmp-algorithm.mdx

---

## 3. Z-Algorithm and the Z-Array

### Definitions

**Core Definition**
The Z-array for a string \( S \) stores at each index \( i \) the length of the longest substring starting at \( i \) that matches a prefix of \( S \). The Z-algorithm computes this array in linear time.

**Technical Definition**
\( Z[i] \) = length of the longest common prefix of \( S \) and \( S[i..n-1] \). The algorithm maintains a Z-box \( [l, r] \) representing the rightmost interval where \( S[l..r] \) matches a prefix, using mirror values to avoid redundant comparisons.

**Beginner-Friendly Explanation**
For each position in the string, the Z-value tells you "how many characters from here match the beginning of the string." If the string starts with "abc" and position 5 also has "abc", then Z[5] = 3.

### Purposes (All begin with "To")

- **To** provide an alternative linear-time pattern matching method to KMP
- **To** find all occurrences of a pattern by concatenating pattern + separator + text
- **To** solve string periodicity and border problems

### Syntax Rules and Structure

**General Syntax**
```
Z = [0] * n
l = r = 0
for i from 1 to n-1:
    if i < r:
        Z[i] = min(r - i, Z[i - l])
    while i + Z[i] < n and S[Z[i]] == S[i + Z[i]]:
        Z[i] += 1
    if i + Z[i] > r:
        l = i
        r = i + Z[i]
Z[0] = n
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `[l, r]` | Z-box (matched prefix interval) | s[l:r] == s[0:r-l] |
| Mirror | `i - l` | Corresponding position inside box |
| Initial value | `min(r-i, Z[i-l])` | Guaranteed match length |
| Expansion | While loop | Extend beyond known match |

**Syntax Rules**

- `Z[0] = n` by convention (entire string matches itself)
- The Z-box always starts with `l = r = 0`
- When `i < r`, the mirror value provides a lower bound
- Expansion continues only beyond the known Z-box boundary

**Constraints and Limitations**

- Requires \( O(n) \) extra space for the Z-array
- Concatenating pattern + separator + text requires \( O(n + m) \) space
- Separator must be a character not in either string

### Annotated Code Examples

**Example 1: Z-Algorithm Implementation (Python)**
```python
def z_algorithm(s):
    """Compute Z-array in O(n) time."""
    n = len(s)
    z = [0] * n
    l = r = 0  # Z-box: s[l:r] matches s[0:r-l]
    
    for i in range(1, n):
        # If inside Z-box, use mirror value
        if i < r:
            z[i] = min(r - i, z[i - l])
        
        # Extend beyond known match
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        
        # Update Z-box if we expanded past r
        if i + z[i] > r:
            l, r = i, i + z[i]
    
    z[0] = n
    return z

def z_search(text, pattern):
    """Pattern matching using Z-algorithm."""
    # Concatenate with separator not in text/pattern
    combined = pattern + "$" + text
    z = z_algorithm(combined)
    m = len(pattern)
    matches = []
    
    # Pattern found where Z-value equals pattern length
    for i in range(m + 1, len(combined)):
        if z[i] == m:
            matches.append(i - m - 1)
    
    return matches

# Test
text = "AABAACAADAABAABA"
pattern = "AABA"
print(f"Z-array for pattern: {z_algorithm(pattern)}")
print(f"Matches: {z_search(text, pattern)}")
```

**Expected Output:**
```
Z-array for pattern: [4, 1, 0, 1]
Matches: [0, 9, 12]
```

**Why This Output Occurs**: The Z-array for "AABA" shows Z[1]=1 (A matches), Z[2]=0, Z[3]=1 (A matches). In the combined string "AABA$AABAACAADAABAABA", positions where Z equals 4 (pattern length) indicate matches .

### Real-World Cases

- **String Periodicity**: Finding repeating patterns in strings
- **Competitive Programming**: Simpler to implement than KMP for some problems
- **Bioinformatics**: Finding repeated sequences in DNA

### References

- Guide to Competitive Programming - Z-Algorithm Scenarios - /hf3fs-jd/hdd/deepseek/shared/kaidong/datasets/sci-hub/81900000/libgen.scimag81950000-81950999.zip%20---%2010.1007/978-3-030-39357-1.pdf
- GitHub - Z-Algorithm Implementation - https://raw.githubusercontent.com/handbook-academy/engineering-handbook/refs/heads/main/content/dsa/part-12-strings-pattern-matching/03-z-algorithm.md

---

## 4. Rabin-Karp Algorithm and Rolling Hash

### Definitions

**Core Definition**
Rabin-Karp uses a rolling hash function to compare the pattern's hash with each text window's hash. When hashes match, it verifies the actual characters to handle collisions.

**Technical Definition**
The hash of a string \( S[0..m-1] \) is \( \sum S[i] \cdot p^{m-1-i} \mod q \) for prime \( q \). The rolling hash updates in O(1): \( h(T[i+1..i+m]) = ((h(T[i..i+m-1]) - T[i] \cdot p^{m-1}) \cdot p + T[i+m]) \mod q \).

**Beginner-Friendly Explanation**
Instead of comparing every character, you compute a "fingerprint" (hash) of the pattern and each text window. If fingerprints match, you check the characters. This is like comparing ID numbers instead of full names—fast, but occasionally two people have the same ID (collision).

### Purposes (All begin with "To")

- **To** achieve average-case \( O(n + m) \) pattern matching with simple implementation
- **To** search for multiple patterns simultaneously by hashing all patterns
- **To** implement plagiarism detection (comparing document hashes)

### Syntax Rules and Structure

**General Syntax**
```
p = 31  # base (prime > alphabet size)
q = 1_000_000_007  # large prime modulus

# Compute hash of pattern
h_pattern = 0
for c in pattern:
    h_pattern = (h_pattern * p + ord(c)) % q

# Compute hash of first window
h_window = 0
for i in range(m):
    h_window = (h_window * p + ord(text[i])) % q

# Slide window
for i in range(n - m + 1):
    if h_window == h_pattern:
        if text[i:i+m] == pattern:
            report match at i
    if i < n - m:
        h_window = ((h_window - ord(text[i]) * p_pow) * p + ord(text[i+m])) % q
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| Base `p` | Hash multiplier | 31, 53, or 101 |
| Modulus `q` | Prime for range | 10^9+7 or 10^9+9 |
| `p_pow` | `p^(m-1) mod q` | Precomputed |
| Rolling update | O(1) window slide | Subtract leading, shift, add trailing |

**Syntax Rules**

- Base must be larger than alphabet size to avoid trivial collisions
- Modulus must be prime and large to minimize collisions
- Always verify actual characters when hashes match (collision handling)
- Use precomputed powers for efficiency

**Constraints and Limitations**

- Hash collisions cause false positives (verified by character comparison)
- Worst-case \( O(n \cdot m) \) if many hash collisions occur
- Choosing poor base/modulus increases collision probability

### Annotated Code Examples

**Example 1: Rabin-Karp with Rolling Hash (Python)**
```python
def rabin_karp(text, pattern):
    """Rabin-Karp pattern matching with rolling hash."""
    n, m = len(text), len(pattern)
    if m > n:
        return []
    
    # Parameters
    p = 31  # base
    q = 1_000_000_007  # prime modulus
    
    # Precompute p^(m-1) mod q
    p_pow = pow(p, m - 1, q)
    
    # Compute hash of pattern
    h_pattern = 0
    for c in pattern:
        h_pattern = (h_pattern * p + ord(c)) % q
    
    # Compute hash of first window
    h_window = 0
    for i in range(m):
        h_window = (h_window * p + ord(text[i])) % q
    
    matches = []
    
    # Slide window
    for i in range(n - m + 1):
        # Hash match: verify actual characters
        if h_window == h_pattern:
            if text[i:i+m] == pattern:
                matches.append(i)
        
        # Rolling hash update for next window
        if i < n - m:
            # Remove leading character, shift, add trailing
            h_window = (h_window - ord(text[i]) * p_pow) % q
            h_window = (h_window * p + ord(text[i + m])) % q
            h_window = (h_window + q) % q  # Ensure positive
    
    return matches

# Test
text = "AABAACAADAABAABA"
pattern = "AABA"
print(rabin_karp(text, pattern))  # [0, 9, 12]
```

**Expected Output:**
```
[0, 9, 12]
```

**Why This Output Occurs**: The rolling hash computes the hash of each window in O(1) using the previous window's hash. When the hash matches the pattern's hash, the actual characters are verified to rule out collisions .

### Real-World Cases

- **Plagiarism Detection**: Comparing document fingerprints
- **Multiple Pattern Search**: Hash all patterns, compare against text windows
- **Bioinformatics**: Finding repeated sequences in DNA
- **Data Deduplication**: Detecting identical chunks in storage

### References

- MIT 6.006 - Rabin-Karp and Rolling Hashes - https://courses.csail.mit.edu/6.006/spring09/notes/lecture07.pdf
- CMU 15-451 - Karp-Rabin Fingerprint Method - http://www.cs.cmu.edu/afs/cs/academic/class/15451-f14/www/lectures/lec6/karp-rabin-09-15-14.pdf
- ITB - Optimizing Rolling Hash in Rabin-Karp - http://informatika.stei.itb.ac.id/~rinaldi.munir/Matdis/2014-2015/Makalah2014/Makalah-IF2120-2014-045.pdf

---

## 5. Boyer-Moore Algorithm (Bad Character and Good Suffix Shifts)

### Definitions

**Core Definition**
Boyer-Moore scans the pattern from right to left and uses two precomputed shift tables—bad character and good suffix—to skip large portions of the text on mismatch.

**Technical Definition**
The bad character shift aligns the mismatched text character with its rightmost occurrence in the pattern. The good suffix shift aligns the matched suffix with its next occurrence in the pattern. The algorithm shifts by the maximum of the two.

**Beginner-Friendly Explanation**
Instead of checking left-to-right, Boyer-Moore checks right-to-left. If the last character doesn't match, it uses a table to jump ahead—sometimes skipping many characters at once. It's like reading a book backwards to find a word faster.

### Purposes (All begin with "To")

- **To** achieve sublinear average-case pattern matching on large alphabets
- **To** power text editors' search-and-replace (e.g., `grep`)
- **To** outperform KMP in practice for natural language text

### Syntax Rules and Structure

**General Syntax (Bad Character Table)**
```
# bmBc[c] = rightmost index of c in pattern, or -1 if not present
for each character c in alphabet:
    bmBc[c] = -1
for i from 0 to m-1:
    bmBc[P[i]] = i
```

**General Syntax (Good Suffix Table)**
```
# bmGs[i] = shift when mismatch at pattern position i
# Computed from suffix and prefix matches
```

**General Syntax (Search)**
```
s = 0  # shift of pattern relative to text
while s <= n - m:
    j = m - 1
    while j >= 0 and P[j] == T[s + j]:
        j -= 1
    if j < 0:
        report match at s
        s += bmGs[0]  # Shift for next match
    else:
        s += max(bmGs[j], j - bmBc[T[s + j]])
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| Bad character | Mismatched text char | `T[s+j]` |
| `bmBc[c]` | Rightmost index of c in P | -1 if absent |
| `bmGs[j]` | Good suffix shift | 0 to m |
| Shift | Maximum of two heuristics | `max(bmGs, bad_char)` |

**Syntax Rules**

- Bad character shift can be negative; good suffix shift always positive
- Use maximum of both shifts to avoid negative movement
- Preprocessing is \( O(m + \sigma) \) where \( \sigma \) is alphabet size
- Best for large alphabets relative to pattern length

**Constraints and Limitations**

- Worst-case \( O(n \cdot m) \) for periodic patterns
- Not recommended for binary strings (small alphabet); KMP is better
- Requires \( O(m + \sigma) \) extra space

### Annotated Code Examples

**Example 1: Boyer-Moore Bad Character Heuristic (Python)**
```python
def boyer_moore_bad_char(text, pattern):
    """Boyer-Moore with bad character heuristic only."""
    n, m = len(text), len(pattern)
    if m > n:
        return []
    
    # Build bad character table: rightmost index of each char in pattern
    bmBc = {}
    for i, c in enumerate(pattern):
        bmBc[c] = i
    
    matches = []
    s = 0  # Shift of pattern
    
    while s <= n - m:
        j = m - 1
        
        # Compare right to left
        while j >= 0 and pattern[j] == text[s + j]:
            j -= 1
        
        if j < 0:
            # Full match
            matches.append(s)
            s += 1  # Shift by 1 for next possible match
        else:
            # Bad character shift
            bad_char = text[s + j]
            last_occurrence = bmBc.get(bad_char, -1)
            shift = j - last_occurrence
            s += max(1, shift)  # Ensure at least shift by 1
    
    return matches

# Test
text = "HERE IS A SIMPLE EXAMPLE"
pattern = "EXAMPLE"
print(boyer_moore_bad_char(text, pattern))  # [17]
```

**Expected Output:**
```
[17]
```

**Why This Output Occurs**: The pattern "EXAMPLE" is found at index 17. Boyer-Moore scans right-to-left: comparing "E" (last char) with text, then shifting based on the bad character table when mismatches occur .

### Real-World Cases

- **Text Editors**: `grep`, `sed`, and IDE search functions
- **Database Search**: Full-text search in large documents
- **Network Security**: Pattern matching in packet payloads

### References

- MathWorks - Boyer-Moore Search Algorithm - https://kr.mathworks.com/matlabcentral/answers/54309-boyer-moore-search-algorithm
- UWA - Boyer-Moore Heuristics - https://teaching.csse.uwa.edu.au/units/CITS3001/Semester1/lectures/lectures/3001%20String%20algorithms.pdf
- TUM - Boyer-Moore Preprocessing - https://db.cs.tum.edu/people/sites/riedl/papers/mathesis.pdf

---

## 6. Aho-Corasick Algorithm (Multi-Pattern Matching)

### Definitions

**Core Definition**
Aho-Corasick builds a finite automaton (trie with failure links) from a set of patterns, enabling simultaneous search for all patterns in a single pass over the text.

**Technical Definition**
The algorithm constructs a trie of all patterns, adds failure links (similar to KMP's LPS) that point to the longest proper suffix of the current state, and output links that report matches at each state.

**Beginner-Friendly Explanation**
Instead of searching for one word at a time, you build a "super-search-machine" that can find all patterns simultaneously. As you read the text once, the machine tracks which patterns are being matched.

### Purposes (All begin with "To")

- **To** search for multiple patterns in a single pass over the text
- **To** power intrusion detection systems (searching for many attack signatures)
- **To** implement dictionary-based word matching and filtering

### Syntax Rules and Structure

**General Syntax (Building Automaton)**
```
# Build trie from all patterns
for each pattern:
    insert into trie

# Build failure links (BFS)
for each node in BFS order:
    for each child c:
        if node == root:
            child.fail = root
        else:
            fail = node.fail
            while fail != root and c not in fail.children:
                fail = fail.fail
            child.fail = fail.children[c] if c in fail.children else root
```

**General Syntax (Searching)**
```
state = root
for each character c in text:
    while state != root and c not in state.children:
        state = state.fail
    state = state.children.get(c, root)
    if state is terminal or has output:
        report all patterns ending at this state
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| Trie | Prefix tree of patterns | Nodes for each prefix |
| Failure link | Longest proper suffix | Similar to LPS |
| Output | Patterns ending at state | List of pattern IDs |
| State | Current trie node | Root to leaf |

**Syntax Rules**

- Build trie first, then add failure links using BFS
- Failure links are computed for each node based on parent's failure
- Output links propagate patterns from failure nodes
- Search time is \( O(n + z) \) where \( z \) is number of matches

**Constraints and Limitations**

- Memory usage grows with total pattern length × alphabet size
- Overlapping matches require special handling (standard AC reports all)
- For very large pattern sets, memory can be a constraint

### Annotated Code Examples

**Example 1: Aho-Corasick Search (Python)**
```python
from collections import deque

class AhoCorasick:
    def __init__(self):
        self.trie = {}  # node -> {char: next_node}
        self.fail = {}  # node -> failure node
        self.output = {}  # node -> list of patterns
        self.next_id = 0
    
    def add_pattern(self, pattern):
        """Insert pattern into trie."""
        node = 0
        for char in pattern:
            if node not in self.trie:
                self.trie[node] = {}
            if char not in self.trie[node]:
                self.next_id += 1
                self.trie[node][char] = self.next_id
            node = self.trie[node][char]
        self.output.setdefault(node, []).append(pattern)
    
    def build(self):
        """Build failure links via BFS."""
        queue = deque()
        self.fail[0] = 0
        
        # Root's children fail to root
        for char, child in self.trie.get(0, {}).items():
            self.fail[child] = 0
            queue.append(child)
        
        while queue:
            node = queue.popleft()
            
            for char, child in self.trie.get(node, {}).items():
                queue.append(child)
                
                # Find failure link
                fail = self.fail[node]
                while fail != 0 and char not in self.trie.get(fail, {}):
                    fail = self.fail[fail]
                
                if char in self.trie.get(fail, {}):
                    self.fail[child] = self.trie[fail][char]
                else:
                    self.fail[child] = 0
                
                # Propagate output
                self.output.setdefault(child, []).extend(
                    self.output.get(self.fail[child], [])
                )
    
    def search(self, text):
        """Find all pattern occurrences."""
        node = 0
        matches = []
        
        for i, char in enumerate(text):
            # Follow failure links until match or root
            while node != 0 and char not in self.trie.get(node, {}):
                node = self.fail[node]
            
            node = self.trie.get(node, {}).get(char, 0)
            
            # Report matches
            for pattern in self.output.get(node, []):
                matches.append((i - len(pattern) + 1, pattern))
        
        return matches

# Test
ac = AhoCorasick()
for p in ["he", "she", "his", "hers"]:
    ac.add_pattern(p)
ac.build()

text = "ushers"
print(ac.search(text))
```

**Expected Output:**
```
[(1, 'she'), (2, 'he'), (2, 'hers')]
```

**Why This Output Occurs**: In "ushers", "she" starts at index 1, "he" at index 2, and "hers" at index 2. The automaton tracks all patterns simultaneously, reporting matches as they complete .

### Real-World Cases

- **Intrusion Detection**: Snort and other NIDS use AC for signature matching
- **Antivirus**: Scanning files for multiple virus signatures
- **Content Filtering**: Blocking multiple banned words simultaneously
- **Bioinformatics**: Finding multiple gene sequences

### References

- Rust Aho-Corasick Crate - https://docs.fuchsia.dev/rust/aho_corasick/index.html
- IEEE - Optimized Aho-Corasick Multi-Pattern Matching - https://xplorestaging.ieee.org/document/9342041/references
- Google Open Source - Aho-Corasick Implementation Notes - https://android.googlesource.com/toolchain/sccache/+/eae7e5b5d1b885123cf8b327da89bf5d9b90f2dd%5E%21/android/vendor/aho-corasick-0.7.20

---

## 7. Manacher's Algorithm (Linear-Time Longest Palindromic Substring)

### Definitions

**Core Definition**
Manacher's algorithm finds the longest palindromic substring in O(n) time by transforming the string to handle even-length palindromes and using palindrome symmetry to avoid redundant expansion.

**Technical Definition**
The transformed string \( S' = \#s_0\#s_1\#\ldots\# \) allows all palindromes to have odd length. The algorithm maintains the rightmost palindrome boundary \( R \) and center \( C \), using mirror values \( P[2C-i] \) to initialize the palindrome radius at each position.

**Beginner-Friendly Explanation**
Instead of checking every possible center (which takes O(n²)), Manacher's algorithm remembers the rightmost palindrome you've found and uses symmetry to skip work. If a position is inside a known palindrome, its mirror's radius gives you a head start.

### Purposes (All begin with "To")

- **To** find the longest palindromic substring in linear time
- **To** count all palindromic substrings efficiently
- **To** solve palindrome-related problems in competitive programming

### Syntax Rules and Structure

**General Syntax**
```
# Transform: insert # between chars and sentinels at ends
T = '#'.join('^{}$'.format(s))
n = len(T)
P = [0] * n  # P[i] = radius of palindrome at i
C = R = 0

for i in range(1, n - 1):
    mirror = 2 * C - i
    if R > i:
        P[i] = min(R - i, P[mirror])
    
    # Expand around i
    while T[i + 1 + P[i]] == T[i - 1 - P[i]]:
        P[i] += 1
    
    # Update C and R if expanded past R
    if i + P[i] > R:
        C = i
        R = i + P[i]
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `T` | Transformed string | `^#a#b#a#$` |
| `P[i]` | Palindromic radius | Number of matched pairs |
| `C` | Center of rightmost palindrome | Updated when R expands |
| `R` | Right boundary | `C + P[C]` |
| Mirror | `2*C - i` | Symmetric position |

**Syntax Rules**

- Transformation handles both odd and even palindromes uniformly
- Sentinel characters (`^`, `$`) prevent bounds checking
- Mirror value provides lower bound when `i < R`
- Expansion only occurs beyond the known boundary

**Constraints and Limitations**

- Requires O(n) extra space for transformed string and P array
- Virtualized augmentation can eliminate transformed string (saves memory)
- Only finds longest palindrome; counting all requires summing P values

### Annotated Code Examples

**Example 1: Manacher's Algorithm (Python)**
```python
def longest_palindrome(s):
    """Find longest palindromic substring using Manacher's algorithm."""
    if not s:
        return ""
    
    # Transform: "abba" -> "^#a#b#b#a#$"
    T = '#'.join('^{}$'.format(s))
    n = len(T)
    P = [0] * n
    C = R = 0
    
    for i in range(1, n - 1):
        # Mirror index
        mirror = 2 * C - i
        
        # Use previously computed info if inside boundary
        if R > i:
            P[i] = min(R - i, P[mirror])
        
        # Expand around i
        while T[i + 1 + P[i]] == T[i - 1 - P[i]]:
            P[i] += 1
        
        # Update center and right boundary
        if i + P[i] > R:
            C = i
            R = i + P[i]
    
    # Find maximum
    max_len = max(P)
    center = P.index(max_len)
    start = (center - max_len) // 2
    return s[start:start + max_len]

# Test
print(longest_palindrome("babad"))    # "bab" or "aba"
print(longest_palindrome("cbbd"))     # "bb"
print(longest_palindrome("racecar"))  # "racecar"
```

**Expected Output:**
```
bab
bb
racecar
```

**Why This Output Occurs**: The transformed string `"^#b#a#b#a#d#$"` allows the algorithm to find palindromes at every center. The `P` array stores the radius at each center. The maximum radius corresponds to the longest palindrome .

### Real-World Cases

- **Text Analysis**: Finding palindromic phrases in literature
- **DNA Sequences**: Palindromic sequences in genetic code
- **Data Validation**: Checking input format constraints
- **Competitive Programming**: LeetCode Longest Palindromic Substring

### References

- GitHub - Manacher's Algorithm Tutorial - https://raw.githubusercontent.com/wingkwong/leetcode-the-hard-way/refs/heads/main/tutorials/strings/manachers-algorithm.md
- GitHub - Manacher's Algorithm Implementation - https://raw.githubusercontent.com/ajay-dhangar/algo/refs/heads/main/docs/extra/string-algorithms/manachers-algorithm.md
- Stack Overflow - Manacher's Algorithm Explanation - https://stackoverflow.com/questions/10468208/manachers-algorithm-algorithm-to-find-longest-palindrome-substring-in-linear-t

---

## References (Consolidated)

- Princeton University - Brute-Force Analysis - https://www.cs.princeton.edu/courses/archive/spring02/cs226/lectures/string.4up.pdf
- LeetCode - KMP Algorithm Explained from Prefixes to Pattern Search - https://leetcode.com/discuss/post/6921580/kmp-algorithm-explained-from-prefixes-to-u83t/
- GitHub - KMP Algorithm Documentation - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/fundamental/algorithm/9-string-algorithms/2-kmp-algorithm.mdx
- Guide to Competitive Programming - Z-Algorithm Scenarios - /hf3fs-jd/hdd/deepseek/shared/kaidong/datasets/sci-hub/81900000/libgen.scimag81950000-81950999.zip%20---%2010.1007/978-3-030-39357-1.pdf
- GitHub - Z-Algorithm Implementation - https://raw.githubusercontent.com/handbook-academy/engineering-handbook/refs/heads/main/content/dsa/part-12-strings-pattern-matching/03-z-algorithm.md
- MIT 6.006 - Rabin-Karp and Rolling Hashes - https://courses.csail.mit.edu/6.006/spring09/notes/lecture07.pdf
- CMU 15-451 - Karp-Rabin Fingerprint Method - http://www.cs.cmu.edu/afs/cs/academic/class/15451-f14/www/lectures/lec6/karp-rabin-09-15-14.pdf
- ITB - Optimizing Rolling Hash in Rabin-Karp - http://informatika.stei.itb.ac.id/~rinaldi.munir/Matdis/2014-2015/Makalah2014/Makalah-IF2120-2014-045.pdf
- MathWorks - Boyer-Moore Search Algorithm - https://kr.mathworks.com/matlabcentral/answers/54309-boyer-moore-search-algorithm
- UWA - Boyer-Moore Heuristics - https://teaching.csse.uwa.edu.au/units/CITS3001/Semester1/lectures/lectures/3001%20String%20algorithms.pdf
- TUM - Boyer-Moore Preprocessing - https://db.cs.tum.edu/people/sites/riedl/papers/mathesis.pdf
- Rust Aho-Corasick Crate - https://docs.fuchsia.dev/rust/aho_corasick/index.html
- IEEE - Optimized Aho-Corasick Multi-Pattern Matching - https://xplorestaging.ieee.org/document/9342041/references
- Google Open Source - Aho-Corasick Implementation Notes - https://android.googlesource.com/toolchain/sccache/+/eae7e5b5d1b885123cf8b327da89bf5d9b90f2dd%5E%21/android/vendor/aho-corasick-0.7.20
- GitHub - Manacher's Algorithm Tutorial - https://raw.githubusercontent.com/wingkwong/leetcode-the-hard-way/refs/heads/main/tutorials/strings/manachers-algorithm.md
- GitHub - Manacher's Algorithm Implementation - https://raw.githubusercontent.com/ajay-dhangar/algo/refs/heads/main/docs/extra/string-algorithms/manachers-algorithm.md
- Stack Overflow - Manacher's Algorithm Explanation - https://stackoverflow.com/questions/10468208/manachers-algorithm-algorithm-to-find-longest-palindrome-substring-in-linear-t