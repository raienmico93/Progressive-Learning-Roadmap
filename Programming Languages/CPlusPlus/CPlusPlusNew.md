# C++ Comprehensive, Structured, and Progressive Learning Roadmap

## From Language Foundations to Advanced Systems Programming, Performance Engineering, and Production C++ Development

C++ is best learned as more than "C with classes." The progression should cover **syntax → types → control flow → functions → pointers → references → OOP → templates → STL → memory management → RAII → move semantics → smart pointers → exceptions → concurrency → metaprogramming → performance → systems programming → build systems → testing → architecture → production engineering**.

---

# I. C++ Foundations

- **1. What C++ Is**
  - C++
  - C++ history
  - Bjarne Stroustrup
  - C with Classes
  - C++ standards
    - C++98
    - C++03
    - C++11
    - C++14
    - C++17
    - C++20
    - C++23
    - C++26
  - C++ philosophy
    - Zero-overhead principle
    - Don't pay for what you don't use
    - Multi-paradigm
  - C++ vs C
  - C++ vs Rust
  - C++ vs Go
  - C++ vs Java
  - C++ vs C#
  - C++ use cases
    - Systems programming
    - Game development
    - Embedded systems
    - High-frequency trading
    - Scientific computing
    - Graphics
    - Databases
    - Browsers
    - Operating systems
    - Machine learning
    - Audio/video processing
    - Compilers
    - Robotics

- **2. Compilation Model**
  - Source files
  - Header files
  - Translation units
  - Preprocessing
  - Compilation
  - Assembly
  - Linking
  - Static linking
  - Dynamic linking
  - Object files
  - Executables
  - Libraries
    - Static libraries
    - Shared libraries
    - Header-only libraries
  - Compilers
    - GCC
    - Clang
    - MSVC
    - Intel C++
    - MinGW
  - Linkers
  - Build tools
    - Make
    - CMake
    - Bazel
    - Meson
    - Ninja
    - Premake
  - Compiler flags
  - Optimization levels
    - `-O0`
    - `-O1`
    - `-O2`
    - `-O3`
    - `-Os`
    - `-Ofast`
  - Debug vs release builds
  - Compilation best practices

- **3. Setting Up C++**
  - Compiler installation
    - Windows
      - MSVC
      - MinGW
      - Clang
    - macOS
      - Xcode
      - Clang
    - Linux
      - GCC
      - Clang
  - IDEs
    - Visual Studio
    - CLion
    - VS Code
    - Qt Creator
    - Code::Blocks
    - Eclipse CDT
    - Xcode
    - KDevelop
    - Sublime Text
    - Vim
    - Neovim
    - Emacs
  - Build systems
    - CMake
    - Make
    - Bazel
    - Meson
    - Ninja
    - Premake
    - Xmake
  - Package managers
    - vcpkg
    - Conan
    - Hunter
    - CPM
    - Buckaroo
  - Debuggers
    - GDB
    - LLDB
    - Visual Studio Debugger
    - WinDbg
  - Profilers
    - perf
    - Valgrind
    - VTune
    - Tracy
    - Instruments
    - gprof
  - Static analysis
    - Clang-Tidy
    - Cppcheck
    - PVS-Studio
    - Coverity
    - SonarQube
  - Formatting
    - clang-format
    - AStyle
    - Uncrustify
  - Linting
    - Clang-Tidy
    - cpplint
  - Editor configuration
    - `.clang-format`
    - `.clang-tidy`
    - `compile_commands.json`
    - `.editorconfig`

- **4. Basic Syntax**
  - Program structure
  - `main` function
  - `int main()`
  - `int main(int argc, char* argv[])`
  - Return values
  - Statements
  - Expressions
  - Semicolons
  - Braces
  - Comments
    - Single-line
    - Multi-line
  - Preprocessor directives
    - `#include`
    - `#define`
    - `#ifdef`
    - `#ifndef`
    - `#if`
    - `#else`
    - `#elif`
    - `#endif`
    - `#pragma`
    - `#error`
    - `#warning`
  - Namespaces
    - `namespace`
    - `using`
    - `using namespace`
    - Nested namespaces
    - Anonymous namespaces
    - Inline namespaces
    - Namespace aliases
  - Identifiers
  - Keywords
  - Reserved words
  - Naming conventions
  - Literals
  - Escape sequences
  - Trigraphs (removed in C++17)
  - Alternative tokens

- **5. First Program**
  - Hello World
  - `#include <iostream>`
  - `std::cout`
  - `std::endl`
  - `std::cin`
  - `std::cerr`
  - `std::clog`
  - Input/output streams
  - Formatting output
  - `std::format` (C++20)
  - `std::print` (C++23)

---

# II. Variables and Data Types

- **6. Variables**
  - Variables
  - Variable declaration
  - Variable initialization
  - Variable assignment
  - Copy initialization
  - Direct initialization
  - Uniform initialization
  - List initialization
  - Brace initialization
  - `auto` keyword
  - `decltype`
  - `const`
  - `constexpr`
  - `consteval` (C++20)
  - `constinit` (C++20)
  - `volatile`
  - `mutable`
  - `static`
  - `extern`
  - `register` (removed in C++17)
  - `thread_local`
  - Variable scope
  - Variable lifetime
  - Storage duration
    - Automatic
    - Static
    - Thread
    - Dynamic
  - Linkage
    - Internal
    - External
    - No linkage
  - Variable naming
  - Variable best practices

- **7. Fundamental Types**
  - Integer types
    - `short`
    - `int`
    - `long`
    - `long long`
    - `unsigned`
    - `signed`
    - Fixed-width integers
      - `int8_t`
      - `int16_t`
      - `int32_t`
      - `int64_t`
      - `uint8_t`
      - `uint16_t`
      - `uint32_t`
      - `uint64_t`
  - Character types
    - `char`
    - `signed char`
    - `unsigned char`
    - `wchar_t`
    - `char8_t` (C++20)
    - `char16_t`
    - `char32_t`
  - Floating-point types
    - `float`
    - `double`
    - `long double`
  - Boolean type
    - `bool`
  - Void type
    - `void`
  - `nullptr_t`
  - `std::byte` (C++17)
  - Type sizes
  - Type ranges
  - Numeric limits
    - `<limits>`
    - `std::numeric_limits`
  - Type aliases
    - `typedef`
    - `using`
  - Type deduction
    - `auto`
    - `decltype`
    - `decltype(auto)`
  - Type conversions
    - Implicit conversions
    - Explicit conversions
    - `static_cast`
    - `dynamic_cast`
    - `const_cast`
    - `reinterpret_cast`
    - C-style casts
    - `std::bit_cast` (C++20)
  - Narrowing conversions
  - Type traits

- **8. Literals**
  - Integer literals
    - Decimal
    - Octal
    - Hexadecimal
    - Binary
    - Digit separators
    - Suffixes
  - Floating-point literals
    - Decimal
    - Scientific
    - Hexadecimal
    - Suffixes
  - Character literals
  - String literals
  - Boolean literals
  - Pointer literals
  - User-defined literals
  - Raw string literals
  - String literal concatenation
  - Unicode literals

- **9. Strings**
  - C-style strings
  - `char*`
  - `const char*`
  - Null-terminated strings
  - String functions
    - `strlen`
    - `strcpy`
    - `strncpy`
    - `strcat`
    - `strncat`
    - `strcmp`
    - `strncmp`
    - `strstr`
    - `strchr`
    - `strrchr`
  - `std::string`
  - `std::string_view` (C++17)
  - `std::wstring`
  - `std::u8string` (C++20)
  - `std::u16string`
  - `std::u32string`
  - String operations
  - String conversion
  - String performance
  - SSO
  - Small String Optimization
  - String best practices

- **10. Arrays**
  - C-style arrays
  - Array declaration
  - Array initialization
  - Array indexing
  - Array decay
  - Multi-dimensional arrays
  - `std::array`
  - `std::vector`
  - Array vs pointer
  - Array best practices

- **11. Enumerations**
  - Unscoped enums
  - Scoped enums
  - `enum class`
  - Enum underlying type
  - Enum values
  - Enum conversion
  - Enum best practices

- **12. Structures and Unions**
  - `struct`
  - `union`
  - Struct members
  - Struct initialization
  - Aggregate initialization
  - Designated initializers
  - Nested structs
  - Anonymous structs
  - Anonymous unions
  - Struct alignment
  - Struct padding
  - Bit fields
  - Struct best practices

- **13. Type Aliases and typedefs**
  - `typedef`
  - `using`
  - Type aliases
  - Alias templates
  - `using` vs `typedef`
  - Alias best practices

---

# III. Operators

- **14. Arithmetic Operators**
  - `+`
  - `-`
  - `*`
  - `/`
  - `%`
  - `++`
  - `--`
  - Prefix vs postfix
  - Unary operators
  - Operator precedence
  - Operator associativity
  - Integer division
  - Floating-point division

- **15. Assignment Operators**
  - `=`
  - `+=`
  - `-=`
  - `*=`
  - `/=`
  - `%=`
  - `&=`
  - `|=`
  - `^=`
  - `<<=`
  - `>>=`
  - Chained assignment
  - Compound assignment
  - Copy assignment
  - Move assignment

- **16. Comparison Operators**
  - `==`
  - `!=`
  - `<`
  - `>`
  - `<=`
  - `>=`
  - `<=>` (C++20)
  - Three-way comparison
  - Spaceship operator
  - Comparison categories
  - Default comparisons
  - Comparison best practices

- **17. Logical Operators**
  - `&&`
  - `||`
  - `!`
  - Short-circuit evaluation
  - Logical vs bitwise
  - Boolean operators

- **18. Bitwise Operators**
  - `&`
  - `|`
  - `^`
  - `~`
  - `<<`
  - `>>`
  - Bit manipulation
  - Bit masks
  - Bit shifting
  - Bitwise applications
  - `std::bitset`
  - Bit manipulation best practices

- **19. Pointer Operators**
  - `&` (address-of)
  - `*` (dereference)
  - `->` (member access)
  - `.` (member access)
  - Pointer arithmetic
  - Pointer comparison
  - Null pointers
  - `nullptr`

- **20. Member Access Operators**
  - `.`
  - `->`
  - `.*`
  - `->*`
  - `::` (scope resolution)
  - Member access best practices

- **21. Other Operators**
  - `sizeof`
  - `alignof`
  - `typeid`
  - `new`
  - `delete`
  - `new[]`
  - `delete[]`
  - `throw`
  - `co_await` (C++20)
  - `co_yield` (C++20)
  - `co_return` (C++20)
  - `,` (comma)
  - `?:` (ternary)
  - `sizeof...`
  - `noexcept`
  - `static_assert`
  - `decltype`

- **22. Operator Overloading**
  - Operator overloading
  - Overloadable operators
  - Non-overloadable operators
  - Member operator overloading
  - Non-member operator overloading
  - Friend operator overloading
  - Arithmetic operator overloading
  - Comparison operator overloading
  - Assignment operator overloading
  - Subscript operator overloading
  - Function call operator overloading
  - Increment/decrement overloading
  - Stream operator overloading
  - Conversion operator overloading
  - `operator<<`
  - `operator>>`
  - `operator[]`
  - `operator()`
  - `operator->`
  - `operator*`
  - `operator&`
  - `operator new`
  - `operator delete`
  - Operator overloading best practices
  - Operator overloading pitfalls

---

# IV. Control Flow

- **23. Conditional Statements**
  - `if`
  - `else if`
  - `else`
  - Nested conditionals
  - Ternary operator
  - `if` with initializer (C++17)
  - `if constexpr` (C++17)
  - `if consteval` (C++23)
  - Conditional best practices

- **24. Switch Statements**
  - `switch`
  - `case`
  - `break`
  - `default`
  - Fall-through
  - `[[fallthrough]]`
  - `switch` with initializer (C++17)
  - Jump tables
  - Switch best practices
  - Switch vs if-else

- **25. Loops**
  - `for`
  - Range-based `for`
  - `while`
  - `do...while`
  - Loop control
    - `break`
    - `continue`
    - `goto`
  - Infinite loops
  - Nested loops
  - Loop optimization
  - Loop best practices

- **26. Range-Based For Loop**
  - Range-based for loop
  - `for (auto& x : container)`
  - `for (const auto& x : container)`
  - `for (auto&& x : container)`
  - Range-based for with initializer (C++20)
  - Range-based for with structured bindings
  - Range-based for best practices

- **27. Jump Statements**
  - `break`
  - `continue`
  - `return`
  - `goto`
  - `goto` best practices
  - `goto` pitfalls

---

# V. Functions

- **28. Function Fundamentals**
  - Functions
  - Function declaration
  - Function definition
  - Function prototype
  - Function signature
  - Function parameters
  - Function arguments
  - Return types
  - `void` return
  - Function body
  - Function call
  - Function overloading
  - Function templates
  - Default arguments
  - Inline functions
  - `constexpr` functions
  - `consteval` functions
  - `constinit`
  - `noexcept`
  - Function best practices

- **29. Parameter Passing**
  - Pass by value
  - Pass by reference
  - Pass by pointer
  - Pass by const reference
  - Pass by rvalue reference
  - Perfect forwarding
  - `std::forward`
  - Parameter passing best practices

- **30. Return Values**
  - Return by value
  - Return by reference
  - Return by pointer
  - Return by rvalue reference
  - Return value optimization
  - RVO
  - NRVO
  - Copy elision
  - Guaranteed copy elision (C++17)
  - Return best practices

- **31. Function Overloading**
  - Function overloading
  - Overloading rules
  - Overload resolution
  - Ambiguity
  - Name mangling
  - `extern "C"`
  - Overloading best practices

- **32. Default Arguments**
  - Default arguments
  - Default argument rules
  - Default argument pitfalls
  - Default arguments vs overloads
  - Default argument best practices

- **33. Inline Functions**
  - `inline` keyword
  - Inline expansion
  - Inline functions vs macros
  - Inline variables (C++17)
  - Inline best practices

- **34. Constexpr**
  - `constexpr`
  - Compile-time evaluation
  - Constant expressions
  - `constexpr` functions
  - `constexpr` variables
  - `constexpr` constructors
  - `consteval` (C++20)
  - `constinit` (C++20)
  - Compile-time computation
  - `static_assert`
  - Constexpr best practices

- **35. Lambda Expressions**
  - Lambda expressions
  - Lambda syntax
  - Capture list
  - Capture by value
  - Capture by reference
  - Capture by move
  - Init capture
  - `mutable` lambdas
  - Generic lambdas
  - Templated lambdas
  - Lambda return type
  - Lambda in STL algorithms
  - Lambda best practices

- **36. Function Pointers**
  - Function pointers
  - Function pointer syntax
  - Function pointer declaration
  - Function pointer invocation
  - Function pointer as parameter
  - Function pointer as return value
  - `std::function`
  - `std::bind`
  - Function pointer best practices

- **37. Variadic Functions**
  - Variadic functions
  - `...`
  - `<cstdarg>`
  - `va_list`
  - `va_start`
  - `va_arg`
  - `va_end`
  - Variadic templates
  - Parameter packs
  - Fold expressions
  - Variadic best practices

- **38. Recursion**
  - Recursion
  - Base case
  - Recursive case
  - Tail recursion
  - Tail call optimization
  - Recursion depth
  - Stack overflow
  - Recursion vs iteration
  - Recursion best practices

---

# VI. Pointers and References

- **39. Pointers**
  - Pointers
  - Pointer declaration
  - Pointer initialization
  - Pointer dereference
  - Pointer arithmetic
  - Pointer comparison
  - Null pointers
  - `nullptr`
  - `NULL`
  - `0`
  - Dangling pointers
  - Wild pointers
  - Void pointers
  - `void*`
  - Pointer to pointer
  - Pointer to function
  - Pointer to member
  - Pointer best practices
  - Pointer pitfalls

- **40. References**
  - References
  - Lvalue references
  - `T&`
  - Rvalue references
  - `T&&`
  - Reference initialization
  - Reference binding
  - Reference collapsing
  - Const references
  - Reference vs pointer
  - Reference best practices

- **41. Const Correctness**
  - `const`
  - Const variables
  - Const pointers
  - Const references
  - Const member functions
  - Const parameters
  - Const return values
  - Mutable
  - Const correctness best practices

- **42. Smart Pointers**
  - Smart pointers
  - `std::unique_ptr`
  - `std::shared_ptr`
  - `std::weak_ptr`
  - `std::auto_ptr` (deprecated)
  - Ownership semantics
  - Reference counting
  - Custom deleters
  - `std::make_unique`
  - `std::make_shared`
  - `std::enable_shared_from_this`
  - Smart pointer best practices
  - Smart pointer pitfalls
  - Circular references

- **43. Memory Management**
  - Stack memory
  - Heap memory
  - Static memory
  - Thread-local memory
  - `new`
  - `delete`
  - `new[]`
  - `delete[]`
  - Placement new
  - `operator new`
  - `operator delete`
  - Memory leaks
  - Memory fragmentation
  - Memory alignment
  - Memory pools
  - Arena allocators
  - Custom allocators
  - `std::allocator`
  - Allocator-aware containers
  - PMR
  - Polymorphic Memory Resources
  - `std::pmr`
  - Memory management best practices

- **44. RAII**
  - RAII
  - Resource Acquisition Is Initialization
  - Resource management
  - Destructors
  - Scope-based cleanup
  - Exception safety
  - RAII best practices
  - RAII examples

- **45. Move Semantics**
  - Move semantics
  - Rvalue references
  - `std::move`
  - Move constructors
  - Move assignment operators
  - Rule of Three
  - Rule of Five
  - Rule of Zero
  - Copy elision
  - Perfect forwarding
  - `std::forward`
  - Move semantics best practices

- **46. Value Categories**
  - Lvalues
  - Rvalues
  - Xvalues
  - PRvalues
  - Glvalues
  - Value categories
  - `std::move`
  - `std::forward`
  - Value category best practices

---

# VII. Object-Oriented Programming

- **47. Classes and Objects**
  - Classes
  - Objects
  - Instances
  - Class declaration
  - Class definition
  - Class members
  - Data members
  - Member functions
  - Access specifiers
    - `public`
    - `private`
    - `protected`
  - `this` pointer
  - Constructor
  - Destructor
  - Copy constructor
  - Move constructor
  - Copy assignment
  - Move assignment
  - Default constructor
  - Deleted functions
  - Defaulted functions
  - `= default`
  - `= delete`
  - Class best practices

- **48. Constructors and Destructors**
  - Constructors
  - Default constructors
  - Parameterized constructors
  - Copy constructors
  - Move constructors
  - Delegating constructors
  - Inheriting constructors
  - Member initializer list
  - Constructor initialization order
  - Explicit constructors
  - `explicit` keyword
  - Destructors
  - Virtual destructors
  - Pure virtual destructors
  - Constructor/destructor best practices

- **49. Inheritance**
  - Inheritance
  - Base classes
  - Derived classes
  - `public` inheritance
  - `protected` inheritance
  - `private` inheritance
  - Multiple inheritance
  - Virtual inheritance
  - Diamond problem
  - `override`
  - `final`
  - Abstract classes
  - Pure virtual functions
  - Interface classes
  - Inheritance best practices
  - Composition over inheritance

- **50. Polymorphism**
  - Polymorphism
  - Compile-time polymorphism
  - Runtime polymorphism
  - Virtual functions
  - Virtual tables
  - Vtable
  - Virtual function pointers
  - Dynamic dispatch
  - Virtual destructors
  - Pure virtual functions
  - Abstract classes
  - Override
  - `final`
  - `override`
  - Polymorphism best practices

- **51. Virtual Functions**
  - Virtual functions
  - `virtual` keyword
  - Virtual function table
  - Vtable
  - Vptr
  - Dynamic dispatch
  - Virtual destructors
  - Pure virtual functions
  - `= 0`
  - Overriding
  - `override` specifier
  - `final` specifier
  - Covariant return types
  - Virtual function best practices

- **52. Abstract Classes and Interfaces**
  - Abstract classes
  - Pure virtual functions
  - Interface classes
  - Multiple inheritance
  - Virtual inheritance
  - Abstract class best practices
  - Interface best practices

- **53. Operator Overloading in Classes**
  - Member operator overloading
  - Non-member operator overloading
  - Friend operator overloading
  - Comparison operators
  - Arithmetic operators
  - Assignment operators
  - Stream operators
  - Subscript operators
  - Function call operators
  - Conversion operators
  - Increment/decrement operators
  - Operator overloading best practices

- **54. Special Member Functions**
  - Default constructor
  - Destructor
  - Copy constructor
  - Copy assignment operator
  - Move constructor
  - Move assignment operator
  - Rule of Three
  - Rule of Five
  - Rule of Zero
  - Special member function best practices

- **55. Friend Functions and Classes**
  - Friend functions
  - Friend classes
  - `friend` keyword
  - Friend function declaration
  - Friend class declaration
  - Friend best practices
  - Friend pitfalls

- **56. Static Members**
  - Static data members
  - Static member functions
  - Static member initialization
  - Static member access
  - Inline static members (C++17)
  - Static member best practices

- **57. Nested Classes**
  - Nested classes
  - Inner classes
  - Nested class access
  - Nested class scope
  - Nested class best practices

- **58. Object Layout**
  - Object layout
  - Memory layout
  - Data member ordering
  - Padding
  - Alignment
  - Empty base optimization
  - EBO
  - Object size
  - Object copy
  - Object move
  - Object best practices

---

# VIII. Templates

- **59. Template Fundamentals**
  - Templates
  - Function templates
  - Class templates
  - Template parameters
  - Template arguments
  - Template instantiation
  - Implicit instantiation
  - Explicit instantiation
  - Template specialization
  - Full specialization
  - Partial specialization
  - Template best practices

- **60. Function Templates**
  - Function templates
  - Template parameter deduction
  - Template argument deduction
  - Overload resolution
  - Template specialization
  - Template function overloading
  - Function template best practices

- **61. Class Templates**
  - Class templates
  - Template member functions
  - Template member variables
  - Template specialization
  - Partial specialization
  - Nested templates
  - Class template best practices

- **62. Template Specialization**
  - Full specialization
  - Partial specialization
  - Template specialization rules
  - Specialization vs overloading
  - Specialization best practices

- **63. Variadic Templates**
  - Variadic templates
  - Parameter packs
  - Pack expansion
  - Fold expressions
  - Recursive variadic templates
  - Variadic template best practices

- **64. Template Metaprogramming**
  - Template metaprogramming
  - TMP
  - Compile-time computation
  - Type traits
  - `std::enable_if`
  - `std::conditional`
  - `std::integral_constant`
  - `std::bool_constant`
  - `std::is_same`
  - `std::is_base_of`
  - `std::is_convertible`
  - SFINAE
  - Substitution Failure Is Not An Error
  - `decltype`
  - `std::declval`
  - Tag dispatch
  - Constexpr if
  - Concepts (C++20)
  - Template metaprogramming best practices

- **65. SFINAE**
  - SFINAE
  - Substitution Failure Is Not An Error
  - `std::enable_if`
  - `std::void_t`
  - Detection idiom
  - SFINAE best practices
  - SFINAE alternatives

- **66. Concepts (C++20)**
  - Concepts
  - `concept` keyword
  - `requires` clause
  - `requires` expression
  - Concept definition
  - Concept composition
  - Standard concepts
    - `std::same_as`
    - `std::derived_from`
    - `std::convertible_to`
    - `std::integral`
    - `std::floating_point`
    - `std::invocable`
    - `std::predicate`
    - `std::ranges::range`
  - Concept best practices
  - Concepts vs SFINAE

- **67. Template Aliases**
  - Template aliases
  - `using` aliases
  - Alias templates
  - Template alias best practices

- **68. CRTP**
  - CRTP
  - Curiously Recurring Template Pattern
  - Static polymorphism
  - Mixins
  - CRTP best practices
  - CRTP examples

- **69. Policy-Based Design**
  - Policy-based design
  - Policy classes
  - Policy composition
  - Policy-based design best practices

---

# IX. Standard Template Library (STL)

- **70. STL Fundamentals**
  - STL
  - Standard Template Library
  - Containers
  - Iterators
  - Algorithms
  - Function objects
  - Adaptors
  - Allocators
  - STL components
  - STL best practices

- **71. Containers**
  - Sequence containers
    - `std::vector`
    - `std::array`
    - `std::deque`
    - `std::list`
    - `std::forward_list`
  - Associative containers
    - `std::set`
    - `std::multiset`
    - `std::map`
    - `std::multimap`
  - Unordered associative containers
    - `std::unordered_set`
    - `std::unordered_multiset`
    - `std::unordered_map`
    - `std::unordered_multimap`
  - Container adaptors
    - `std::stack`
    - `std::queue`
    - `std::priority_queue`
  - Container operations
  - Container performance
  - Container selection
  - Container best practices

- **72. Vectors**
  - `std::vector`
  - Vector declaration
  - Vector initialization
  - Vector operations
    - `push_back()`
    - `pop_back()`
    - `emplace_back()`
    - `insert()`
    - `erase()`
    - `clear()`
    - `resize()`
    - `reserve()`
    - `shrink_to_fit()`
    - `at()`
    - `operator[]`
    - `front()`
    - `back()`
    - `data()`
    - `size()`
    - `capacity()`
    - `empty()`
  - Vector iteration
  - Vector performance
  - Vector best practices

- **73. Arrays**
  - `std::array`
  - Array declaration
  - Array initialization
  - Array operations
  - Array vs C-style array
  - Array best practices

- **74. Lists**
  - `std::list`
  - Doubly linked list
  - List operations
  - List performance
  - List best practices
  - `std::forward_list`
  - Singly linked list
  - Forward list best practices

- **75. Deques**
  - `std::deque`
  - Double-ended queue
  - Deque operations
  - Deque performance
  - Deque best practices

- **76. Sets and Maps**
  - `std::set`
  - `std::multiset`
  - `std::map`
  - `std::multimap`
  - Ordered containers
  - Red-black tree
  - Set operations
  - Map operations
  - Set best practices
  - Map best practices

- **77. Unordered Containers**
  - `std::unordered_set`
  - `std::unordered_multiset`
  - `std::unordered_map`
  - `std::unordered_multimap`
  - Hash functions
  - Bucket interface
  - Load factor
  - Rehashing
  - Unordered container best practices

- **78. Container Adaptors**
  - `std::stack`
  - `std::queue`
  - `std::priority_queue`
  - Adaptor operations
  - Adaptor best practices

- **79. Iterators**
  - Iterators
  - Iterator categories
    - Input iterators
    - Output iterators
    - Forward iterators
    - Bidirectional iterators
    - Random access iterators
    - Contiguous iterators (C++20)
  - Iterator operations
  - Iterator adaptors
    - `std::reverse_iterator`
    - `std::back_insert_iterator`
    - `std::front_insert_iterator`
    - `std::insert_iterator`
    - `std::move_iterator`
  - Iterator traits
  - Iterator best practices

- **80. Algorithms**
  - `<algorithm>`
  - Non-modifying algorithms
    - `std::all_of`
    - `std::any_of`
    - `std::none_of`
    - `std::for_each`
    - `std::count`
    - `std::count_if`
    - `std::find`
    - `std::find_if`
    - `std::find_if_not`
    - `std::find_end`
    - `std::find_first_of`
    - `std::adjacent_find`
    - `std::search`
    - `std::search_n`
    - `std::mismatch`
    - `std::equal`
    - `std::is_permutation`
  - Modifying algorithms
    - `std::copy`
    - `std::copy_if`
    - `std::copy_n`
    - `std::copy_backward`
    - `std::move`
    - `std::move_backward`
    - `std::fill`
    - `std::fill_n`
    - `std::transform`
    - `std::generate`
    - `std::generate_n`
    - `std::remove`
    - `std::remove_if`
    - `std::remove_copy`
    - `std::remove_copy_if`
    - `std::replace`
    - `std::replace_if`
    - `std::replace_copy`
    - `std::replace_copy_if`
    - `std::swap`
    - `std::swap_ranges`
    - `std::iter_swap`
    - `std::reverse`
    - `std::reverse_copy`
    - `std::rotate`
    - `std::rotate_copy`
    - `std::shuffle`
    - `std::sample`
    - `std::unique`
    - `std::unique_copy`
  - Sorting algorithms
    - `std::sort`
    - `std::stable_sort`
    - `std::partial_sort`
    - `std::partial_sort_copy`
    - `std::is_sorted`
    - `std::is_sorted_until`
    - `std::nth_element`
  - Binary search algorithms
    - `std::lower_bound`
    - `std::upper_bound`
    - `std::binary_search`
    - `std::equal_range`
  - Set operations
    - `std::merge`
    - `std::inplace_merge`
    - `std::includes`
    - `std::set_difference`
    - `std::set_intersection`
    - `std::set_symmetric_difference`
    - `std::set_union`
  - Heap operations
    - `std::make_heap`
    - `std::push_heap`
    - `std::pop_heap`
    - `std::sort_heap`
    - `std::is_heap`
    - `std::is_heap_until`
  - Min/max operations
    - `std::min`
    - `std::max`
    - `std::minmax`
    - `std::min_element`
    - `std::max_element`
    - `std::minmax_element`
    - `std::clamp`
  - Numeric algorithms
    - `std::accumulate`
    - `std::reduce`
    - `std::inner_product`
    - `std::adjacent_difference`
    - `std::partial_sum`
    - `std::exclusive_scan`
    - `std::inclusive_scan`
    - `std::transform_reduce`
    - `std::transform_exclusive_scan`
    - `std::transform_inclusive_scan`
  - Algorithm best practices

- **81. Ranges (C++20)**
  - Ranges
  - `<ranges>`
  - Range concepts
  - Range views
  - Range adaptors
  - Range algorithms
  - Range factories
  - Pipes
  - Lazy evaluation
  - Range best practices

- **82. Function Objects**
  - Function objects
  - Functors
  - `operator()`
  - Predefined function objects
    - `std::plus`
    - `std::minus`
    - `std::multiplies`
    - `std::divides`
    - `std::modulus`
    - `std::negate`
    - `std::equal_to`
    - `std::not_equal_to`
    - `std::greater`
    - `std::less`
    - `std::greater_equal`
    - `std::less_equal`
    - `std::logical_and`
    - `std::logical_or`
    - `std::logical_not`
    - `std::bit_and`
    - `std::bit_or`
    - `std::bit_xor`
    - `std::bit_not`
  - Function object best practices

- **83. Smart Pointers**
  - `std::unique_ptr`
  - `std::shared_ptr`
  - `std::weak_ptr`
  - Smart pointer operations
  - Custom deleters
  - Smart pointer best practices

- **84. Utility Components**
  - `std::pair`
  - `std::tuple`
  - `std::optional`
  - `std::variant`
  - `std::any`
  - `std::string_view`
  - `std::span`
  - `std::function`
  - `std::reference_wrapper`
  - `std::ref`
  - `std::cref`
  - Utility best practices

- **85. Time and Date**
  - `<chrono>`
  - Duration
  - Time point
  - Clock
  - `std::chrono::system_clock`
  - `std::chrono::steady_clock`
  - `std::chrono::high_resolution_clock`
  - Time zones (C++20)
  - Calendars (C++20)
  - Date and time best practices

---

# X. Exception Handling

- **86. Exception Fundamentals**
  - Exceptions
  - `throw`
  - `try`
  - `catch`
  - Exception hierarchy
  - `std::exception`
  - Standard exceptions
  - Custom exceptions
  - Exception propagation
  - Stack unwinding
  - Exception best practices

- **87. Try-Catch**
  - `try` block
  - `catch` block
  - Multiple catch blocks
  - Catch-all
  - `catch (...)`
  - Nested try-catch
  - Re-throwing exceptions
  - Exception chaining
  - Exception messages
  - Exception best practices

- **88. Exception Safety**
  - Exception safety
  - Basic guarantee
  - Strong guarantee
  - No-throw guarantee
  - Exception safety levels
  - RAII and exception safety
  - Exception safety best practices

- **89. Noexcept**
  - `noexcept`
  - `noexcept` specifier
  - `noexcept` operator
  - Conditional noexcept
  - `noexcept` best practices
  - `noexcept` pitfalls

- **90. Custom Exceptions**
  - Custom exception classes
  - Exception inheritance
  - `std::exception`
  - `what()`
  - Exception constructors
  - Exception best practices

- **91. Error Handling Patterns**
  - Error codes
  - `std::error_code`
  - `std::error_condition`
  - `std::expected` (C++23)
  - Exceptions vs error codes
  - Error handling best practices

---

# XI. Modern C++ Features

- **92. C++11 Features**
  - `auto`
  - Range-based for
  - Lambda expressions
  - Rvalue references
  - Move semantics
  - `nullptr`
  - `enum class`
  - `constexpr`
  - Variadic templates
  - `std::unique_ptr`
  - `std::shared_ptr`
  - `std::thread`
  - `std::mutex`
  - `std::chrono`
  - Uniform initialization
  - Initializer lists
  - `std::array`
  - `std::tuple`
  - `std::function`
  - `std::bind`
  - `std::ref`
  - `std::cref`
  - `decltype`
  - `static_assert`
  - `alignas`
  - `alignof`
  - Attributes

- **93. C++14 Features**
  - Generic lambdas
  - Return type deduction
  - Variable templates
  - `constexpr` improvements
  - Binary literals
  - Digit separators
  - `std::make_unique`
  - `std::shared_timed_mutex`
  - `std::integer_sequence`
  - `std::exchange`
  - `std::quoted`

- **94. C++17 Features**
  - Structured bindings
  - `if constexpr`
  - `if` with initializer
  - `switch` with initializer
  - Fold expressions
  - Class template argument deduction
  - CTAD
  - `std::optional`
  - `std::variant`
  - `std::any`
  - `std::string_view`
  - `std::filesystem`
  - `std::byte`
  - Inline variables
  - Nested namespaces
  - `[[nodiscard]]`
  - `[[maybe_unused]]`
  - `[[fallthrough]]`
  - Guaranteed copy elision
  - Parallel algorithms
  - `std::scoped_lock`
  - `std::shared_mutex`
  - `std::apply`
  - `std::invoke`
  - `std::not_fn`
  - `std::as_const`
  - `std::clamp`
  - `std::gcd`
  - `std::lcm`
  - `std::hypot`
  - `std::size`
  - `std::data`
  - `std::empty`
  - `std::bool_constant`

- **95. C++20 Features**
  - Concepts
  - Ranges
  - Coroutines
  - Modules
  - Three-way comparison
  - `<=>`
  - Designated initializers
  - `constexpr` improvements
  - `consteval`
  - `constinit`
  - `std::span`
  - `std::format`
  - `std::chrono` improvements
  - Calendar and time zones
  - `std::bit_cast`
  - `std::ssize`
  - `std::midpoint`
  - `std::lerp`
  - `std::to_array`
  - `std::erase`
  - `std::erase_if`
  - `std::atomic_ref`
  - `std::jthread`
  - `std::stop_token`
  - `std::latch`
  - `std::barrier`
  - `std::semaphore`
  - `std::source_location`
  - `std::is_constant_evaluated`
  - `std::assume_aligned`
  - `std::remove_cvref`
  - `std::type_identity`
  - `std::bind_front`
  - `std::cmp_*`
  - `[[likely]]`
  - `[[unlikely]]`
  - `[[no_unique_address]]`
  - `[[nodiscard]]` with message

- **96. C++23 Features**
  - `std::expected`
  - `std::optional` monadic operations
  - `std::mdspan`
  - `std::flat_map`
  - `std::flat_set`
  - `std::stacktrace`
  - `std::print`
  - `std::println`
  - `std::format` improvements
  - `if consteval`
  - Multidimensional subscript operator
  - `import std`
  - `static operator()`
  - `static operator[]`
  - `[[assume]]`
  - `std::to_underlying`
  - `std::unreachable`
  - `std::byteswap`
  - `std::ranges` improvements
  - `std::generator`
  - `std::basic_string::contains`
  - `std::basic_string_view::contains`
  - `std::string::resize_and_overwrite`
  - Deducing `this`
  - `constexpr` improvements

- **97. C++26 Features**
  - Contracts
  - Reflection
  - Pattern matching
  - `std::execution`
  - Senders/receivers
  - `std::simd`
  - `std::linalg`
  - `std::hive`
  - `std::inplace_vector`
  - `std::rcu`
  - `std::function_ref`
  - `std::is_within_lifetime`
  - `std::nodiscard` with reason
  - `std::optional` improvements
  - `std::variant` improvements
  - `std::expected` improvements
  - `std::flat_map` improvements

---

# XII. Memory Management and RAII

- **98. Memory Model**
  - Memory model
  - Object model
  - Storage duration
  - Object lifetime
  - Object representation
  - Alignment
  - Strict aliasing
  - Effective type
  - Memory ordering
  - Memory barriers
  - Memory model best practices

- **99. Dynamic Memory**
  - `new`
  - `delete`
  - `new[]`
  - `delete[]`
  - Placement new
  - `operator new`
  - `operator delete`
  - `malloc`
  - `free`
  - `calloc`
  - `realloc`
  - Memory leaks
  - Memory fragmentation
  - Dynamic memory best practices

- **100. Smart Pointers**
  - `std::unique_ptr`
  - `std::shared_ptr`
  - `std::weak_ptr`
  - Ownership semantics
  - Reference counting
  - Custom deleters
  - `std::make_unique`
  - `std::make_shared`
  - `std::enable_shared_from_this`
  - `std::owner_less`
  - Smart pointer best practices
  - Smart pointer pitfalls
  - Circular references

- **101. Allocators**
  - Allocators
  - `std::allocator`
  - Allocator traits
  - Custom allocators
  - Allocator-aware containers
  - `std::allocator_traits`
  - Polymorphic allocators
  - `std::pmr`
  - `std::pmr::polymorphic_allocator`
  - `std::pmr::memory_resource`
  - `std::pmr::monotonic_buffer_resource`
  - `std::pmr::unsynchronized_pool_resource`
  - `std::pmr::synchronized_pool_resource`
  - Allocator best practices

- **102. RAII**
  - RAII
  - Resource Acquisition Is Initialization
  - Resource management
  - Destructors
  - Scope-based cleanup
  - Exception safety
  - RAII best practices
  - RAII examples
  - RAII wrappers

- **103. Move Semantics**
  - Move semantics
  - Rvalue references
  - `std::move`
  - Move constructors
  - Move assignment operators
  - Rule of Five
  - Rule of Zero
  - Copy elision
  - Perfect forwarding
  - `std::forward`
  - Move semantics best practices

- **104. Garbage Collection**
  - Garbage collection
  - C++ GC
  - `std::gc`
  - GC best practices
  - GC alternatives
  - Reference counting
  - Smart pointers

- **105. Memory Profiling**
  - Memory profiling
  - Valgrind
  - Massif
  - Heaptrack
  - AddressSanitizer
  - LeakSanitizer
  - MemorySanitizer
  - Memory profiling best practices

---

# XIII. Concurrency and Multithreading

- **106. Concurrency Fundamentals**
  - Concurrency
  - Parallelism
  - Threads
  - Processes
  - Thread lifecycle
  - Thread scheduling
  - Context switching
  - Concurrency best practices

- **107. Threads**
  - `std::thread`
  - Thread creation
  - Thread join
  - Thread detach
  - `std::jthread` (C++20)
  - Thread ID
  - `std::this_thread`
  - `std::this_thread::sleep_for`
  - `std::this_thread::sleep_until`
  - `std::this_thread::yield`
  - Thread best practices

- **108. Mutexes**
  - `std::mutex`
  - `std::recursive_mutex`
  - `std::timed_mutex`
  - `std::recursive_timed_mutex`
  - `std::shared_mutex` (C++17)
  - `std::shared_timed_mutex` (C++14)
  - Lock guards
    - `std::lock_guard`
    - `std::unique_lock`
    - `std::shared_lock`
    - `std::scoped_lock` (C++17)
  - `std::lock`
  - `std::try_lock`
  - Mutex best practices

- **109. Condition Variables**
  - `std::condition_variable`
  - `std::condition_variable_any`
  - Condition variable operations
    - `wait`
    - `wait_for`
    - `wait_until`
    - `notify_one`
    - `notify_all`
  - Predicate wait
  - Condition variable best practices

- **110. Atomics**
  - `std::atomic`
  - Atomic types
  - Atomic operations
  - Memory ordering
    - `memory_order_relaxed`
    - `memory_order_consume`
    - `memory_order_acquire`
    - `memory_order_release`
    - `memory_order_acq_rel`
    - `memory_order_seq_cst`
  - `std::atomic_flag`
  - `std::atomic_ref` (C++20)
  - Atomic best practices

- **111. Futures and Promises**
  - `std::future`
  - `std::promise`
  - `std::packaged_task`
  - `std::async`
  - Future operations
    - `get`
    - `wait`
    - `wait_for`
    - `wait_until`
    - `valid`
  - `std::shared_future`
  - Future best practices

- **112. Thread Synchronization**
  - Synchronization primitives
  - Barriers
    - `std::barrier` (C++20)
    - `std::latch` (C++20)
  - Semaphores
    - `std::counting_semaphore` (C++20)
    - `std::binary_semaphore` (C++20)
  - Synchronization best practices

- **113. Memory Model**
  - C++ memory model
  - Memory ordering
  - Happens-before
  - Synchronizes-with
  - Data races
  - Atomic operations
  - Memory fences
  - `std::atomic_thread_fence`
  - `std::atomic_signal_fence`
  - Memory model best practices

- **114. Lock-Free Programming**
  - Lock-free programming
  - Wait-free programming
  - Lock-free data structures
  - Compare-and-swap
  - CAS
  - ABA problem
  - Memory reclamation
  - Hazard pointers
  - RCU
  - Lock-free best practices

- **115. Parallel Algorithms**
  - Parallel algorithms
  - Execution policies
    - `std::execution::seq`
    - `std::execution::par`
    - `std::execution::par_unseq`
    - `std::execution::unseq` (C++20)
  - Parallel STL
  - Parallel algorithms best practices

- **116. Coroutines (C++20)**
  - Coroutines
  - `co_await`
  - `co_yield`
  - `co_return`
  - Coroutine framework
  - Promise type
  - Awaitable
  - Coroutine handles
  - Generators
  - Async tasks
  - Coroutine best practices

---

# XIV. Input/Output

- **117. I/O Fundamentals**
  - I/O streams
  - `iostream`
  - Input streams
  - Output streams
  - Stream states
  - Stream formatting
  - Stream manipulators
  - I/O best practices

- **118. Standard I/O**
  - `std::cin`
  - `std::cout`
  - `std::cerr`
  - `std::clog`
  - `std::wcin`
  - `std::wcout`
  - `std::wcerr`
  - `std::wclog`
  - Stream operators
  - Stream state
  - Stream formatting
  - I/O manipulators
    - `std::endl`
    - `std::flush`
    - `std::ends`
    - `std::setw`
    - `std::setprecision`
    - `std::setfill`
    - `std::setbase`
    - `std::fixed`
    - `std::scientific`
    - `std::hex`
    - `std::dec`
    - `std::oct`
    - `std::boolalpha`
    - `std::noboolalpha`
    - `std::showbase`
    - `std::noshowbase`
    - `std::showpoint`
    - `std::noshowpoint`
    - `std::showpos`
    - `std::noshowpos`
    - `std::uppercase`
    - `std::nouppercase`
    - `std::left`
    - `std::right`
    - `std::internal`
  - I/O best practices

- **119. File I/O**
  - `<fstream>`
  - `std::ifstream`
  - `std::ofstream`
  - `std::fstream`
  - File modes
  - File operations
  - File reading
  - File writing
  - File positioning
  - File state
  - File I/O best practices

- **120. String Streams**
  - `<sstream>`
  - `std::istringstream`
  - `std::ostringstream`
  - `std::stringstream`
  - String stream operations
  - String stream best practices

- **121. Filesystem (C++17)**
  - `<filesystem>`
  - `std::filesystem::path`
  - Path operations
  - File operations
  - Directory operations
  - File system operations
  - `std::filesystem::directory_iterator`
  - `std::filesystem::recursive_directory_iterator`
  - Filesystem best practices

- **122. Formatting (C++20)**
  - `<format>`
  - `std::format`
  - `std::format_to`
  - `std::format_to_n`
  - `std::formatted_size`
  - `std::formatter`
  - Format specifications
  - `std::print` (C++23)
  - `std::println` (C++23)
  - Formatting best practices

- **123. Regular Expressions**
  - `<regex>`
  - `std::regex`
  - `std::smatch`
  - `std::cmatch`
  - `std::regex_match`
  - `std::regex_search`
  - `std::regex_replace`
  - Regex iterators
  - Regex best practices
  - Regex performance

- **124. Serialization**
  - Serialization
  - JSON libraries
    - nlohmann/json
    - RapidJSON
    - simdjson
  - Protocol Buffers
  - FlatBuffers
  - MessagePack
  - CBOR
  - XML libraries
    - pugixml
    - TinyXML
    - libxml2
  - YAML libraries
    - yaml-cpp
  - Binary serialization
  - Serialization best practices

---

# XV. Build Systems and Tooling

- **125. Build Systems**
  - Build systems
  - Make
  - CMake
  - Bazel
  - Meson
  - Ninja
  - Premake
  - Xmake
  - Build system comparison
  - Build system selection

- **126. CMake**
  - CMake
  - `CMakeLists.txt`
  - CMake commands
  - Targets
  - Libraries
  - Executables
  - Dependencies
  - `find_package`
  - `target_link_libraries`
  - `target_include_directories`
  - `target_compile_options`
  - `target_compile_features`
  - Generator expressions
  - CMake presets
  - CMake best practices

- **127. Package Managers**
  - vcpkg
  - Conan
  - Hunter
  - CPM
  - Buckaroo
  - Package manager comparison
  - Package manager best practices

- **128. Compiler Flags**
  - Warning flags
    - `-Wall`
    - `-Wextra`
    - `-Wpedantic`
    - `-Werror`
    - `-Wconversion`
    - `-Wsign-conversion`
    - `-Wshadow`
    - `-Wnon-virtual-dtor`
    - `-Wold-style-cast`
    - `-Wcast-align`
    - `-Wunused`
    - `-Woverloaded-virtual`
    - `-Wnull-dereference`
    - `-Wdouble-promotion`
    - `-Wformat=2`
  - Optimization flags
    - `-O0`
    - `-O1`
    - `-O2`
    - `-O3`
    - `-Os`
    - `-Ofast`
    - `-march=native`
    - `-mtune=native`
  - Debug flags
    - `-g`
    - `-ggdb`
    - `-fno-omit-frame-pointer`
  - Sanitizer flags
    - `-fsanitize=address`
    - `-fsanitize=thread`
    - `-fsanitize=undefined`
    - `-fsanitize=memory`
    - `-fsanitize=leak`
  - LTO
    - `-flto`
  - PGO
    - `-fprofile-generate`
    - `-fprofile-use`
  - Compiler flag best practices

- **129. Static Analysis**
  - Clang-Tidy
  - Cppcheck
  - PVS-Studio
  - Coverity
  - SonarQube
  - Include What You Use
  - IWYU
  - Static analysis best practices

- **130. Formatting**
  - clang-format
  - AStyle
  - Uncrustify
  - Formatting configuration
  - Formatting best practices

- **131. Documentation**
  - Doxygen
  - Sphinx
  - MkDocs
  - Breathe
  - Documentation best practices

- **132. Debugging**
  - GDB
  - LLDB
  - Visual Studio Debugger
  - WinDbg
  - Debugging techniques
  - Breakpoints
  - Watchpoints
  - Conditional breakpoints
  - Reverse debugging
  - Core dumps
  - Debugging best practices

- **133. Profiling**
  - perf
  - Valgrind
    - Callgrind
    - Massif
    - Cachegrind
    - Helgrind
    - DRD
  - VTune
  - Tracy
  - Instruments
  - gprof
  - Profiling best practices

- **134. Sanitizers**
  - AddressSanitizer
  - ASan
  - ThreadSanitizer
  - TSan
  - UndefinedBehaviorSanitizer
  - UBSan
  - MemorySanitizer
  - MSan
  - LeakSanitizer
  - LSan
  - Sanitizer best practices

- **135. Testing Frameworks**
  - Google Test
  - Catch2
  - doctest
  - Boost.Test
  - CppUnit
  - UnitTest++
  - Testing framework comparison

---

# XVI. Testing

- **136. Testing Fundamentals**
  - Testing
  - Test types
    - Unit tests
    - Integration tests
    - End-to-end tests
    - Functional tests
    - Performance tests
  - Test pyramid
  - Test-driven development
  - Behavior-driven development
  - Test coverage
  - Test isolation
  - Test doubles
    - Mocks
    - Stubs
    - Spies
    - Fakes
  - Testing best practices

- **137. Unit Testing**
  - Unit testing
  - Google Test
  - Catch2
  - doctest
  - Test cases
  - Test fixtures
  - Assertions
  - Matchers
  - Parameterized tests
  - Test organization
  - Test naming
  - Unit testing best practices

- **138. Google Test**
  - Google Test
  - `TEST`
  - `TEST_F`
  - `TEST_P`
  - Assertions
    - `EXPECT_EQ`
    - `EXPECT_NE`
    - `EXPECT_LT`
    - `EXPECT_LE`
    - `EXPECT_GT`
    - `EXPECT_GE`
    - `EXPECT_TRUE`
    - `EXPECT_FALSE`
    - `EXPECT_THROW`
    - `EXPECT_NO_THROW`
    - `EXPECT_ANY_THROW`
    - `ASSERT_*`
  - Test fixtures
  - Parameterized tests
  - Mocking
  - Google Mock
  - Google Test best practices

- **139. Catch2**
  - Catch2
  - `TEST_CASE`
  - `SECTION`
  - `REQUIRE`
  - `CHECK`
  - `REQUIRE_FALSE`
  - `CHECK_FALSE`
  - `REQUIRE_THROWS`
  - `REQUIRE_NOTHROW`
  - BDD-style tests
  - Catch2 best practices

- **140. Mocking**
  - Mocking
  - Google Mock
  - FakeIt
  - Trompeloeil
  - Mock objects
  - Mock methods
  - Mock expectations
  - Mocking best practices

- **141. Integration Testing**
  - Integration testing
  - Database testing
  - API testing
  - External service testing
  - Integration testing best practices

- **142. Test Automation**
  - CI integration
  - Test pipelines
  - Parallel testing
  - Test reporting
  - Code coverage
  - Mutation testing
  - Property-based testing
  - Fuzz testing
  - Testing best practices

---

# XVII. Performance Optimization

- **143. Performance Fundamentals**
  - Performance
  - Latency
  - Throughput
  - Resource utilization
  - Performance metrics
  - Performance budgets
  - Performance best practices

- **144. Profiling**
  - Profiling
  - CPU profiling
  - Memory profiling
  - Cache profiling
  - I/O profiling
  - Concurrency profiling
  - Profiling tools
  - Profiling best practices

- **145. Compiler Optimizations**
  - Compiler optimizations
  - `-O` levels
  - LTO
  - PGO
  - Inlining
  - Loop optimization
  - Vectorization
  - Auto-vectorization
  - Dead code elimination
  - Constant folding
  - Compiler optimization best practices

- **146. Cache Optimization**
  - Cache hierarchy
  - Cache lines
  - Cache misses
  - Cache locality
  - Spatial locality
  - Temporal locality
  - Cache-friendly data structures
  - Data-oriented design
  - Cache optimization best practices

- **147. Memory Optimization**
  - Memory allocation
  - Memory pools
  - Arena allocators
  - Custom allocators
  - Memory alignment
  - False sharing
  - Memory optimization best practices

- **148. Branch Optimization**
  - Branch prediction
  - Branch misprediction
  - Branchless programming
  - `[[likely]]`
  - `[[unlikely]]`
  - Branch optimization best practices

- **149. SIMD**
  - SIMD
  - SSE
  - AVX
  - AVX2
  - AVX-512
  - NEON
  - Intrinsics
  - Auto-vectorization
  - `std::simd` (C++26)
  - SIMD best practices

- **150. Multithreading Performance**
  - Thread pools
  - Work stealing
  - Lock-free data structures
  - False sharing
  - Thread affinity
  - Multithreading performance best practices

- **151. Benchmarking**
  - Benchmarking
  - Google Benchmark
  - Celero
  - Hayai
  - Benchmarking best practices
  - Benchmarking pitfalls

- **152. Low-Latency Programming**
  - Low-latency programming
  - Cache warming
  - Memory pinning
  - Huge pages
  - Real-time systems
  - Low-latency best practices

---

# XVIII. Systems Programming

- **153. Operating System Interfaces**
  - POSIX
  - Windows API
  - System calls
  - File descriptors
  - Process management
  - Signal handling
  - Inter-process communication
  - Shared memory
  - Pipes
  - Sockets
  - System programming best practices

- **154. Networking**
  - Sockets
  - TCP
  - UDP
  - IPv4
  - IPv6
  - Socket programming
  - `asio`
  - Boost.Asio
  - Networking best practices

- **155. Embedded Systems**
  - Embedded C++
  - Bare metal
  - RTOS
  - FreeRTOS
  - Zephyr
  - Memory constraints
  - Real-time constraints
  - Embedded best practices

- **156. Game Development**
  - Game engines
  - Unreal Engine
  - Unity (C++)
  - Godot
  - Custom engines
  - Graphics programming
  - Game loop
  - ECS
  - Entity Component System
  - Game development best practices

- **157. Graphics Programming**
  - OpenGL
  - Vulkan
  - DirectX
  - Metal
  - WebGPU
  - Shaders
  - Rendering pipelines
  - Graphics best practices

- **158. High-Performance Computing**
  - HPC
  - MPI
  - OpenMP
  - CUDA
  - OpenCL
  - SYCL
  - GPU programming
  - HPC best practices

- **159. Database Internals**
  - Database internals
  - Storage engines
  - B-trees
  - LSM trees
  - Query processing
  - Transactions
  - Concurrency control
  - Database internals best practices

- **160. Compilers and Interpreters**
  - Compiler design
  - Lexer
  - Parser
  - AST
  - Semantic analysis
  - IR
  - Optimization
  - Code generation
  - LLVM
  - Interpreter design
  - Compiler best practices

---

# XIX. Design Patterns and Architecture

- **161. Design Patterns**
  - Creational patterns
    - Singleton
    - Factory Method
    - Abstract Factory
    - Builder
    - Prototype
    - Object Pool
  - Structural patterns
    - Adapter
    - Bridge
    - Composite
    - Decorator
    - Facade
    - Flyweight
    - Proxy
  - Behavioral patterns
    - Chain of Responsibility
    - Command
    - Interpreter
    - Iterator
    - Mediator
    - Memento
    - Observer
    - State
    - Strategy
    - Template Method
    - Visitor
  - Concurrency patterns
  - Design pattern best practices

- **162. C++ Idioms**
  - RAII
  - Pimpl
  - Copy-and-swap
  - Non-virtual interface
  - Curiously Recurring Template Pattern
  - CRTP
  - Tag dispatch
  - SFINAE
  - Type erasure
  - Rule of Three/Five/Zero
  - Erase-remove idiom
  - Return value optimization
  - Named parameter idiom
  - Address of
  - Attorney-client
  - Base-from-member
  - Barton-Nackman trick
  - Boost mutant
  - Capability query
  - Coercion by member template
  - Construct on first use
  - Empty base optimization
  - Envelope letter
  - Fast Pimpl
  - Int-to-type
  - Interface class
  - Making new friends
  - Metafunction
  - Named constructor
  - Non-copyable
  - Null object
  - Parameterized base class
  - Policy-based design
  - Resource return
  - Scope guard
  - Shrink-to-fit
  - Small string optimization
  - Smart pointer
  - Substitution failure is not an error
  - Temporary base class
  - Trait class
  - Type generator
  - Type safe enum
  - Type traits
  - Value semantics
  - Virtual constructor
  - C++ idiom best practices

- **163. Architectural Patterns**
  - Layered architecture
  - Hexagonal architecture
  - Clean architecture
  - Onion architecture
  - MVC
  - MVP
  - MVVM
  - Microservices
  - Event-driven architecture
  - CQRS
  - Event sourcing
  - Component-based architecture
  - Data-oriented design
  - Architectural pattern best practices

- **164. SOLID Principles**
  - Single Responsibility Principle
  - Open/Closed Principle
  - Liskov Substitution Principle
  - Interface Segregation Principle
  - Dependency Inversion Principle
  - SOLID in C++
  - SOLID best practices

- **165. Data-Oriented Design**
  - Data-oriented design
  - DOD
  - Cache-friendly data
  - Structure of arrays
  - SoA
  - Array of structures
  - AoS
  - Entity Component System
  - ECS
  - Data-oriented design best practices

- **166. Domain-Driven Design**
  - DDD
  - Ubiquitous language
  - Bounded contexts
  - Entities
  - Value objects
  - Aggregates
  - Domain events
  - Repositories
  - DDD best practices

---

# XX. Security

- **167. Security Fundamentals**
  - Security
  - Threat modeling
  - Attack surface
  - Defense in depth
  - Least privilege
  - Secure defaults
  - Security best practices

- **168. Common Vulnerabilities**
  - Buffer overflows
  - Stack overflows
  - Heap overflows
  - Integer overflows
  - Format string vulnerabilities
  - Use-after-free
  - Double-free
  - Null pointer dereference
  - Race conditions
  - TOCTOU
  - Uninitialized memory
  - Memory leaks
  - Type confusion
  - Injection attacks
  - OWASP Top 10
  - CWE Top 25

- **169. Secure Coding**
  - Input validation
  - Bounds checking
  - Safe string handling
  - Safe integer arithmetic
  - Memory safety
  - RAII
  - Smart pointers
  - `std::span`
  - `std::string_view`
  - Secure coding best practices

- **170. Cryptography**
  - Cryptography
  - OpenSSL
  - libsodium
  - Crypto++
  - Hashing
  - Encryption
  - Digital signatures
  - Key management
  - Cryptography best practices

- **171. Static Analysis**
  - Clang-Tidy
  - Cppcheck
  - PVS-Studio
  - Coverity
  - SonarQube
  - Static analysis best practices

- **172. Dynamic Analysis**
  - AddressSanitizer
  - ThreadSanitizer
  - UndefinedBehaviorSanitizer
  - MemorySanitizer
  - Valgrind
  - Dynamic analysis best practices

- **173. Fuzz Testing**
  - Fuzz testing
  - libFuzzer
  - AFL
  - AFL++
  - Honggfuzz
  - OSS-Fuzz
  - Fuzz testing best practices

- **174. Secure Development Lifecycle**
  - Secure development
  - Threat modeling
  - Security requirements
  - Security design
  - Security implementation
  - Security testing
  - Security deployment
  - Security maintenance
  - Secure development best practices

---

# XXI. C++ Projects by Difficulty

## Beginner Projects

- **1. Calculator**
  - Functions
  - User input
  - Arithmetic operations
  - Error handling

- **2. To-Do List CLI**
  - `std::vector`
  - `std::string`
  - File I/O
  - CRUD operations

- **3. Bank Account System**
  - Classes
  - Inheritance
  - Polymorphism
  - Exception handling

- **4. Student Management System**
  - Classes
  - `std::vector`
  - File I/O
  - CRUD operations

- **5. Quiz Application**
  - Classes
  - `std::map`
  - Loops
  - User input

---

## Intermediate Projects

- **6. Library Management System**
  - OOP
  - STL containers
  - File I/O
  - Exception handling
  - Templates

- **7. Chat Application**
  - Sockets
  - `std::thread`
  - Mutexes
  - Networking

- **8. Matrix Library**
  - Templates
  - Operator overloading
  - Linear algebra
  - Performance optimization

- **9. JSON Parser**
  - Recursive descent parser
  - `std::variant`
  - `std::string_view`
  - Error handling

- **10. HTTP Server**
  - Sockets
  - `std::thread`
  - HTTP protocol
  - Routing
  - Request/response handling

---

## Advanced Projects

- **11. Game Engine**
  - ECS
  - Rendering
  - Physics
  - Input handling
  - Scene management
  - Performance optimization

- **12. Database Engine**
  - Storage engine
  - B-trees
  - Query parser
  - Query executor
  - Transactions
  - Concurrency control

- **13. Compiler**
  - Lexer
  - Parser
  - AST
  - Semantic analysis
  - Code generation
  - Optimization

- **14. High-Performance HTTP Server**
  - epoll
  - io_uring
  - Thread pools
  - Lock-free data structures
  - Memory pools
  - Performance tuning

- **15. Real-Time Trading System**
  - Low-latency
  - Lock-free programming
  - Memory pools
  - Cache optimization
  - Network programming
  - Performance tuning

---

## Expert Projects

- **16. Operating System Kernel**
  - Bootloader
  - Memory management
  - Process scheduling
  - File system
  - Device drivers
  - System calls

- **17. Distributed Database**
  - Distributed consensus
  - Replication
  - Sharding
  - Transactions
  - Fault tolerance
  - Performance

- **18. Machine Learning Framework**
  - Tensors
  - Autograd
  - Neural networks
  - GPU acceleration
  - Optimization
  - Deployment

- **19. Game Engine with Vulkan**
  - Vulkan
  - Rendering pipeline
  - Shaders
  - Physics
  - Animation
  - Performance

- **20. Compiler with LLVM**
  - Lexer
  - Parser
  - AST
  - LLVM IR
  - Optimization passes
  - Code generation
  - JIT compilation

---

# XXII. Progressive C++ Learning Sequence

## Level 1 — C++ Fundamentals

- Master:
  - Compilation model
  - Basic syntax
  - Variables
  - Data types
  - Operators
  - Control flow
  - Functions
  - Arrays

## Level 2 — Pointers and References

- Master:
  - Pointers
  - References
  - Const correctness
  - Memory management
  - RAII
  - Smart pointers

## Level 3 — Object-Oriented Programming

- Master:
  - Classes
  - Objects
  - Constructors
  - Destructors
  - Inheritance
  - Polymorphism
  - Virtual functions
  - Abstract classes
  - Operator overloading

## Level 4 — Templates and Generic Programming

- Master:
  - Function templates
  - Class templates
  - Template specialization
  - Variadic templates
  - Template metaprogramming
  - SFINAE
  - Concepts
  - CRTP

## Level 5 — STL

- Master:
  - Containers
  - Iterators
  - Algorithms
  - Function objects
  - Smart pointers
  - Utility components
  - Ranges

## Level 6 — Modern C++

- Master:
  - C++11 features
  - C++14 features
  - C++17 features
  - C++20 features
  - C++23 features
  - C++26 features

## Level 7 — Exception Handling and I/O

- Master:
  - Exceptions
  - Try-catch
  - Exception safety
  - Noexcept
  - I/O streams
  - File I/O
  - String streams
  - Filesystem
  - Formatting

## Level 8 — Concurrency

- Master:
  - Threads
  - Mutexes
  - Condition variables
  - Atomics
  - Futures
  - Promises
  - Memory model
  - Lock-free programming
  - Parallel algorithms
  - Coroutines

## Level 9 — Build Systems and Tooling

- Master:
  - CMake
  - Package managers
  - Compiler flags
  - Static analysis
  - Formatting
  - Documentation
  - Debugging
  - Profiling
  - Sanitizers
  - Testing frameworks

## Level 10 — Testing

- Master:
  - Unit testing
  - Google Test
  - Catch2
  - Mocking
  - Integration testing
  - Test automation
  - Benchmarking

## Level 11 — Performance Optimization

- Master:
  - Profiling
  - Compiler optimizations
  - Cache optimization
  - Memory optimization
  - Branch optimization
  - SIMD
  - Multithreading performance
  - Benchmarking
  - Low-latency programming

## Level 12 — Systems Programming

- Master:
  - Operating system interfaces
  - Networking
  - Embedded systems
  - Game development
  - Graphics programming
  - HPC
  - Database internals
  - Compilers

## Level 13 — Architecture and Design

- Master:
  - Design patterns
  - C++ idioms
  - Architectural patterns
  - SOLID principles
  - Data-oriented design
  - Domain-driven design

## Level 14 — Security

- Master:
  - Security fundamentals
  - Common vulnerabilities
  - Secure coding
  - Cryptography
  - Static analysis
  - Dynamic analysis
  - Fuzz testing
  - Secure development lifecycle

## Level 15 — Production Engineering

- Master:
  - Build systems
  - CI/CD
  - Deployment
  - Monitoring
  - Profiling
  - Performance tuning
  - Code review
  - Documentation
  - Maintenance
  - Legacy modernization

---

# XXIII. Final C++ Competency Map

- **Foundations**

  - Compilation model
  - Syntax
  - Variables
  - Data types
  - Operators
  - Control flow
  - Functions
  - Arrays

- **Pointers and References**

  - Pointers
  - References
  - Const correctness
  - Memory management
  - RAII
  - Smart pointers

- **OOP**

  - Classes
  - Objects
  - Constructors
  - Destructors
  - Inheritance
  - Polymorphism
  - Virtual functions
  - Abstract classes
  - Operator overloading
  - Special member functions
  - Friend functions
  - Static members

- **Templates**

  - Function templates
  - Class templates
  - Template specialization
  - Variadic templates
  - Template metaprogramming
  - SFINAE
  - Concepts
  - CRTP

- **STL**

  - Containers
  - Iterators
  - Algorithms
  - Function objects
  - Smart pointers
  - Utility components
  - Ranges
  - Time and date

- **Modern C++**

  - C++11
  - C++14
  - C++17
  - C++20
  - C++23
  - C++26

- **Exception Handling**

  - Exceptions
  - Try-catch
  - Exception safety
  - Noexcept
  - Custom exceptions
  - Error handling patterns

- **I/O**

  - I/O streams
  - File I/O
  - String streams
  - Filesystem
  - Formatting
  - Regular expressions
  - Serialization

- **Concurrency**

  - Threads
  - Mutexes
  - Condition variables
  - Atomics
  - Futures
  - Promises
  - Memory model
  - Lock-free programming
  - Parallel algorithms
  - Coroutines

- **Build Systems**

  - CMake
  - Package managers
  - Compiler flags
  - Static analysis
  - Formatting
  - Documentation

- **Testing**

  - Unit testing
  - Google Test
  - Catch2
  - Mocking
  - Integration testing
  - Test automation
  - Benchmarking

- **Performance**

  - Profiling
  - Compiler optimizations
  - Cache optimization
  - Memory optimization
  - Branch optimization
  - SIMD
  - Multithreading performance
  - Low-latency programming

- **Systems Programming**

  - Operating system interfaces
  - Networking
  - Embedded systems
  - Game development
  - Graphics programming
  - HPC
  - Database internals
  - Compilers

- **Architecture**

  - Design patterns
  - C++ idioms
  - Architectural patterns
  - SOLID principles
  - Data-oriented design
  - Domain-driven design

- **Security**

  - Security fundamentals
  - Common vulnerabilities
  - Secure coding
  - Cryptography
  - Static analysis
  - Dynamic analysis
  - Fuzz testing
  - Secure development lifecycle

---

## Recommended Overall Progression

**C++ Fundamentals → Pointers and References → OOP → Templates → STL → Modern C++ → Exception Handling → I/O → Concurrency → Build Systems → Testing → Performance Optimization → Systems Programming → Design Patterns → Architecture → Security → Production Engineering**

For maximum practical mastery, combine this C++ roadmap with the DSA, Java, Python, JavaScript, Node.js, REST API, SQL, Discrete Mathematics, React, Laravel, jQuery, and Jupyter roadmaps above so the progression becomes:

**Discrete Mathematics → DSA Foundations → C++ Fundamentals → OOP → Templates → STL → Modern C++ → Concurrency → Build Systems → Testing → Performance Optimization → Systems Programming → Compilers → Database Internals → Game Development → Graphics Programming → HPC → Distributed Systems → Security → Architecture → Production C++ Engineering → Enterprise Systems → Low-Latency Systems → Operating Systems → Compiler Engineering.**