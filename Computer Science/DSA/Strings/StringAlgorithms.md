# DSA Core String Algorithmic Patterns: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
String algorithmic patterns are reusable computational strategies that exploit the sequential, character-based nature of strings to solve problems involving counting, matching, searching, and transforming text data efficiently.

**Technical Definition**
These patterns constitute template solutions characterized by specific index movement invariants, frequency tracking mechanisms, or precomputation schemes that reduce naive O(n²) or O(n³) approaches to O(n) or O(n log n) while maintaining optimal auxiliary space.

**Beginner-Friendly Explanation**
Think of string patterns as recipes for cooking with words. Instead of figuring out how to find anagrams or palindromes from scratch every time, you recognize "this problem is like checking if two words use the same letters" and apply the frequency-counting recipe.

### Key Characteristics

- **Character-Level Processing**: Operations work on individual characters or code units
- **Frequency-Driven**: Many patterns rely on counting character occurrences
- **Contiguity Awareness**: Distinguishing between substrings (contiguous) and subsequences (non-contiguous)
- **Linear Time Optimization**: Patterns typically reduce brute-force approaches to O(n)

### Prerequisites

- String fundamentals (indexing, traversal, comparison)
- Arrays and hash maps for frequency counting
- Two-pointer and sliding window techniques
- Big O notation for complexity analysis

### Related Programming Areas

- **Text Processing**: Search engines, compilers, editors
- **Bioinformatics**: DNA/RNA sequence analysis
- **Natural Language Processing**: Tokenization, stemming
- **Competitive Programming**: Interview and contest problems

### Core Concepts / Features

| Pattern | Primary Use Case | Complexity |
|---------|-----------------|------------|
| Frequency Analysis | Anagrams, duplicates, majority | O(n) |
| Palindrome Detection | Palindromic substrings | O(n) to O(n²) |
| Anagram Detection | Word games, categorization | O(n log n) or O(n) |
| String Reversal | In-place or word-by-word | O(n) |
| Sliding Window on Strings | Substring constraints | O(n) |
| Prefix/Suffix Analysis | LCP, pattern matching | O(n) to O(n log n) |
| String Conversion/Parsing | atoi, itoa, validation | O(n) |

---

## 1. Frequency Analysis and Frequency Arrays / Hash Maps

### Definitions

**Core Definition**
Frequency analysis counts how many times each character (or element) appears in a string, using either a fixed-size array for bounded character sets or a hash map for arbitrary keys.

**Technical Definition**
For ASCII strings, an integer array `freq[128]` maps each character to its occurrence count in O(n) time. For Unicode or arbitrary keys, a hash map provides O(1) average lookup with O(distinct) space.

**Beginner-Friendly Explanation**
Tallying how many times each letter appears in a sentence. You make a mark next to each letter on a tally sheet, then read off the counts at the end.

### Purposes (All begin with "To")

- **To** detect anagrams by comparing character frequency distributions
- **To** find the most or least frequent character in a string
- **To** implement the Boyer-Moore majority vote for finding elements occurring more than n/2 times
- **To** validate input constraints (e.g., checking character set compliance)

### Syntax Rules and Structure

**General Syntax (Frequency Array)**
```python
# For lowercase English letters
freq = [0] * 26
for char in s:
    freq[ord(char) - ord('a')] += 1
```

**General Syntax (Hash Map)**
```python
from collections import Counter
freq = Counter(s)
# Access: freq[char] returns count
```

**Component Breakdown**

| Component | Description | Array | Hash Map |
|-----------|-------------|-------|----------|
| Storage | Count per key | `int[128]` or `int[26]` | Dictionary |
| Index | Character code | `ord(c) - offset` | Direct key |
| Lookup | Get frequency | O(1) | O(1) average |
| Space | Total slots | O(alphabet size) | O(distinct chars) |

**Syntax Rules**

- Array-based: character codes must map to non-negative indices within bounds
- Hash-based: any hashable key (characters, strings, tuples)
- Initialize all counts to 0 before processing
- The Boyer-Moore majority vote requires a second pass to verify the candidate 

**Constraints and Limitations**

- Frequency arrays waste memory for sparse character sets with large ranges
- Hash maps have higher constant overhead than arrays
- The majority vote algorithm only finds an element occurring **more than** n/2 times; it does not find the most frequent element (mode) if no majority exists 

### Annotated Code Examples

**Example 1: Boyer-Moore Majority Vote Algorithm**
```python
def majority_element(nums):
    """
    Find element appearing more than n/2 times using Boyer-Moore vote.
    Returns the majority element if it exists, otherwise returns a candidate.
    """
    # Phase 1: Find a candidate
    candidate = None
    count = 0
    
    for num in nums:
        if count == 0:
            candidate = num
            count = 1
        elif num == candidate:
            count += 1
        else:
            count -= 1
    
    # Phase 2: Verify the candidate (REQUIRED)
    if nums.count(candidate) > len(nums) // 2:
        return candidate
    return None  # No majority exists

# Test
print(majority_element([3, 2, 3]))              # 3 (occurs 2/3 > 1.5)
print(majority_element([2, 2, 1, 1, 1, 2, 2]))  # 2 (occurs 4/7 > 3.5)
print(majority_element([1, 2, 3]))              # None (no majority)
```

**Expected Output:**
```
3
2
None
```

**Why This Output Occurs**: The algorithm cancels out pairs of different elements, leaving the majority element as the surviving candidate . The second pass verifies that the candidate truly appears more than n/2 times. Without the second pass, the algorithm returns a candidate that may not be the majority .

**Example 2: Anagram Detection via Frequency**
```python
def are_anagrams(s1, s2):
    """Check if two strings are anagrams using frequency counting."""
    if len(s1) != len(s2):
        return False
    
    freq = {}
    for char in s1:
        freq[char] = freq.get(char, 0) + 1
    
    for char in s2:
        if char not in freq:
            return False
        freq[char] -= 1
        if freq[char] < 0:
            return False
    
    return all(v == 0 for v in freq.values())

# Test
print(are_anagrams("listen", "silent"))  # True
print(are_anagrams("hello", "world"))    # False
```

**Expected Output:**
```
True
False
```

**Why This Output Occurs**: The first string's character counts are incremented; the second string's counts are decremented. If the strings are anagrams, all counts return to zero.

### Real-World Cases

- **Word Games**: Finding anagrams in Scrabble or crossword puzzles
- **Data Validation**: Ensuring passwords contain required character types
- **Bioinformatics**: Counting nucleotide frequencies in DNA sequences
- **Voting Systems**: Finding majority candidates in elections 

### References

- Wikipedia - Boyer–Moore majority vote algorithm - https://en.m.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm 
- Stack Overflow - Linear Time Voting Algorithm Discussion - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/780937/ 
- Stony Brook University - Majority Boyer-Moore Two-Pass Algorithm - https://www3.cs.stonybrook.edu/~pramod.ganapathi/doc/algorithms/Algo-AlgorithmicProblemSolving.pdf 

---

## 2. Palindrome Detection and Subsequence vs. Substring Tracking

### Definitions

**Core Definition**
A palindrome reads the same forwards and backwards. Detection involves checking symmetry. Substrings are contiguous segments; subsequences maintain relative order but need not be contiguous.

**Technical Definition**
Palindrome detection via two pointers compares `s[i]` with `s[n-1-i]` for `i` from 0 to n/2. Manacher's algorithm finds the longest palindromic substring in O(n) using mirror properties and a transformed string with separators .

**Beginner-Friendly Explanation**
A palindrome is a word like "racecar" that looks the same in a mirror. A substring is a continuous piece of text ("race" in "racecar"). A subsequence can skip letters ("rcr" is a subsequence of "racecar").

### Purposes (All begin with "To")

- **To** check if a string reads the same forwards and backwards
- **To** find the longest palindromic substring in O(n) using Manacher's algorithm
- **To** count palindromic substrings efficiently
- **To** distinguish between palindromic subsequences (DP-based) and substrings (contiguous)

### Syntax Rules and Structure

**General Syntax (Two-Pointer Palindrome Check)**
```
left = 0
right = n - 1
while left < right:
    if s[left] != s[right]:
        return False
    left += 1
    right -= 1
return True
```

**Manacher's Algorithm Structure**
```
# Transform: "abba" → "^#a#b#b#a#$"
T = '#'.join('^{}$'.format(s))
P = [0] * len(T)  # P[i] = radius of palindrome centered at i
C, R = 0, 0       # center and right boundary

for i in range(1, len(T) - 1):
    mirror = 2 * C - i
    if R > i:
        P[i] = min(R - i, P[mirror])
    # Expand around i
    while T[i + 1 + P[i]] == T[i - 1 - P[i]]:
        P[i] += 1
    # Update center and boundary
    if i + P[i] > R:
        C, R = i, i + P[i]
```

**Component Breakdown**

| Component | Description | Manacher |
|-----------|-------------|----------|
| Transformation | Add separators for even-length palindromes | `#` between chars |
| `P[i]` | Radius of palindrome at center i | Minimum definite length |
| Mirror | `2*C - i` | Symmetric position |
| `R` | Rightmost boundary reached | Used to bound expansion |

**Syntax Rules**

- Manacher's algorithm requires transforming the string to handle even-length palindromes 
- The mirror value `P[2*C - i]` provides a lower bound on `P[i]` when `i < R` 
- Expansion continues beyond the known boundary until mismatch 
- Sentinel characters (`^` and `$`) avoid bounds checking 

**Constraints and Limitations**

- Manacher's algorithm is O(n) but requires O(n) extra space for the transformed string and `P` array
- The virtualized augmentation variant eliminates the transformed string, saving memory 
- Palindrome subsequence problems require dynamic programming (O(n²) time and space)

### Annotated Code Examples

**Example 1: Manacher's Algorithm (Longest Palindromic Substring)**
```python
def longest_palindrome(s):
    """Find longest palindromic substring using Manacher's algorithm."""
    if not s:
        return ""
    
    # Transform: add sentinels and separators
    T = '#'.join('^{}$'.format(s))
    n = len(T)
    P = [0] * n
    C = R = 0
    
    for i in range(1, n - 1):
        # Mirror index
        mirror = 2 * C - i
        
        # Use previously computed info if within boundary
        if R > i:
            P[i] = min(R - i, P[mirror])
        
        # Expand palindrome centered at i
        while T[i + 1 + P[i]] == T[i - 1 - P[i]]:
            P[i] += 1
        
        # Update center and right boundary if expanded past R
        if i + P[i] > R:
            C = i
            R = i + P[i]
    
    # Find maximum
    max_len = max(P)
    center = P.index(max_len)
    start = (center - max_len) // 2
    return s[start:start + max_len]

# Test
print(longest_palindrome("babad"))  # "bab" or "aba"
print(longest_palindrome("cbbd"))   # "bb"
print(longest_palindrome("racecar")) # "racecar"
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

- Educative - Longest Palindromic Substring with Manacher's Algorithm - https://www.educative.io/answers/longest-palindromic-substring-in-on-with-manachers-algorithm 
- GitHub - Manacher's Algorithm Tutorial - https://raw.githubusercontent.com/wingkwong/leetcode-the-hard-way/refs/heads/main/tutorials/strings/manachers-algorithm.md 
- arXiv - New Implementation of Manacher's Algorithm (Virtualized Augmentation) - http://web3.arxiv.org/pdf/2003.08211v1 
- Stack Overflow - Manacher's Algorithm Explanation - https://stackoverflow.com/questions/10468208/ 

---

## 3. Anagram Detection and Categorization

### Definitions

**Core Definition**
Anagrams are words or phrases formed by rearranging the letters of another, using all original letters exactly once. Detection compares character frequency distributions.

**Technical Definition**
Anagram detection via sorting: sort both strings and compare. Via frequency: build count arrays and verify all counts match. Categorization groups anagrams by a canonical key (sorted string or frequency signature).

**Beginner-Friendly Explanation**
"Listen" and "silent" are anagrams because they use the exact same letters. To check, you could sort both to "eilnst" and see they match, or count the letters in each.

### Purposes (All begin with "To")

- **To** detect if two strings are anagrams
- **To** group a list of words into anagram families
- **To** solve word puzzles and games
- **To** implement efficient comparison-based categorization

### Syntax Rules and Structure

**General Syntax (Sorting Approach)**
```
def is_anagram(s1, s2):
    return sorted(s1) == sorted(s2)
```

**General Syntax (Frequency Approach)**
```
def is_anagram(s1, s2):
    if len(s1) != len(s2):
        return False
    freq = [0] * 26
    for c in s1: freq[ord(c)-ord('a')] += 1
    for c in s2: freq[ord(c)-ord('a')] -= 1
    return all(f == 0 for f in freq)
```

**General Syntax (Categorization)**
```
def group_anagrams(words):
    groups = {}
    for word in words:
        key = tuple(sorted(word))  # Canonical form
        groups.setdefault(key, []).append(word)
    return list(groups.values())
```

**Component Breakdown**

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| Sorting | O(n log n) | O(n) | Simple, works for any chars |
| Frequency Array | O(n) | O(alphabet) | Bounded character set |
| Frequency Hash | O(n) | O(distinct) | Arbitrary characters |
| Categorization | O(n·k log k) | O(n·k) | Grouping words |

**Syntax Rules**

- Sorting approach is O(n log n) per comparison
- Frequency approach is O(n) with O(1) extra space for fixed alphabets
- Categorization uses a canonical key (sorted string or frequency tuple)
- Both approaches require `len(s1) == len(s2)` for anagram detection

**Constraints and Limitations**

- Sorting destroys original order but provides deterministic keys
- Frequency arrays require known alphabet bounds
- Hash-based frequency has higher constant overhead

### Annotated Code Examples

**Example 1: Anagram Grouping (Categorization)**
```python
from collections import defaultdict

def group_anagrams(words):
    """Group words that are anagrams of each other."""
    groups = defaultdict(list)
    
    for word in words:
        # Canonical key: sorted characters as tuple
        key = tuple(sorted(word))
        groups[key].append(word)
    
    return list(groups.values())

# Test
words = ["eat", "tea", "tan", "ate", "nat", "bat"]
result = group_anagrams(words)
for group in result:
    print(group)
```

**Expected Output:**
```
['eat', 'tea', 'ate']
['tan', 'nat']
['bat']
```

**Why This Output Occurs**: All anagrams share the same sorted character tuple as key. "eat", "tea", and "ate" all sort to `('a', 'e', 't')`, so they're grouped together .

### Real-World Cases

- **Word Puzzles**: Finding anagram solutions in games
- **Dictionary Organization**: Grouping words by letter composition
- **Plagiarism Detection**: Detecting rearranged text
- **Data Deduplication**: Finding similar records

### References

- Virtual Labs - String Operations References - https://ps-iiith.vlabs.ac.in/exp/string-operations/references.html 
- GitHub - String Mastery Toolkit - https://github.com/HimalayaMinds/string-mastery-toolkit 

---

## 4. String Reversal (In-Place vs. Out-of-Place, Word-by-Word)

### Definitions

**Core Definition**
String reversal inverts the order of characters. In-place reversal swaps characters from both ends using two pointers. Word-by-word reversal reverses the order of words while preserving character order within each word.

**Technical Definition**
In-place reversal: `swap(s[i], s[n-1-i])` for `i` from 0 to n/2. Word reversal: split by delimiter, reverse the word list, then rejoin.

**Beginner-Friendly Explanation**
Reversing a string is like reading a sentence backwards. Reversing word-by-word is like flipping the order of words but keeping each word intact: "hello world" becomes "world hello".

### Purposes (All begin with "To")

- **To** reverse a string efficiently without extra memory
- **To** implement word-order reversal for text processing
- **To** solve palindrome and rotation problems
- **To** prepare data for certain algorithms (e.g., reverse Polish notation)

### Syntax Rules and Structure

**General Syntax (In-Place Character Reversal)**
```
left = 0
right = n - 1
while left < right:
    s[left], s[right] = s[right], s[left]
    left += 1
    right -= 1
```

**General Syntax (Word-by-Word Reversal)**
```
words = s.split()
words.reverse()
result = ' '.join(words)
```

**Component Breakdown**

| Type | Method | Space | Notes |
|------|--------|-------|-------|
| In-place (char) | Two-pointer swap | O(1) | Mutable strings only |
| Out-of-place | `s[::-1]` | O(n) | Immutable strings |
| Word-by-word | Split + reverse + join | O(n) | Handles delimiters |

**Syntax Rules**

- In-place reversal requires a mutable character array (C, C++ std::string) 
- Python/Java strings are immutable; reversal creates a new string
- Word reversal must handle multiple spaces and leading/trailing whitespace
- `split()` with no argument handles whitespace normalization in Python

**Constraints and Limitations**

- In-place reversal is not possible with immutable strings
- Word-by-word reversal changes the length if delimiters are normalized
- Reversing by Unicode code points requires care with surrogate pairs

### Annotated Code Examples

**Example 1: In-Place Reversal (C++)**
```cpp
#include <iostream>
#include <algorithm>
#include <string>
using namespace std;

int main() {
    string s = "Hello World";
    
    // In-place reversal using std::reverse
    reverse(s.begin(), s.end());
    cout << "Reversed: " << s << endl;
    
    // Manual two-pointer reversal
    string t = "Hello World";
    int left = 0, right = t.length() - 1;
    while (left < right) {
        swap(t[left], t[right]);
        left++;
        right--;
    }
    cout << "Manual: " << t << endl;
    
    return 0;
}
```

**Expected Output:**
```
Reversed: dlroW olleH
Manual: dlroW olleH
```

**Why This Output Occurs**: Both approaches swap characters from opposite ends, meeting in the middle. C++ `std::string` is mutable, so no new string is allocated .

### Real-World Cases

- **Text Processing**: Reversing word order in sentences
- **Cipher Algorithms**: Simple encryption/decryption
- **Data Formatting**: Reversing byte order for endianness conversion
- **Competitive Programming**: Reverse words in a string problems

### References

- UTEP - Strings in C++ Reference - http://www.pkirs.utep.edu/cis3355/Additional%20Coverage/Reference%20Links/strings3.htm 

---

## 5. Sliding Window on Strings

### Definitions

**Core Definition**
The sliding window technique maintains a contiguous substring (the "window") that slides through the string, expanding and contracting based on constraints, to solve substring problems in O(n) time.

**Technical Definition**
A window `[left, right]` is maintained such that the substring satisfies a given property. `right` advances to include new characters; `left` advances to restore validity when constraints are violated.

**Beginner-Friendly Explanation**
Imagine looking at a sentence through a small window. You slide the window forward one word at a time, adjusting its size based on what you're looking for—like finding the longest stretch without repeating a letter.

### Purposes (All begin with "To")

- **To** find the longest substring without repeating characters
- **To** find the minimum window containing all characters of another string
- **To** count substrings satisfying specific constraints
- **To** solve substring anagram problems efficiently

### Syntax Rules and Structure

**General Syntax (Variable Window)**
```
left = 0
window_state = {}  # or frequency array
best = 0

for right in range(len(s)):
    # Expand: add s[right]
    add_to_window(s[right])
    
    # Shrink: while invalid
    while not valid(window_state):
        remove_from_window(s[left])
        left += 1
    
    # Update best answer
    best = max(best, right - left + 1)
```

**Component Breakdown**

| Component | Description | Example Problem |
|-----------|-------------|-----------------|
| Window | `[left, right]` | Current substring |
| Expand | Add `s[right]` | Always done |
| Shrink | Move `left` | When constraint violated |
| State | Track window contents | Frequency map, set |

**Syntax Rules**

- `right` always advances; `left` advances only when needed
- Window validity condition must be monotonic (adding characters never fixes a violation)
- Each character is added and removed at most once, giving O(n) total

**Constraints and Limitations**

- Only works for contiguous substring problems
- The validity condition must be checkable in O(1) or O(log n)
- Requires careful handling of window state updates

### Annotated Code Examples

**Example 1: Longest Substring Without Repeating Characters**
```python
def length_of_longest_substring(s):
    """Find length of longest substring without repeating characters."""
    char_index = {}  # Store last index of each character
    left = 0
    max_length = 0
    
    for right, char in enumerate(s):
        # If character seen before and within current window
        if char in char_index and char_index[char] >= left:
            # Shrink window: move left past the duplicate
            left = char_index[char] + 1
        
        # Update last seen index
        char_index[char] = right
        
        # Update max length
        max_length = max(max_length, right - left + 1)
    
    return max_length

# Test
print(length_of_longest_substring("abcabcbb"))  # 3 ("abc")
print(length_of_longest_substring("bbbbb"))      # 1 ("b")
print(length_of_longest_substring("pwwkew"))    # 3 ("wke")
```

**Expected Output:**
```
3
1
3
```

**Why This Output Occurs**: For "abcabcbb", the window expands to "abc" (length 3). When the second 'a' is encountered, `left` jumps to index 1 (past the first 'a'), and the window becomes "bca". The maximum length is 3 .

### Real-World Cases

- **Text Analysis**: Finding unique character sequences
- **Network Monitoring**: Detecting patterns in packet streams
- **Bioinformatics**: Finding unique DNA subsequences
- **Interview Problems**: Minimum Window Substring, Find All Anagrams

### References

- GitHub - String Mastery Toolkit - https://github.com/HimalayaMinds/string-mastery-toolkit 

---

## 6. Prefix/Suffix Analysis and Longest Common Prefix (LCP)

### Definitions

**Core Definition**
Prefix analysis examines the beginning portion of strings; suffix analysis examines the ending. The Longest Common Prefix (LCP) is the longest string that is a prefix of all strings in a set.

**Technical Definition**
LCP of strings `S1, S2, ..., Sk` is the longest string `P` such that `P` is a prefix of every `Si`. LCP can be computed by vertical scanning (compare character positions across all strings) or horizontal scanning (compare pairwise).

**Beginner-Friendly Explanation**
The LCP is the shared beginning of a group of words. For "flower", "flow", and "flight", the LCP is "fl" because all three start with those letters.

### Purposes (All begin with "To")

- **To** find the common beginning of multiple strings
- **To** optimize string sorting and comparison
- **To** implement autocomplete and prefix matching
- **To** solve pattern matching problems efficiently

### Syntax Rules and Structure

**General Syntax (Horizontal Scanning)**
```
def longest_common_prefix(strs):
    if not strs:
        return ""
    
    prefix = strs[0]
    for s in strs[1:]:
        while not s.startswith(prefix):
            prefix = prefix[:-1]
            if not prefix:
                return ""
    return prefix
```

**General Syntax (Vertical Scanning)**
```
def longest_common_prefix(strs):
    if not strs:
        return ""
    
    for i in range(len(strs[0])):
        char = strs[0][i]
        for s in strs[1:]:
            if i >= len(s) or s[i] != char:
                return strs[0][:i]
    return strs[0]
```

**Component Breakdown**

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Horizontal | O(S) | O(1) | S = sum of all chars |
| Vertical | O(S) | O(1) | Early termination on mismatch |
| Sorting + LCP | O(n log n + m) | O(1) | Compare first and last |

**Syntax Rules**

- Horizontal scanning: start with first string, trim until it's a prefix of each subsequent string
- Vertical scanning: compare character by character across all strings simultaneously
- Empty string or empty list returns `""`

**Constraints and Limitations**

- Horizontal scanning may do unnecessary comparisons if the prefix shrinks quickly
- Vertical scanning is more efficient when there's an early mismatch
- Both approaches are O(S) where S is the total number of characters

### Annotated Code Examples

**Example 1: Longest Common Prefix (Vertical Scanning)**
```python
def longest_common_prefix(strs):
    """Find LCP using vertical scanning."""
    if not strs:
        return ""
    
    # Iterate character positions in first string
    for i in range(len(strs[0])):
        char = strs[0][i]
        
        # Compare with all other strings
        for s in strs[1:]:
            # Mismatch or string too short
            if i >= len(s) or s[i] != char:
                return strs[0][:i]
    
    return strs[0]

# Test
print(longest_common_prefix(["flower", "flow", "flight"]))  # "fl"
print(longest_common_prefix(["dog", "racecar", "car"]))     # ""
print(longest_common_prefix(["apple", "app", "application"])) # "app"
```

**Expected Output:**
```
fl

app
```

**Why This Output Occurs**: For `["flower", "flow", "flight"]`, position 0 is 'f' in all, position 1 is 'l' in all, position 2 is 'o' vs 'i'—mismatch, so LCP is "fl". For `["dog", "racecar", "car"]`, position 0 has 'd', 'r', 'c'—immediate mismatch, so "".

### Real-World Cases

- **Autocomplete**: Finding common prefix for suggestions
- **File Systems**: Finding common directory paths
- **Version Control**: Comparing commit messages
- **Bioinformatics**: Finding common DNA sequences

### References

- Virtual Labs - String Operations References - https://ps-iiith.vlabs.ac.in/exp/string-operations/references.html 

---

## 7. String Conversion and Parsing (atoi, itoa, Expression Validation)

### Definitions

**Core Definition**
String conversion transforms between string and numeric representations. `atoi` (ASCII to integer) parses a string into an integer. `itoa` converts an integer to its string representation. Expression validation checks whether a mathematical expression is well-formed.

**Technical Definition**
`atoi` processes leading whitespace, optional sign, and consecutive digits until a non-digit is encountered. Expression validation uses a stack to match parentheses and verify operator placement.

**Beginner-Friendly Explanation**
Converting "123" to the number 123 is `atoi`. Converting 123 to "123" is `itoa`. Validation ensures that "((1+2)*3)" is well-formed but "((1+2)" is not.

### Purposes (All begin with "To")

- **To** parse numeric input from strings (user input, files, APIs)
- **To** convert numbers to strings for display or serialization
- **To** validate mathematical expressions for correctness
- **To** implement custom parsers for domain-specific languages

### Syntax Rules and Structure

**General Syntax (atoi Implementation)**
```
def my_atoi(s):
    i = 0
    n = len(s)
    
    # Skip whitespace
    while i < n and s[i] == ' ':
        i += 1
    
    # Sign
    sign = 1
    if i < n and s[i] in '+-':
        sign = -1 if s[i] == '-' else 1
        i += 1
    
    # Digits
    num = 0
    while i < n and s[i].isdigit():
        num = num * 10 + int(s[i])
        i += 1
    
    return sign * num
```

**General Syntax (Expression Validation with Stack)**
```
stack = []
for char in expression:
    if char == '(':
        stack.append(char)
    elif char == ')':
        if not stack: return False  # Unmatched
        stack.pop()
return len(stack) == 0  # All matched
```

**Component Breakdown**

| Component | Description | atoi | Validation |
|-----------|-------------|------|------------|
| Whitespace | Skip leading spaces | Yes | N/A |
| Sign | Handle +/- | Yes | N/A |
| Digits | Accumulate number | Yes | N/A |
| Stack | Match brackets | No | Yes |

**Syntax Rules**

- `atoi` stops at first non-digit character
- `atoi` returns 0 for invalid input (C standard)
- Expression validation must handle nested parentheses
- Operator precedence requires parsing (not just stack matching)

**Constraints and Limitations**

- `atoi` does not detect overflow (behavior undefined in C)
- Expression validation for full arithmetic requires more than stack matching (precedence, associativity)
- Unicode digits require `str.isdigit()` rather than ASCII check

### Annotated Code Examples

**Example 1: Custom atoi Implementation**
```python
def my_atoi(s):
    """Implement atoi: convert string to integer."""
    i = 0
    n = len(s)
    
    # Skip leading whitespace
    while i < n and s[i] == ' ':
        i += 1
    
    # Handle sign
    sign = 1
    if i < n and s[i] in '+-':
        if s[i] == '-':
            sign = -1
        i += 1
    
    # Parse digits
    num = 0
    while i < n and s[i].isdigit():
        num = num * 10 + int(s[i])
        i += 1
    
    # Clamp to 32-bit integer range (optional)
    result = sign * num
    INT_MAX = 2**31 - 1
    INT_MIN = -2**31
    
    if result > INT_MAX:
        return INT_MAX
    if result < INT_MIN:
        return INT_MIN
    return result

# Test
print(my_atoi("42"))           # 42
print(my_atoi("   -42"))       # -42
print(my_atoi("4193 with"))    # 4193
print(my_atoi("words"))        # 0
```

**Expected Output:**
```
42
-42
4193
0
```

**Why This Output Occurs**: The parser skips whitespace, handles the sign, then accumulates digits. "4193 with" stops at the space. "words" has no digits, returning 0.

**Example 2: Valid Parentheses (Expression Validation)**
```python
def is_valid_parentheses(s):
    """Check if brackets are properly matched."""
    stack = []
    mapping = {')': '(', ']': '[', '}': '{'}
    
    for char in s:
        if char in '([{':
            stack.append(char)
        elif char in ')]}':
            if not stack or stack[-1] != mapping[char]:
                return False
            stack.pop()
    
    return len(stack) == 0

# Test
print(is_valid_parentheses("()"))           # True
print(is_valid_parentheses("()[]{}"))       # True
print(is_valid_parentheses("(]"))           # False
print(is_valid_parentheses("([)]"))         # False
```

**Expected Output:**
```
True
True
False
False
```

**Why This Output Occurs**: Opening brackets are pushed onto the stack. Closing brackets must match the top of the stack. "([)]" fails because the closing ']' doesn't match the top '('.

### Real-World Cases

- **User Input Parsing**: Converting form fields to numbers
- **Configuration Files**: Parsing numeric settings
- **Expression Evaluators**: Calculator applications
- **JSON/XML Parsing**: Extracting numeric values

### References

- UTEP - Strings in C++ Reference (Conversions) - http://www.pkirs.utep.edu/cis3355/Additional%20Coverage/Reference%20Links/strings3.htm 
- GitHub - String Mastery Toolkit - https://github.com/HimalayaMinds/string-mastery-toolkit 

---

## References (Consolidated)

- Wikipedia - Boyer–Moore majority vote algorithm - https://en.m.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm 
- Stack Overflow - Linear Time Voting Algorithm Discussion - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/780937/ 
- Educative - Longest Palindromic Substring with Manacher's Algorithm - https://www.educative.io/answers/longest-palindromic-substring-in-on-with-manachers-algorithm 
- GitHub - Manacher's Algorithm Tutorial - https://raw.githubusercontent.com/wingkwong/leetcode-the-hard-way/refs/heads/main/tutorials/strings/manachers-algorithm.md 
- arXiv - New Implementation of Manacher's Algorithm - http://web3.arxiv.org/pdf/2003.08211v1 
- Stack Overflow - Manacher's Algorithm Explanation - https://stackoverflow.com/questions/10468208/ 
- Stony Brook University - Majority Boyer-Moore Two-Pass Algorithm - https://www3.cs.stonybrook.edu/~pramod.ganapathi/doc/algorithms/Algo-AlgorithmicProblemSolving.pdf 
- Virtual Labs - String Operations References - https://ps-iiith.vlabs.ac.in/exp/string-operations/references.html 
- GitHub - String Mastery Toolkit - https://github.com/HimalayaMinds/string-mastery-toolkit 
- UTEP - Strings in C++ Reference - http://www.pkirs.utep.edu/cis3355/Additional%20Coverage/Reference%20Links/strings3.htm 
- GitHub - Boyer-Moore Soccer Analogy - https://raw.githubusercontent.com/kunigami/kunigami.github.io/master/blog/_posts/2021-03-06-boyer-moore-vote-algorithm.md 