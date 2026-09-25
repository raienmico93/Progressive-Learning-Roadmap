# DSA Arrays and Basic Linear Structures: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
An array is a fundamental data structure that stores a collection of elements of the same type in contiguous memory locations, where each element is accessible through an integer index.

**Technical Definition**
An array constitutes a built-in data structure denoting a collection of related quantities with common ordering properties, where each object must be of the same type and is accessible through an integer index representing an offset from the array's base address.

**Beginner-Friendly Explanation**
Think of an array as a row of numbered boxes in a warehouse. Each box holds one item of the same kind (like all integers or all strings), and because the boxes are lined up right next to each other, you can instantly find any box if you know its position number (index).

### Key Characteristics

- **Fixed Capacity**: Static arrays have a predetermined, immutable size at allocation time.
- **Homogeneous Elements**: All elements must be of the same data type.
- **Contiguous Memory**: Elements are stored in consecutive memory locations.
- **Random Access**: Any element can be accessed directly in constant time using its index.
- **Zero-Based Indexing**: Most modern languages (C, C++, Java, Python) index the first element at position 0.

### Prerequisites

- Basic understanding of variables and data types
- Familiarity with memory concepts (addresses, allocation)
- Knowledge of loops and conditional statements
- Understanding of functions and parameters

### Related Programming Areas

- **Algorithm Analysis**: Big O notation and complexity analysis
- **Data Structures**: Stacks, queues, hash tables, matrices
- **Memory Management**: Static vs. dynamic allocation, garbage collection
- **Computer Architecture**: Cache efficiency, memory hierarchy

### Core Concepts / Features

| Concept | Description |
|---------|-------------|
| Static Arrays | Fixed-size arrays with compile-time allocation |
| Dynamic Arrays | Resizable arrays with automatic memory management |
| Multidimensional Arrays | Arrays with multiple indices (matrices, tensors) |
| Memory Layout | Row-major vs. column-major ordering |
| Array Operations | Traversal, insertion, deletion, searching, updating |
| Complexity Analysis | Time and space complexity of array operations |

---

## 1. Static Arrays and Memory Allocation

### Definitions

**Core Definition**
A static array is an array whose size is fixed at compile time or declaration, with memory allocated on the stack or in the data segment.

**Technical Definition**
Static arrays have their memory allocated at compile-time or at the beginning of program execution, with the size being a constant expression that cannot be modified during runtime.

**Beginner-Friendly Explanation**
A static array is like reserving a specific number of parking spots in advance. Once you choose how many spots you need, that number cannot change—you either use them or leave them empty.

### Purposes (All begin with "To")

- **To** provide predictable memory usage with no runtime allocation overhead
- **To** enable fast access to elements through direct memory addressing
- **To** allow the compiler to optimize memory layout and access patterns
- **To** ensure data persistence throughout the program's lifetime when declared globally

### Syntax Rules and Structure

**General Syntax (C/C++/Java)**
```c
// C/C++ syntax
type arrayName[size];           // Declaration
type arrayName[size] = {values}; // Declaration with initialization

// Java syntax
type[] arrayName = new type[size]; // Allocation
type[] arrayName = {values};       // Initialization
```

**Component Breakdown**

| Component | Description | Example |
|-----------|-------------|---------|
| `type` | The data type of elements (int, float, char, etc.) | `int` |
| `arrayName` | Identifier for the array variable | `scores` |
| `size` | Positive integer constant expression | `5` |
| `values` | Comma-separated initial values | `{90, 85, 78}` |

**Syntax Rules**

- The size must be a compile-time constant expression in C (non-VLA)
- Array indices range from 0 to size-1
- Accessing elements outside this range is undefined behavior in C/C++
- In Java, arrays are objects allocated on the heap, while local array references are on the stack

**Constraints and Limitations**

- Size cannot be changed after declaration
- Wasted memory if allocated size exceeds usage
- Potential stack overflow for very large arrays
- No bounds checking in C/C++ (Java provides bounds checking)

### Annotated Code Examples

**Example 1: Static Array Declaration and Initialization (C++)**
```cpp
#include <iostream>
using namespace std;

int main() {
    // Declare a static array of 5 integers
    // Memory is allocated at compile-time on the stack
    int scores[5];
    
    // Initialize elements individually
    scores[0] = 90;  // First element (index 0)
    scores[1] = 85;  // Second element (index 1)
    scores[2] = 78;  // Third element (index 2)
    scores[3] = 92;  // Fourth element (index 3)
    scores[4] = 88;  // Fifth element (index 4)
    
    // Access and print all elements using a loop
    // The loop variable i serves as the index
    for (int i = 0; i < 5; i++) {
        cout << "scores[" << i << "] = " << scores[i] << endl;
    }
    
    return 0;
}
```

**Expected Output:**
```
scores[0] = 90
scores[1] = 85
scores[2] = 78
scores[3] = 92
scores[4] = 88
```

**Why This Output Occurs**: The array occupies 5 consecutive memory locations (typically 20 bytes for 5 integers on a 32-bit system). Each assignment writes a value to a specific offset from the base address. The loop iterates from 0 to 4, accessing each element by computing `base_address + (index * element_size)`.

**Example 2: Static Array with Initialization (Java)**
```java
public class StaticArrayExample {
    public static void main(String[] args) {
        // Declare and initialize a static array in one statement
        // The size is inferred from the number of values
        int[] temperatures = {72, 68, 75, 80, 77};
        
        // Access elements by index
        // Java performs bounds checking at runtime
        int firstTemp = temperatures[0];  // First element
        int lastTemp = temperatures[temperatures.length - 1]; // Last element
        
        System.out.println("First temperature: " + firstTemp);
        System.out.println("Last temperature: " + lastTemp);
        System.out.println("Array length: " + temperatures.length);
        
        // Modify an element
        temperatures[2] = 70;  // Change the third element
        System.out.println("Updated third temperature: " + temperatures[2]);
    }
}
```

**Expected Output:**
```
First temperature: 72
Last temperature: 77
Array length: 5
Updated third temperature: 70
```

**Why This Output Occurs**: Java arrays are objects with a `length` field. The array `temperatures` is allocated on the heap, and the reference is stored in the local variable. Index access triggers bounds checking; accessing `temperatures[5]` would throw `ArrayIndexOutOfBoundsException`.

### Real-World Cases

- **Embedded Systems**: Static arrays for sensor readings with fixed buffer sizes
- **Game Development**: Static arrays for fixed-size inventories or level data
- **Data Processing**: Fixed-size buffers for network packet handling
- **Matrix Operations**: Mathematical matrices with known dimensions

### References

- Microsoft Learn - Declaring and Indexing Arrays - https://learn.microsoft.com/th-th/previous-versions/visualstudio/visual-studio-6.0/aa293547(v=vs.60)

---

## 2. Dynamic Arrays and Underlying Mechanisms

### Definitions

**Core Definition**
A dynamic array (also called array list, growable array, or vector) is a data structure that wraps a static array and automatically resizes when more space is needed.

**Technical Definition**
A dynamic array maintains an underlying fixed-size array along with a logical size counter and capacity counter, performing geometric expansion (typically doubling) when capacity is exceeded.

**Beginner-Friendly Explanation**
Imagine a parking lot that can automatically expand when it gets full. You start with a certain number of spots, and when all are occupied, a larger lot is built next door, all cars are moved over, and you continue parking. You rarely have to think about the expansion—it just happens.

### Purposes (All begin with "To")

- **To** provide the fast O(1) access of arrays while allowing flexible sizing
- **To** eliminate the need to know the exact number of elements in advance
- **To** amortize the cost of resizing across many append operations
- **To** abstract away manual memory management from the programmer

### Syntax Rules and Structure

**General Syntax (C++ std::vector)**
```cpp
#include <vector>

std::vector<type> name;              // Empty vector
std::vector<type> name(size);        // Vector with size elements
std::vector<type> name = {values};   // Initializer list
name.push_back(value);               // Append element
name[i];                             // Access element
name.size();                         // Get number of elements
name.capacity();                     // Get current capacity
```

**Component Breakdown**

| Component | Description | Example |
|-----------|-------------|---------|
| `type` | Element data type | `int` |
| `name` | Vector variable name | `numbers` |
| `push_back` | Method to append element | `numbers.push_back(42)` |
| `size()` | Returns logical length | `numbers.size()` |
| `capacity()` | Returns physical storage size | `numbers.capacity()` |

**Syntax Rules**

- Dynamic arrays manage memory automatically
- Elements are stored contiguously like static arrays
- Reallocation invalidates all iterators and references
- Growth factor is typically 2 (doubling strategy)

**Constraints and Limitations**

- Individual resizes are O(n) when copying is required
- Amortized O(1) for append operations
- Memory overhead from unused capacity
- Not available in all languages (e.g., C lacks built-in dynamic arrays)

### Annotated Code Examples

**Example 1: C++ Dynamic Array Growth**
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // Create an empty vector
    // Initial capacity is implementation-defined (often 0 or small)
    vector<int> numbers;
    
    cout << "Initial size: " << numbers.size() << endl;
    cout << "Initial capacity: " << numbers.capacity() << endl;
    
    // Append elements one by one
    // push_back handles resizing automatically
    for (int i = 1; i <= 10; i++) {
        numbers.push_back(i * 10);
        
        // Print size and capacity after each append
        // Observe when capacity changes
        cout << "After adding " << i * 10 
             << ": size=" << numbers.size() 
             << ", capacity=" << numbers.capacity() << endl;
    }
    
    // Access elements using array notation
    cout << "\nFirst element: " << numbers[0] << endl;
    cout << "Last element: " << numbers[numbers.size() - 1] << endl;
    
    return 0;
}
```

**Expected Output** (typical):
```
Initial size: 0
Initial capacity: 0
After adding 10: size=1, capacity=1
After adding 20: size=2, capacity=2
After adding 30: size=3, capacity=4
After adding 40: size=4, capacity=4
After adding 50: size=5, capacity=8
After adding 60: size=6, capacity=8
After adding 70: size=7, capacity=8
After adding 80: size=8, capacity=8
After adding 90: size=9, capacity=16
After adding 100: size=10, capacity=16

First element: 10
Last element: 100
```

**Why This Output Occurs**: The vector doubles its capacity when `size == capacity`. When adding the third element, capacity grows from 2 to 4. The capacity changes at sizes 1, 3, 5, and 9 in this example. This doubling strategy ensures amortized O(1) append cost.

**Example 2: Java ArrayList Operations**
```java
import java.util.ArrayList;

public class DynamicArrayExample {
    public static void main(String[] args) {
        // Create an ArrayList of Strings
        // Initial capacity defaults to 10 if not specified
        ArrayList<String> names = new ArrayList<>();
        
        // Add elements (enqueue equivalent)
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");
        
        System.out.println("After adds: " + names);
        System.out.println("Size: " + names.size());
        
        // Insert at specific index (shifts subsequent elements)
        names.add(1, "Diana");  // Insert at index 1
        System.out.println("After insert at index 1: " + names);
        
        // Remove element (shifts subsequent elements left)
        names.remove(0);  // Remove first element
        System.out.println("After removing index 0: " + names);
        
        // Access element by index
        System.out.println("Element at index 1: " + names.get(1));
    }
}
```

**Expected Output:**
```
After adds: [Alice, Bob, Charlie]
Size: 3
After insert at index 1: [Alice, Diana, Bob, Charlie]
After removing index 0: [Diana, Bob, Charlie]
Element at index 1: Bob
```

**Why This Output Occurs**: `add(index, element)` shifts all elements from that index onward to the right. `remove(index)` shifts all subsequent elements left to fill the gap. These operations are O(n) because they require element movement.

### Real-World Cases

- **Web Servers**: Dynamically growing lists of active connections
- **Data Analytics**: Buffers that accumulate streaming data
- **Text Editors**: Dynamic arrays of characters for document content
- **Game Engines**: Entity lists that grow as objects are spawned

### References

- Brown University CS18 - Dynamic Arrays (ArrayLists) - https://brown-csci0200.github.io/assets/lectures/11arraylists2/11dynArrays.pdf
- IISc DS286 - Linear Lists - https://cds.iisc.ac.in/wp-content/uploads/DS286.AUG2016.L4-5.Lists_.pdf

---

## 3. One-Dimensional and Multidimensional Arrays

### Definitions

**Core Definition**
A one-dimensional array uses a single index to access elements in a linear sequence. A multidimensional array uses multiple indices to organize elements in a grid or higher-dimensional structure.

**Technical Definition**
A multidimensional array of dimension n is an array where each element is itself an array of dimension n-1, allowing access through n indices, with the total memory being the product of all dimension sizes.

**Beginner-Friendly Explanation**
A one-dimensional array is like a single row of lockers. A two-dimensional array is like a grid of lockers arranged in rows and columns—you need both a row number and a column number to find a specific locker.

### Purposes (All begin with "To")

- **To** represent naturally tabular or matrix data (spreadsheets, images, game boards)
- **To** organize hierarchical relationships between data elements
- **To** enable efficient mathematical operations on matrices
- **To** provide intuitive indexing for multidimensional data

### Syntax Rules and Structure

**General Syntax (C/C++/Java)**
```c
// C/C++: 2D array declaration
type arrayName[rows][columns];

// Java: 2D array (array of arrays)
type[][] arrayName = new type[rows][columns];

// Access: arrayName[row][column]
```

**Component Breakdown**

| Component | Description | Example |
|-----------|-------------|---------|
| `rows` | Number of rows (first dimension) | `3` |
| `columns` | Number of columns (second dimension) | `4` |
| `arrayName[row][col]` | Access syntax | `matrix[1][2]` |
| `arrayName.length` | Number of rows (Java) | `matrix.length` |
| `arrayName[0].length` | Number of columns in first row | `matrix[0].length` |

**Syntax Rules**

- In C/C++, 2D arrays are allocated as a single contiguous block
- In Java, 2D arrays are arrays of arrays (jagged arrays are possible)
- Row-major order: rightmost index changes fastest
- Column-major order: leftmost index changes fastest

**Constraints and Limitations**

- Multidimensional arrays consume significant memory (product of dimensions)
- Row-major languages (C, C++, Python) access `[row][col]` efficiently for row traversal
- Column-major languages (Fortran, MATLAB, R) access `[col][row]` efficiently for column traversal
- Jagged arrays (Java) can have rows of different lengths

### Annotated Code Examples

**Example 1: 2D Array Traversal and Sum (Java)**
```java
public class MatrixSum {
    public static void main(String[] args) {
        // Create a 3x4 matrix (3 rows, 4 columns)
        int[][] matrix = {
            {1, 2, 3, 4},    // Row 0
            {5, 6, 7, 8},    // Row 1
            {9, 10, 11, 12}  // Row 2
        };
        
        int totalSum = 0;
        
        // Outer loop iterates over rows
        for (int row = 0; row < matrix.length; row++) {
            // Inner loop iterates over columns in current row
            for (int col = 0; col < matrix[row].length; col++) {
                // Add each element to the running total
                totalSum += matrix[row][col];
            }
        }
        
        System.out.println("Matrix dimensions: " + 
            matrix.length + " x " + matrix[0].length);
        System.out.println("Sum of all elements: " + totalSum);
        
        // Access specific element
        System.out.println("Element at [1][2]: " + matrix[1][2]);
    }
}
```

**Expected Output:**
```
Matrix dimensions: 3 x 4
Sum of all elements: 78
Element at [1][2]: 7
```

**Why This Output Occurs**: The nested loops traverse every element. The sum 1+2+...+12 = 78. The element at row 1, column 2 is 7 (third element in second row). In Java, `matrix[1]` returns the second row array, and `[2]` accesses its third element.

**Example 2: 3D Array Coordinates (C++)**
```cpp
#include <iostream>
using namespace std;

int main() {
    // 3D array: 2 layers, 3 rows, 4 columns
    int cube[2][3][4];
    
    // Initialize with layer*100 + row*10 + col
    for (int layer = 0; layer < 2; layer++) {
        for (int row = 0; row < 3; row++) {
            for (int col = 0; col < 4; col++) {
                cube[layer][row][col] = layer * 100 + row * 10 + col;
            }
        }
    }
    
    // Access and display specific coordinates
    cout << "cube[0][1][2] = " << cube[0][1][2] << endl;  // Layer 0, row 1, col 2
    cout << "cube[1][2][3] = " << cube[1][2][3] << endl;  // Layer 1, row 2, col 3
    
    // Calculate total memory usage
    int totalElements = 2 * 3 * 4;
    cout << "Total elements: " << totalElements << endl;
    cout << "Memory (int = 4 bytes): " << totalElements * 4 << " bytes" << endl;
    
    return 0;
}
```

**Expected Output:**
```
cube[0][1][2] = 12
cube[1][2][3] = 123
Total elements: 24
Memory (int = 4 bytes): 96 bytes
```

**Why This Output Occurs**: The 3D array is stored in row-major order in C++. The total memory is calculated as 2 × 3 × 4 × 4 bytes = 96 bytes. The value at `cube[0][1][2]` is computed as 0×100 + 1×10 + 2 = 12.

### Real-World Cases

- **Image Processing**: 2D arrays for pixel intensity values
- **Game Boards**: Chess, tic-tac-toe, and other grid-based games
- **Scientific Computing**: Matrices for linear algebra operations
- **Machine Learning**: Tensors for neural network weights

### References

- Carnegie Mellon University - Two-dimensional Arrays - http://www.cs.cmu.edu/afs/andrew/course/15/110-m12/www/applications/ln/Unit06PtAPtB.pdf
- CRAN Reticulate - Arrays in R and Python (Row/Column Major) - https://mirrors.cstcloud.cn/CRAN/web/packages/reticulate/vignettes/arrays.html

---

## 4. Array Indexing and Zero-Based Indexing

### Definitions

**Core Definition**
Array indexing is the mechanism by which elements are identified and accessed using integer subscripts. Zero-based indexing assigns the first element index 0.

**Technical Definition**
Zero-based indexing computes element addresses as `base_address + (index * element_size)`, where the first element has index 0, making the offset calculation direct and efficient.

**Beginner-Friendly Explanation**
If you're standing at the beginning of a row of boxes, the first box is at position 0, not position 1. This means the "nth" box is actually at position n-1.

### Purposes (All begin with "To")

- **To** provide direct memory address computation for O(1) element access
- **To** eliminate off-by-one errors in address arithmetic
- **To** align with hardware memory addressing conventions
- **To** simplify loop conditions and pointer arithmetic

### Syntax Rules and Structure

**General Syntax**
```c
// Access element at index i
element = arrayName[i];

// Valid indices: 0 to size-1
for (int i = 0; i < size; i++) {
    // Access arrayName[i]
}
```

**Component Breakdown**

| Component | Description | Example |
|-----------|-------------|---------|
| `i` | Index variable (0 ≤ i < size) | `2` |
| `arrayName[i]` | Element at position i | `scores[2]` |
| `size` | Number of elements | `5` |
| Valid range | [0, size-1] | [0, 4] for size 5 |

**Syntax Rules**

- First element is at index 0
- Last element is at index size-1
- Address of element i = base + (i × element_size)
- Accessing out-of-bounds is undefined behavior (C/C++) or throws exception (Java)

**Constraints and Limitations**

- Languages differ: Fortran uses 1-based indexing
- Some languages allow custom lower bounds (Pascal, Ada)
- Python supports negative indexing (from the end)
- Off-by-one errors are a common source of bugs

### Annotated Code Examples

**Example 1: Zero-Based Access Demonstration (C++)**
```cpp
#include <iostream>
using namespace std;

int main() {
    // Array of 5 elements
    int data[5] = {50, 42, 85, 71, 99};
    
    // Assume array starts at memory address 100
    // Each int is 4 bytes
    // data[0] is at 100
    // data[1] is at 104
    // data[2] is at 108
    // data[3] is at 112
    // data[4] is at 116
    
    cout << "Index | Value | Address (base=100, size=4)" << endl;
    cout << "------|-------|---------------------------" << endl;
    
    for (int i = 0; i < 5; i++) {
        // Simulating address calculation
        int simulatedAddress = 100 + (i * 4);
        cout << "  " << i << "   |  " << data[i] 
             << "   | " << simulatedAddress << endl;
    }
    
    // Demonstrating why 0-based makes sense:
    // data[i] is at address: base + (i * element_size)
    // This formula works perfectly with i starting at 0
    
    return 0;
}
```

**Expected Output:**
```
Index | Value | Address (base=100, size=4)
------|-------|---------------------------
  0   |  50   | 100
  1   |  42   | 104
  2   |  85   | 108
  3   |  71   | 112
  4   |  99   | 116
```

**Why This Output Occurs**: The address of each element is computed as `100 + (index × 4)`. With zero-based indexing, the first element is at the base address (offset 0). If indexing started at 1, the formula would need adjustment: `base + ((index-1) × size)`, adding an unnecessary subtraction.

**Example 2: Negative Indexing in Python**
```python
# Python lists support negative indexing
data = [50, 42, 85, 71, 99]

print(f"data[0] = {data[0]}")    # First element
print(f"data[4] = {data[4]}")    # Last element (index size-1)
print(f"data[-1] = {data[-1]}")  # Last element (negative indexing)
print(f"data[-2] = {data[-2]}")  # Second to last element

# Slicing with indices
print(f"data[1:4] = {data[1:4]}")  # Elements at indices 1, 2, 3
print(f"data[:3] = {data[:3]}")    # First three elements
print(f"data[2:] = {data[2:]}")    # Elements from index 2 onward
```

**Expected Output:**
```
data[0] = 50
data[4] = 99
data[-1] = 99
data[-2] = 71
data[1:4] = [42, 85, 71]
data[:3] = [50, 42, 85]
data[2:] = [85, 71, 99]
```

**Why This Output Occurs**: Python's negative indexing starts from -1 (last element) and counts backward. Slicing `data[start:end]` includes elements from `start` up to but not including `end`. This differs from C++ where manual bounds checking is required.

### Real-World Cases

- **Buffer Management**: Network packet buffers indexed from 0
- **String Processing**: Character arrays with zero-based access
- **Pixel Manipulation**: Image arrays where (0,0) is top-left
- **Database Records**: Row indices in result sets

### References

- Microsoft Learn - Array Indexing and Lower Bounds - https://learn.microsoft.com/th-th/previous-versions/visualstudio/visual-studio-6.0/aa293547(v=vs.60)
- Carnegie Mellon University - Arrays in Memory - https://www.cs.cmu.edu/~15110-n15/lectures/unit06-1-Arrays_handout.pdf

---

## 5. Memory Layout: Contiguous Memory, Row-Major and Column-Major Order

### Definitions

**Core Definition**
Contiguous memory means array elements are stored in consecutive memory addresses. Row-major and column-major orders define how multidimensional array elements map to this linear memory space.

**Technical Definition**
In row-major order, the rightmost dimension index varies fastest with memory address. In column-major order, the leftmost dimension index varies fastest.

**Beginner-Friendly Explanation**
Imagine writing a book. Row-major order writes all words in the first row, then the second row. Column-major order writes all words in the first column, then the second column.

### Purposes (All begin with "To")

- **To** enable efficient linear traversal of multidimensional data
- **To** optimize memory access patterns for cache efficiency
- **To** ensure compatibility with foreign function interfaces and libraries
- **To** provide predictable data layout for serialization

### Syntax Rules and Structure

**General Formula (Row-Major)**
```
address = base + (row * num_cols + col) * element_size
```

**General Formula (Column-Major)**
```
address = base + (col * num_rows + row) * element_size
```

**Component Breakdown**

| Component | Description | Row-Major | Column-Major |
|-----------|-------------|-----------|--------------|
| `base` | Starting memory address | ✓ | ✓ |
| `row` | Row index | Multiplied by cols | Added directly |
| `col` | Column index | Added directly | Multiplied by rows |
| Languages | | C, C++, Java, Python | Fortran, MATLAB, R |

**Syntax Rules**

- Row-major (C-style): `array[row][col]`, rightmost index varies fastest
- Column-major (Fortran-style): `array(col, row)`, leftmost index varies fastest
- Python/NumPy defaults to row-major (C_CONTIGUOUS)
- CLR (.NET) uses row-major order for multidimensional arrays

**Constraints and Limitations**

- Incompatible layouts require data transposition (performance cost)
- Cache performance depends on traversal order matching memory layout
- Foreign function interfaces (FFI) must account for layout differences

### Annotated Code Examples

**Example 1: Row-Major Layout (C++)**
```cpp
#include <iostream>
using namespace std;

int main() {
    // 2x3 matrix in row-major order
    int matrix[2][3] = {
        {1, 2, 3},  // Row 0
        {4, 5, 6}   // Row 1
    };
    
    // Memory layout: 1, 2, 3, 4, 5, 6
    // Address increases: matrix[0][0], [0][1], [0][2], [1][0], [1][1], [1][2]
    
    cout << "Row-major memory order:" << endl;
    for (int row = 0; row < 2; row++) {
        for (int col = 0; col < 3; col++) {
            cout << matrix[row][col] << " ";
        }
    }
    cout << endl;
    
    // Prove contiguous layout using pointer arithmetic
    int* flat = &matrix[0][0];
    cout << "Linear access: ";
    for (int i = 0; i < 6; i++) {
        cout << flat[i] << " ";  // Accesses elements in memory order
    }
    cout << endl;
    
    return 0;
}
```

**Expected Output:**
```
Row-major memory order:
1 2 3 4 5 6 
Linear access: 1 2 3 4 5 6 
```

**Why This Output Occurs**: In row-major order, all elements of row 0 come first (1, 2, 3), followed by row 1 (4, 5, 6). The pointer `flat` traverses the contiguous memory directly.

**Example 2: Column-Major Layout (R/Python Comparison)**
```python
# Python with NumPy defaults to row-major (C order)
import numpy as np

# Create 2x3 array
arr = np.array([[1, 2, 3], [4, 5, 6]])

print("NumPy array (C order):")
print(arr)
print(f"Memory order (flattened): {arr.flatten(order='C')}")
print(f"Fortran order (column-major): {arr.flatten(order='F')}")

# For column-major, the sequence would be: 1, 4, 2, 5, 3, 6
```

**Expected Output:**
```
NumPy array (C order):
[[1 2 3]
 [4 5 6]]
Memory order (flattened): [1 2 3 4 5 6]
Fortran order (column-major): [1 4 2 5 3 6]
```

**Why This Output Occurs**: NumPy's default C order flattens row by row (1,2,3,4,5,6). For column-major (Fortran) order, it flattens column by column: column 0 gives (1,4), column 1 gives (2,5), column 2 gives (3,6).

### Real-World Cases

- **BLAS/LAPACK**: Linear algebra libraries often use column-major (Fortran heritage)
- **Image Processing**: Row-major (C-style) for row-wise filter operations
- **Database Storage**: Row-major for record-oriented access
- **Scientific Data**: Column-major for dimension-oriented access

### References

- CRAN Reticulate - Addressing Array Layout Issues - https://mirrors.cstcloud.cn/CRAN/web/packages/reticulate/vignettes/arrays.html
- Microsoft Learn - CLR Array Storage Order - https://learn.microsoft.com/vi-vn/archive/msdn-technet-forums/0c00d9af-4c10-4451-b7eb-972de0944ff8

---

## 6. Time and Space Complexity (Big O Notation for Array Operations)

### Definitions

**Core Definition**
Big O notation describes the upper bound of an algorithm's time or space requirements as a function of input size.

**Technical Definition**
For functions f(n) and g(n), f(n) = O(g(n)) if there exists a constant c > 0 such that f(n) ≤ c × g(n) for all sufficiently large n.

**Beginner-Friendly Explanation**
Big O tells you how an algorithm's speed changes as the data grows. O(1) means it stays fast regardless of size. O(n) means doubling the data doubles the time.

### Purposes (All begin with "To")

- **To** compare algorithm efficiency independent of hardware
- **To** predict performance as input size scales
- **To** identify bottlenecks in data structure operations
- **To** guide selection of appropriate data structures

### Complexity Table for Array Operations

| Operation | Static Array | Dynamic Array | Explanation |
|-----------|-------------|---------------|-------------|
| **Access** | O(1) | O(1) | Direct address computation |
| **Search (linear)** | O(n) | O(n) | Must inspect each element |
| **Search (binary, sorted)** | O(log n) | O(log n) | Divide and conquer |
| **Insert (end)** | N/A (fixed) | O(1) amortized | Occasional O(n) resize |
| **Insert (middle)** | O(n) | O(n) | Requires shifting |
| **Delete (end)** | O(1) | O(1) | Just decrement size |
| **Delete (middle)** | O(n) | O(n) | Requires shifting |
| **Resize** | N/A | O(n) | Copy all elements |

### Syntax Rules and Structure

**General Big O Analysis**
```
// O(1) - constant: independent of n
element = array[i];

// O(n) - linear: proportional to n
for (int i = 0; i < n; i++) { sum += array[i]; }

// O(n²) - quadratic: nested loops over n
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++) { ... }
```

**Component Breakdown**

| Complexity | Name | Growth (n→2n) | Example |
|------------|------|---------------|---------|
| O(1) | Constant | Same | Array access |
| O(log n) | Logarithmic | +1 step | Binary search |
| O(n) | Linear | 2× | Linear search |
| O(n log n) | Linearithmic | 2× + small | Efficient sorting |
| O(n²) | Quadratic | 4× | Nested loops |

### Annotated Code Examples

**Example 1: O(1) vs O(n) Array Operations**
```cpp
#include <iostream>
#include <vector>
#include <chrono>
using namespace std;
using namespace std::chrono;

int main() {
    // O(1) access: any index accessed in same time
    vector<int> arr(1000000, 42);  // 1 million elements
    
    auto start = high_resolution_clock::now();
    int x = arr[500000];  // Access middle element
    auto end = high_resolution_clock::now();
    
    auto duration = duration_cast<nanoseconds>(end - start).count();
    cout << "O(1) access time: " << duration << " ns" << endl;
    cout << "Value: " << x << endl;
    
    // O(n) search: time proportional to position
    vector<int> unsorted = {5, 3, 8, 1, 9, 2, 7, 4, 6};
    int target = 6;  // Last element (worst case)
    
    start = high_resolution_clock::now();
    for (int i = 0; i < unsorted.size(); i++) {
        if (unsorted[i] == target) {
            break;  // Found at last position
        }
    }
    end = high_resolution_clock::now();
    
    duration = duration_cast<nanoseconds>(end - start).count();
    cout << "O(n) search time: " << duration << " ns" << endl;
    
    return 0;
}
```

**Expected Output** (timing varies):
```
O(1) access time: 50 ns
Value: 42
O(n) search time: 200 ns
```

**Why This Output Occurs**: Array access is constant time regardless of index because the address is computed directly. Linear search time grows with position—finding the last element requires checking all previous elements.

**Example 2: Amortized O(1) for Dynamic Array Append**
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> vec;
    
    // Track total operations across many appends
    // Even though individual resizes are O(n), average is O(1)
    
    for (int i = 0; i < 20; i++) {
        size_t oldCap = vec.capacity();
        vec.push_back(i);
        size_t newCap = vec.capacity();
        
        cout << "push_back(" << i << "): size=" << vec.size();
        if (oldCap != newCap) {
            cout << " [RESIZE: " << oldCap << " -> " << newCap << "]";
        }
        cout << endl;
    }
    
    cout << "\nTotal elements: " << vec.size() << endl;
    cout << "Final capacity: " << vec.capacity() << endl;
    cout << "Wasted capacity: " << vec.capacity() - vec.size() << endl;
    
    return 0;
}
```

**Expected Output:**
```
push_back(0): size=1 [RESIZE: 0 -> 1]
push_back(1): size=2 [RESIZE: 1 -> 2]
push_back(2): size=3 [RESIZE: 2 -> 4]
push_back(3): size=4
push_back(4): size=5 [RESIZE: 4 -> 8]
push_back(5): size=6
push_back(6): size=7
push_back(7): size=8
push_back(8): size=9 [RESIZE: 8 -> 16]
push_back(9): size=10
push_back(10): size=11
push_back(11): size=12
push_back(12): size=13
push_back(13): size=14
push_back(14): size=15
push_back(15): size=16
push_back(16): size=17 [RESIZE: 16 -> 32]
push_back(17): size=18
push_back(18): size=19
push_back(19): size=20

Total elements: 20
Final capacity: 32
Wasted capacity: 12
```

**Why This Output Occurs**: Resizes occur at sizes 1, 2, 3, 5, 9, 17 (powers of 2 plus 1). Each resize doubles capacity, so resizes become exponentially less frequent. Total resize work across n appends is O(n), giving O(1) amortized cost per append.

### Real-World Cases

- **Database Query Optimization**: Choosing between indexed access (O(1)) and table scan (O(n))
- **API Rate Limiting**: O(1) hash lookups for token validation
- **Memory Management**: Trade-off between wasted capacity and resize frequency
- **Algorithm Selection**: QuickSort O(n log n) vs Bubble Sort O(n²)

### References

- Cornell University - Asymptotic Complexity - https://www.cs.cornell.edu/courses/cs211/2002sp/Lectures/Complexity.pdf
- CMU - Order of Complexity - http://www.cs.cmu.edu/%7Etcortina/15110f11/Unit04PtC.pdf

---

## 7. Array Traversal

### Definitions

**Core Definition**
Array traversal is the process of visiting each element of an array exactly once, typically to perform an operation on each element.

**Technical Definition**
Traversal involves iterating through array indices from a starting index to an ending index, accessing each element in sequence.

**Beginner-Friendly Explanation**
Traversal is like walking through every house on a street to deliver flyers—you visit each one systematically without skipping any.

### Purposes (All begin with "To")

- **To** process every element (sum, find max, transform)
- **To** search for a specific value
- **To** print or display all elements
- **To** validate data integrity across the array

### Syntax Rules and Structure

**General Syntax (C++/Java/Python)**
```cpp
// C++ for loop
for (int i = 0; i < size; i++) {
    // Access arr[i]
}
```
```java
// Java enhanced for
for (int element : arr) {
    // Use element
}
```
```py
# Python
for element in arr:
    # Use element
```

**Component Breakdown**

| Component | Description | Example |
|-----------|-------------|---------|
| `i` | Loop counter/index | `int i = 0` |
| `size` | Number of elements | `arr.length` |
| `i < size` | Continuation condition | Loop while valid |
| `i++` | Increment | Move to next |

### Annotated Code Examples

**Example 1: Finding Maximum Value (C++)**
```cpp
#include <iostream>
#include <algorithm>
using namespace std;

int main() {
    int data[] = {42, 17, 93, 58, 64, 21, 88};
    int size = 7;
    
    // Initialize max with first element
    // (not 0, in case all values are negative)
    int maxVal = data[0];
    
    // Traverse from second element
    for (int i = 1; i < size; i++) {
        // Update max if current element is larger
        if (data[i] > maxVal) {
            maxVal = data[i];
        }
    }
    
    cout << "Maximum value: " << maxVal << endl;
    
    // Alternative: use std::max_element
    int* maxPtr = max_element(data, data + size);
    cout << "Maximum (algorithm): " << *maxPtr << endl;
    
    return 0;
}
```

**Expected Output:**
```
Maximum value: 93
Maximum (algorithm): 93
```

**Why This Output Occurs**: The loop visits each element exactly once, comparing to the running maximum. 93 is the largest value in the array.

### Real-World Cases

- **Data Validation**: Checking all form fields for required values
- **Image Processing**: Applying filters to every pixel
- **Financial Calculations**: Summing all transactions
- **Sensor Networks**: Reading all sensor values

### References

- University of Miskolc - Elementary Dynamic Sets and Array Data Structure - https://web.uni-miskolc.hu/~matha/4_Elementary_Dynamic_Sets.pdf

---

## 8. Insertion

### Definitions

**Core Definition**
Insertion is the operation of adding a new element to an array at a specified position.

**Technical Definition**
Insertion into a fixed-size array requires shifting all elements at and after the target position one position to the right to make room, then writing the new value.

**Beginner-Friendly Explanation**
Insertion is like squeezing a new person into an already-full row of seats—everyone from that point onward has to scoot over one seat.

### Purposes (All begin with "To")

- **To** add new data while maintaining sorted order
- **To** build arrays incrementally
- **To** implement insertion-based sorting algorithms
- **To** maintain ordered collections

### Syntax Rules and Structure

**General Syntax (Pseudocode)**
```
insert(array, position, value):
    if position < 0 or position > length:
        error "Invalid position"
    if length == capacity:
        resize array
    for i from length-1 down to position:
        array[i+1] = array[i]
    array[position] = value
    length = length + 1
```

### Annotated Code Examples

**Example 1: Insertion into Sorted Array (C++)**
```cpp
#include <iostream>
using namespace std;

void insertSorted(int arr[], int& size, int capacity, int value) {
    if (size >= capacity) {
        cout << "Array full, cannot insert" << endl;
        return;
    }
    
    // Find insertion position (start from end)
    int i = size - 1;
    while (i >= 0 && arr[i] > value) {
        arr[i + 1] = arr[i];  // Shift right
        i--;
    }
    
    // Insert at correct position
    arr[i + 1] = value;
    size++;
}

int main() {
    int arr[10] = {10, 20, 30, 50};  // Sorted, size=4
    int size = 4;
    int capacity = 10;
    
    cout << "Before insertion: ";
    for (int i = 0; i < size; i++) cout << arr[i] << " ";
    cout << endl;
    
    insertSorted(arr, size, capacity, 40);
    
    cout << "After inserting 40: ";
    for (int i = 0; i < size; i++) cout << arr[i] << " ";
    cout << endl;
    
    return 0;
}
```

**Expected Output:**
```
Before insertion: 10 20 30 50 
After inserting 40: 10 20 30 40 50 
```

**Why This Output Occurs**: The algorithm finds 40 belongs between 30 and 50. Element 50 shifts right, then 40 is placed at index 3. Insertion at a position is O(n) due to shifting.

### Real-World Cases

- **Leaderboards**: Inserting new high scores in order
- **Database Indexing**: Maintaining sorted index entries
- **Scheduling**: Adding appointments in time order
- **Priority Queues**: Inserting based on priority

### References

- BITS Pilani - Dynamic Arrays and Insertion/Deletion - https://www.bits-pilani.ac.in/wp-content/uploads/DSA-L3.pdf

---

## 9. Deletion

### Definitions

**Core Definition**
Deletion is the operation of removing an element from an array, which requires shifting subsequent elements to fill the gap.

**Technical Definition**
Deletion at index i requires copying all elements from i+1 to length-1 to positions starting at i, then decrementing the length.

**Beginner-Friendly Explanation**
Deletion is like removing a person from a row of seats—everyone after that person scoots over one seat to close the gap.

### Purposes (All begin with "To")

- **To** remove obsolete or processed data
- **To** maintain dynamic collections
- **To** implement deletion-based algorithms
- **To** manage memory by removing unneeded elements

### Annotated Code Examples

**Example 1: Deletion from Array (Java)**
```java
public class DeletionExample {
    public static int[] deleteAt(int[] arr, int size, int index) {
        if (index < 0 || index >= size) {
            System.out.println("Invalid index");
            return arr;
        }
        
        // Shift elements left to fill gap
        for (int i = index; i < size - 1; i++) {
            arr[i] = arr[i + 1];
        }
        
        // Last position is now duplicate, logically ignored
        return arr;
    }
    
    public static void main(String[] args) {
        int[] arr = {10, 20, 30, 40, 50};
        int size = 5;
        
        System.out.print("Before: ");
        for (int i = 0; i < size; i++) System.out.print(arr[i] + " ");
        System.out.println();
        
        // Delete element at index 2 (value 30)
        deleteAt(arr, size, 2);
        size--;  // Reduce logical size
        
        System.out.print("After deleting index 2: ");
        for (int i = 0; i < size; i++) System.out.print(arr[i] + " ");
        System.out.println();
    }
}
```

**Expected Output:**
```
Before: 10 20 30 40 50 
After deleting index 2: 10 20 40 50 
```

**Why This Output Occurs**: Deleting index 2 (value 30) causes elements at indices 3 and 4 to shift left. The size decrements, effectively ignoring the last position.

### Real-World Cases

- **Task Management**: Removing completed tasks from a list
- **Memory Cleanup**: Removing invalidated cache entries
- **Game Development**: Removing destroyed entities
- **Inventory Systems**: Removing sold items

### References

- BITS Pilani - Array Insertion and Deletion - https://www.bits-pilani.ac.in/wp-content/uploads/DSA-L3.pdf

---

## 10. Searching

### Definitions

**Core Definition**
Searching is the operation of finding the position or existence of a specific value within an array.

**Technical Definition**
Search algorithms traverse array elements comparing each against a target key until a match is found or the array is exhausted.

**Beginner-Friendly Explanation**
Searching is like looking for a specific book on a shelf—you check each book until you find the one you want.

### Purposes (All begin with "To")

- **To** determine if a value exists in the collection
- **To** retrieve the position of a specific element
- **To** validate membership before performing operations
- **To** enable conditional processing based on presence

### Annotated Code Examples

**Example 1: Linear Search (C++)**
```cpp
#include <iostream>
using namespace std;

int linearSearch(int arr[], int size, int target) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == target) {
            return i;  // Return index if found
        }
    }
    return -1;  // Not found
}

int main() {
    int data[] = {42, 17, 93, 58, 64, 21, 88};
    int size = 7;
    
    int targets[] = {58, 100};
    
    for (int t : targets) {
        int result = linearSearch(data, size, t);
        if (result != -1) {
            cout << t << " found at index " << result << endl;
        } else {
            cout << t << " not found" << endl;
        }
    }
    
    return 0;
}
```

**Expected Output:**
```
58 found at index 3
100 not found
```

**Why This Output Occurs**: Linear search checks each element sequentially. 58 is found at index 3 (fourth element). 100 is not in the array, so -1 is returned after checking all elements. Worst case is O(n).

### Real-World Cases

- **Contact Lookup**: Finding a phone number by name
- **Inventory Check**: Determining if a product is in stock
- **Security**: Checking if a user is in an authorized list
- **Data Validation**: Verifying a value exists before processing

### References

- University of Miskolc - Linear Search in Arrays - https://web.uni-miskolc.hu/~matha/4_Elementary_Dynamic_Sets.pdf

---

## 11. Updating

### Definitions

**Core Definition**
Updating is the operation of modifying an existing element at a known index.

**Technical Definition**
Update is O(1) because it directly computes the address of the target element and writes the new value.

**Beginner-Friendly Explanation**
Updating is like changing the contents of a specific box after you already know its position.

### Purposes (All begin with "To")

- **To** modify existing data without changing structure
- **To** reflect state changes in elements
- **To** correct erroneous values
- **To** implement algorithms that transform data in place

### Annotated Code Examples

**Example 1: Array Update with Validation (Python)**
```python
def update_element(arr, index, new_value):
    """Update array element with bounds checking"""
    if index < 0 or index >= len(arr):
        raise IndexError(f"Index {index} out of bounds")
    
    old_value = arr[index]
    arr[index] = new_value
    return old_value

# Example usage
data = [10, 20, 30, 40, 50]

print(f"Before: {data}")
old = update_element(data, 2, 35)
print(f"After: {data}")
print(f"Replaced {old} with 35")

# Demonstrate in-place modification
data[0] *= 2  # Double first element
print(f"After doubling first element: {data}")
```

**Expected Output:**
```
Before: [10, 20, 30, 40, 50]
After: [10, 20, 35, 40, 50]
Replaced 30 with 35
After doubling first element: [20, 20, 35, 40, 50]
```

**Why This Output Occurs**: Update replaces the value at index 2 from 30 to 35. The in-place modification doubles index 0 from 10 to 20. Both operations are O(1).

### Real-World Cases

- **Score Tracking**: Updating player scores during a game
- **Sensor Data**: Updating latest reading values
- **Configuration**: Modifying settings in memory
- **State Machines**: Transitioning element states

---

## 12. Resizing

### Definitions

**Core Definition**
Resizing is the operation of changing an array's capacity, typically by allocating a new larger array and copying existing elements.

**Technical Definition**
Resizing involves allocating a new contiguous block of memory, copying all elements from the old array, and deallocating the old memory.

**Beginner-Friendly Explanation**
Resizing is like moving to a bigger house—you build the new house first, move all your furniture, then leave the old house.

### Purposes (All begin with "To")

- **To** accommodate more elements than current capacity allows
- **To** reduce memory waste by shrinking oversized arrays
- **To** implement dynamic array growth strategies
- **To** maintain amortized O(1) append performance

### Syntax Rules and Structure

**General Algorithm**
```
resize(array, new_capacity):
    new_array = allocate(new_capacity * element_size)
    copy old elements to new_array (up to min(old_size, new_capacity))
    free old_array
    array = new_array
    capacity = new_capacity
```

**Growth Strategies**

| Strategy | Growth Factor | Amortized Append | Space Overhead |
|----------|--------------|------------------|----------------|
| Constant | +k | O(n) | Minimal |
| Doubling | ×2 | O(1) | Up to 100% |
| 1.5x | ×1.5 | O(1) | Up to 50% |

### Annotated Code Examples

**Example 1: Manual Resize in C**
```c
#include <stdio.h>
#include <stdlib.h>

int* resize(int* old_arr, int old_size, int new_capacity) {
    // Allocate new memory
    int* new_arr = (int*)malloc(new_capacity * sizeof(int));
    
    if (new_arr == NULL) {
        printf("Memory allocation failed\n");
        return old_arr;  // Return original on failure
    }
    
    // Copy existing elements
    for (int i = 0; i < old_size; i++) {
        new_arr[i] = old_arr[i];
    }
    
    // Free old memory
    free(old_arr);
    
    return new_arr;
}

int main() {
    int capacity = 4;
    int size = 4;
    int* arr = (int*)malloc(capacity * sizeof(int));
    
    // Fill original array
    for (int i = 0; i < size; i++) {
        arr[i] = (i + 1) * 10;
    }
    
    printf("Original: capacity=%d, size=%d\n", capacity, size);
    for (int i = 0; i < size; i++) printf("%d ", arr[i]);
    printf("\n");
    
    // Resize to double capacity
    int new_capacity = capacity * 2;
    arr = resize(arr, size, new_capacity);
    capacity = new_capacity;
    
    // Add new element
    arr[size++] = 50;
    
    printf("After resize: capacity=%d, size=%d\n", capacity, size);
    for (int i = 0; i < size; i++) printf("%d ", arr[i]);
    printf("\n");
    
    free(arr);
    return 0;
}
```

**Expected Output:**
```
Original: capacity=4, size=4
10 20 30 40 
After resize: capacity=8, size=5
10 20 30 40 50 
```

**Why This Output Occurs**: The resize function allocates 8 integers (32 bytes) instead of 4 (16 bytes), copies the existing 4 elements, and frees the old array. The new element 50 is added at index 4. This is the fundamental mechanism behind dynamic arrays.

### Real-World Cases

- **std::vector**: Automatic resizing in C++ STL
- **ArrayList**: Java's dynamic array implementation
- **Python list**: Dynamic array with overallocation
- **StringBuilder**: Resizable character buffers

### References

- University of Texas at San Antonio - Dynamic Array Lists - http://www.cs.utsa.edu/~vonronne/classes/cs2213-s09/dynamic-arrays.pdf
- IISc DS286 - List Operations and Resizing - https://cds.iisc.ac.in/wp-content/uploads/DS286.AUG2016.L4-5.Lists_.pdf

---

## References (Consolidated)

- Carnegie Mellon University - Data Structure and Two-dimensional Arrays - http://www.cs.cmu.edu/afs/andrew/course/15/110-m12/www/applications/ln/Unit06PtAPtB.pdf
- Carnegie Mellon University - UNIT 6A: Stacks and Queues - http://www.cs.cmu.edu/afs/andrew/scs/cs/15-494-sp09/dst/D/I/www/Unit06PtA-handout.pdf
- Brown University CS18 - Dynamic Arrays (ArrayLists) - https://brown-csci0200.github.io/assets/lectures/11arraylists2/11dynArrays.pdf
- CRAN Reticulate - Addressing an Issue: Arrays in R and Python - https://mirrors.cstcloud.cn/CRAN/web/packages/reticulate/vignettes/arrays.html
- Cornell University - Asymptotic Complexity - https://www.cs.cornell.edu/courses/cs211/2002sp/Lectures/Complexity.pdf
- University of Miskolc - Elementary Dynamic Sets and Array Data Structure - https://web.uni-miskolc.hu/~matha/4_Elementary_Dynamic_Sets.pdf
- IISc DS286 - Linear Lists - https://cds.iisc.ac.in/wp-content/uploads/DS286.AUG2016.L4-5.Lists_.pdf
- Cambridge University Press - Arrays and Matrices - https://www.cambridge.org/core/books/abs/short-course-in-computational-science-and-engineering/arrays-and-matrices/A7EBA0A8F6F0A65BB01C90182747B6D4
- DSC 190 - Dynamic Arrays and Amortized Analysis - https://dsc-courses.github.io/dsc190-algorithms-2022-wi/materials/lectures/02-dynamic_arrays/slides-marked.pdf
- Microsoft Learn - Declaring and Indexing Arrays - https://learn.microsoft.com/th-th/previous-versions/visualstudio/visual-studio-6.0/aa293547(v=vs.60)
- BITS Pilani - Dynamic Arrays Example: Lab3 - https://www.bits-pilani.ac.in/wp-content/uploads/DSA-L3.pdf
- University of the Philippines - Basic Data Structures - https://forums.eee.upd.edu.ph/mediawiki/images/c/ca/Coe163_2s2122_01a_review_algorithms.pdf
- Drexel University - Algorithms and Data Structures - https://www.cs.drexel.edu/~johnsojr/2004-05/winter/cs265/lectures/lec4.pdf
- University of Texas at San Antonio - Dynamic Array Lists - http://www.cs.utsa.edu/~vonronne/classes/cs2213-s09/dynamic-arrays.pdf
- Microsoft Learn - CLR Array Storage Order - https://learn.microsoft.com/vi-vn/archive/msdn-technet-forums/0c00d9af-4c10-4451-b7eb-972de0944ff8
- CMU - Order of Complexity - http://www.cs.cmu.edu/%7Etcortina/15110f11/Unit04PtC.pdf
- CMU - Arrays and Linked Lists - https://www.cs.cmu.edu/~15110-n15/lectures/unit06-1-Arrays_handout.pdf
- LeetCode - DSA Interview Cheatsheet (C++ Edition) - https://leetcode.com/discuss/post/7442385/dsa-interview-cheatsheet-c-edition-by-mo-jim1/
- UPC - Linked Lists and Arrays Comparison - https://upcommons.upc.edu/bitstream/handle/2117/94179/08_PD_Capitol07_pp_115_182.pdf