# Nested Classes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A nested class is a class **declared inside another class** (the enclosing class), scoping the nested class's name within the enclosing class and giving it special access to the enclosing class's members.

**Technical Definition**
A nested class (also called an *inner class*) is a class declared within the member-specification of another class. Since C++11, a nested class is a member of the enclosing class and has the same access rights as any other member: it can access all names (including private and protected members) to which the enclosing class has access. The nested class's name is in the scope of the enclosing class, so it is accessed as `Enclosing::Nested`. A nested class is **not** a special kind of object — it does not automatically have access to an instance of the enclosing class; it must be given one (e.g., via a pointer or reference parameter). The nested class's member functions can be defined outside the enclosing class using the `Enclosing::Nested::` qualification.

**Beginner-Friendly Explanation**
A nested class is a class **defined inside another class**, like a room inside a house. It belongs to the enclosing class and is named within it (`House::Room`). The nested class can see the enclosing class's private parts (since C++11), but it doesn't automatically have a specific house — it just knows the house's blueprint. Nested classes are useful for grouping tightly related helper types, iterators, and implementation details with the class they serve.

### Key Characteristics

- **Scoped name**: The nested class's name lives inside the enclosing class (`Enclosing::Nested`).
- **Member access**: Since C++11, nested classes can access all members of the enclosing class (including private).
- **No implicit object**: A nested class does not have an implicit `this` to the enclosing object; it must be given one.
- **Independent type**: A nested class is a distinct type, independent of any enclosing object.
- **Access control**: The nested class's name follows the enclosing class's access specifiers.
- **Definition location**: Can be defined inside the enclosing class or outside using `Enclosing::Nested`.
- **Forward declaration**: Nested classes can be forward-declared inside the enclosing class.
- **No special privileges for enclosing**: The enclosing class has no special access to the nested class's private members (unless declared a friend).

### Prerequisites

- Understanding of classes, objects, and member functions.
- Familiarity with access specifiers (`public`, `private`, `protected`).
- Knowledge of scope and name lookup.
- Basic awareness of friend declarations (for access directionality).

### Related Programming Areas

- **Encapsulation** – nested classes hide implementation details.
- **Iterators** – container iterator classes are often nested.
- **Helper types** – `Node`, `Impl`, `Builder` classes.
- **Pimpl idiom** – forward-declared nested implementation class.
- **Design patterns** – nested builders, visitors, and state classes.
- **Templates** – nested classes in class templates.

### Core Concepts / Features

1. Nested Classes
2. Inner Classes
3. Nested Class Access
4. Nested Class Scope
5. Nested Class Best Practices

---

## 1. Nested Classes

### Definitions

**Core Definition**
A nested class is a class declared inside another class, giving it a name scoped to the enclosing class and access to the enclosing class's members.

**Technical Definition**
A nested class is a member of the enclosing class, declared in its member-specification. Since C++11, the nested class has the same access rights as any other member of the enclosing class: it can access all names (including private and protected members) to which the enclosing class has access. The nested class is a complete type when defined; it can be used as a member type, in function signatures, and as a template argument. Nested classes do not have an implicit enclosing object; they are independent types.

**Beginner-Friendly Explanation**
A nested class is a **class inside a class**. It's like a component that belongs to the enclosing class. For example, a `LinkedList` class might have a nested `Node` class representing its elements.

### Purposes

- To group tightly related helper types with the class they serve.
- To hide implementation details (e.g., `Node`, `Impl`).
- To provide iterators and proxies scoped to the container.
- To support the Pimpl idiom.
- To organize complex classes with multiple related types.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Enclosing {
public:
    class Nested {
        // nested class body
    };
    // ...
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class Enclosing` | The outer class. |
| `class Nested` | The nested class (member of Enclosing). |
| Access | Nested follows Enclosing's access specifiers. |

**Syntax Rules**
- The nested class is declared in the enclosing class's member-specification.
- The nested class's name is in the enclosing class's scope (`Enclosing::Nested`).
- The nested class can be defined inline (inside the enclosing class) or outside.
- Since C++11, the nested class can access all members of the enclosing class.
- The enclosing class has no special access to the nested class's private members.

**Constraints and Limitations**
- A nested class does not have an implicit object of the enclosing class.
- The enclosing class cannot access the nested class's private members without friendship.
- Nested classes cannot be used before their declaration (unless forward-declared).

### Annotated Code Examples

**Example 1: Basic nested class**

```cpp
#include <iostream>

class LinkedList {
public:
    class Node {
    public:
        int value;
        Node* next;
        Node(int v) : value(v), next(nullptr) {}
    };

    LinkedList() : head_(nullptr) {}
    void push_front(int value) {
        Node* n = new Node(value);
        n->next = head_;
        head_ = n;
    }
    int front() const { return head_->value; }

private:
    Node* head_;
};

int main() {
    LinkedList list;
    list.push_front(42);
    std::cout << list.front() << '\n';
}
```

**Expected Output**
```
42
```

**Why this output?**  
`Node` is a nested class of `LinkedList`. `push_front` creates a `Node`; `front` reads its value.

### Real-World Cases

- **Containers** – `std::vector<T>::iterator` (implementation-defined nesting).
- **Linked lists** – nested `Node` class.
- **Parsers** – nested token or AST node classes.
- **Builders** – nested `Builder` class for fluent construction.
- **Pimpl** – nested `Impl` class.

---

## 2. Inner Classes

### Definitions

**Core Definition**
“Inner class” is a synonym for “nested class,” often used in other languages (e.g., Java, C#) and informally in C++ to emphasize that the class is inside another.

**Technical Definition**
In C++, “inner class” and “nested class” are used interchangeably. The term “inner class” sometimes carries the connotation of a class that is closely associated with the enclosing class's instances, but in C++ there is no special “inner” relationship beyond nesting. A nested class is a member of the enclosing class and, since C++11, has access to its private members, but it does not hold an implicit reference to an enclosing object.

**Beginner-Friendly Explanation**
“Inner class” means the same as “nested class” — a class **inside another class**. In C++, it's just a naming/scoping relationship; the inner class doesn't automatically belong to a specific outer object.

### Purposes

- (Synonym usage) To refer to nested classes in documentation and discussion.
- To emphasize the close relationship between the nested and enclosing classes.
- To align with terminology from other languages.

### Syntax Rules and Structure

Same as nested classes:
```cpp
class Outer {
    class Inner {   // inner class == nested class
        // ...
    };
};
```

**Component Breakdown**

| Term | Meaning in C++ |
|------|----------------|
| Nested class | Class declared inside another class. |
| Inner class | Synonym for nested class. |

**Syntax Rules**
- Same syntax as nested classes.
- No implicit enclosing object (unlike Java's non-static inner classes).
- Access to enclosing class members since C++11.

**Constraints and Limitations**
- Do not confuse C++ nested classes with Java inner classes (which can hold an implicit outer reference).
- C++ nested classes are independent types.

### Annotated Code Examples

**Example 1: Inner class terminology**

```cpp
#include <iostream>

class Outer {
public:
    class Inner {   // inner/nested class
    public:
        void greet() const { std::cout << "Inner\n"; }
    };
};

int main() {
    Outer::Inner inner;   // qualified name
    inner.greet();
}
```

**Expected Output**
```
Inner
```

**Why this output?**  
`Inner` is a nested (inner) class of `Outer`; it is accessed as `Outer::Inner`.

### Real-World Cases

- **Documentation** – using “inner class” to describe nested types.
- **Cross-language discussions** – comparing C++ nested classes with Java inner classes.
- **Teaching** – introducing the concept of nesting.

---

## 3. Nested Class Access

### Definitions

**Core Definition**
Nested class access refers to the rules governing which members of the enclosing class and the nested class can be accessed from each other.

**Technical Definition**
Since C++11, a nested class is a member of the enclosing class and has the same access rights as any other member: it can access all members of the enclosing class (including private and protected). However, the **enclosing class has no special access** to the nested class's private members; the nested class must declare the enclosing class (or specific functions) as a friend to grant access. A nested class also does not have an implicit object of the enclosing class; to access non-static members of the enclosing class, it must be given an object (reference or pointer).

**Beginner-Friendly Explanation**
The nested class can **see the enclosing class's private members** (since C++11), but the enclosing class **cannot see the nested class's private members** unless the nested class grants friendship. Also, the nested class doesn't have an automatic reference to an enclosing object — it needs one to access non-static members.

### Purposes

- To control access between nested and enclosing classes.
- To implement helpers that need private access to the enclosing class.
- To use friendship for mutual access when needed.
- To design encapsulated nested types.

### Syntax Rules and Structure

**Access Directionality**

| Direction | Access |
|-----------|--------|
| Nested → Enclosing (all members) | Yes (since C++11). |
| Enclosing → Nested (public members) | Yes. |
| Enclosing → Nested (private members) | No (unless friendship). |
| Nested → Enclosing non-static members | Requires an enclosing object. |

**Syntax Rules**
- The nested class can access private members of the enclosing class.
- The enclosing class can access public members of the nested class.
- The enclosing class needs friendship to access private members of the nested class.
- The nested class needs an enclosing object to access non-static members.

**Constraints and Limitations**
- No implicit enclosing object; the nested class must be given one.
- Friendship is not mutual by default.
- Access to enclosing private members is a C++11 feature (pre-C++11 required friendship).

### Annotated Code Examples

**Example 1: Nested class accessing enclosing private members**

```cpp
#include <iostream>

class Enclosing {
    int secret_ = 42;
public:
    class Nested {
    public:
        void reveal(const Enclosing& e) {
            std::cout << e.secret_ << '\n';   // access private member
        }
    };
};

int main() {
    Enclosing e;
    Enclosing::Nested n;
    n.reveal(e);
}
```

**Expected Output**
```
42
```

**Why this output?**  
The nested class can access `Enclosing::secret_` (since C++11) when given an `Enclosing` object.

**Example 2: Enclosing accessing nested private members via friendship**

```cpp
#include <iostream>

class Enclosing {
public:
    class Nested {
        int value_ = 99;
        friend class Enclosing;   // grant access
    };

    void show(const Nested& n) {
        std::cout << n.value_ << '\n';   // access via friendship
    }
};

int main() {
    Enclosing e;
    Enclosing::Nested n;
    e.show(n);
}
```

**Expected Output**
```
99
```

**Why this output?**  
`Enclosing` is a friend of `Nested`, so it can access `Nested::value_`.

### Real-World Cases

- **Iterators** – iterator classes access container internals.
- **Pimpl** – the enclosing class accesses the nested `Impl`'s members.
- **Builders** – the enclosing class reads builder state.
- **Helpers** – nested helpers that need private access.

---

## 4. Nested Class Scope

### Definitions

**Core Definition**
Nested class scope refers to the name lookup rules: the nested class's name is in the enclosing class's scope, and the nested class's own members are in its own scope.

**Technical Definition**
The nested class's name is a member of the enclosing class, so it is accessed as `Enclosing::Nested`. Inside the nested class, name lookup searches the nested class's scope, then the enclosing class's scope, then the enclosing namespaces. The nested class can use the enclosing class's types and static members directly. The enclosing class cannot use the nested class's members without qualification. Nested classes can be forward-declared inside the enclosing class.

**Beginner-Friendly Explanation**
The nested class's name lives **inside the enclosing class's scope**. You write `Outer::Inner`. Inside `Inner`, you can see `Outer`'s names (including private ones since C++11). Nested classes can be forward-declared inside the enclosing class, allowing circular references.

### Purposes

- To organize related names within the enclosing class.
- To enable name lookup from nested to enclosing scope.
- To support forward declarations and circular references.
- To avoid polluting the enclosing namespace.

### Syntax Rules and Structure

**Name Lookup Order (inside nested class)**
1. Nested class's own scope.
2. Enclosing class's scope.
3. Enclosing namespaces and global scope.

**Syntax Rules**
- Nested class names are qualified with `Enclosing::`.
- Inside the nested class, the enclosing class's names are visible.
- Nested classes can be forward-declared inside the enclosing class.
- Nested classes can be defined outside the enclosing class using `Enclosing::Nested::`.

**Constraints and Limitations**
- The nested class cannot use the enclosing class's non-static members without an object.
- Name lookup may find enclosing names before global names, which can shadow.
- Nested classes in templates have dependent scope.

### Annotated Code Examples

**Example 1: Nested class scope and forward declaration**

```cpp
#include <iostream>

class Outer {
public:
    class Inner;   // forward declaration inside Outer
    void use_inner(Inner& i);
};

class Outer::Inner {   // definition outside
public:
    void greet() const { std::cout << "Inner\n"; }
};

void Outer::use_inner(Inner& i) {
    i.greet();
}

int main() {
    Outer::Inner inner;
    Outer o;
    o.use_inner(inner);
}
```

**Expected Output**
```
Inner
```

**Why this output?**  
`Inner` is forward-declared inside `Outer`, defined outside as `Outer::Inner`, and used by `Outer::use_inner`.

### Real-World Cases

- **Circular references** – nested classes referring to the enclosing class.
- **Header organization** – forward-declaring nested helpers.
- **Templates** – nested types in class templates.
- **Implementation details** – hiding nested classes in `.cpp` files.

---

## 5. Nested Class Best Practices

### Definitions

**Core Definition**
Nested class best practices are guidelines for declaring, defining, and using nested classes correctly, safely, and idiomatically.

**Technical Definition**
These practices are based on the C++ Core Guidelines and community standards. Key principles: use nested classes for tightly related helper types; prefer nesting for implementation details (e.g., `Node`, `Impl`); avoid deep nesting; use forward declarations for circular references; be explicit about access (friendship when needed); and prefer composition or separate types when the relationship is not tight.

**Beginner-Friendly Explanation**
Best practices for nested classes are simple: **nest classes only when they're tightly related**, **use nesting to hide implementation details**, **avoid deep nesting**, **forward-declare when needed**, and **be clear about access**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Tight relationship** | Nest only tightly related types. | `Node` inside `List`, `Impl` inside `Widget`. |
| **Hide implementation** | Use nesting for implementation details. | Keep helpers out of the public namespace. |
| **Avoid deep nesting** | Deeply nested classes are hard to read. | Flatten when possible. |
| **Forward declarations** | Use forward declarations for circular references. | Declare `class Inner;` inside the outer class. |
| **Explicit access** | Use friendship when the outer class needs inner access. | `friend class Outer;`. |
| **Prefer separate types** | If the relationship is not tight, use a separate class. | Avoid unnecessary nesting. |
| **Document** | Document nested types and their purpose. | Comments and API docs. |

### Annotated Code Examples

**Example 1: Tightly related nested class**

```cpp
#include <iostream>

class Tree {
public:
    class Node {   // tightly related helper
    public:
        int value;
        Node* left = nullptr;
        Node* right = nullptr;
        explicit Node(int v) : value(v) {}
    };

    Tree() : root_(nullptr) {}
    void insert(int value) {
        insert(root_, value);
    }
    void print() const { print(root_); }

private:
    Node* root_;
    void insert(Node*& node, int value) {
        if (!node) { node = new Node(value); return; }
        if (value < node->value) insert(node->left, value);
        else insert(node->right, value);
    }
    void print(const Node* node) const {
        if (!node) return;
        print(node->left);
        std::cout << node->value << ' ';
        print(node->right);
    }
};

int main() {
    Tree t;
    t.insert(5);
    t.insert(3);
    t.insert(7);
    t.print();
    std::cout << '\n';
}
```

**Expected Output**
```
3 5 7 
```

**Why this is best practice?**  
`Node` is tightly related to `Tree` and hidden inside it, avoiding namespace pollution.

**Example 2: Pimpl idiom with nested `Impl`**

```cpp
#include <iostream>
#include <memory>

class Widget {
public:
    Widget() : impl_(std::make_unique<Impl>()) {}
    void show() const { impl_->show(); }
private:
    class Impl {   // nested implementation detail
    public:
        void show() const { std::cout << "Widget::Impl\n"; }
    };
    std::unique_ptr<Impl> impl_;
};

int main() {
    Widget w;
    w.show();
}
```

**Expected Output**
```
Widget::Impl
```

**Why this is best practice?**  
`Impl` is a nested implementation detail, hidden from the public interface (Pimpl idiom).

### Real-World Cases

- **Standard library** – iterators and node types are nested.
- **Pimpl idiom** – nested `Impl` class.
- **Builders** – nested `Builder` class.
- **Parsers** – nested AST node classes.
- **Code review** – nested classes are reviewed for tight coupling.

---

## References

- Nested classes (cppreference) – https://en.cppreference.com/w/cpp/language/nested_types
- Nested class declarations (Microsoft Learn) – https://learn.microsoft.com/en-us/cpp/cpp/nested-class-declarations
- C++ Core Guidelines — C.1: Organize related data into structures – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-org
- C++ Core Guidelines — C.2: Use `class` if the class has an invariant; use `struct` if the data members can vary independently – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-struct
- C++ Core Guidelines — C.8: Use `class` rather than `struct` if there is an invariant – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-class
- Microsoft Learn — Nested types – https://learn.microsoft.com/en-us/cpp/cpp/nested-class-declarations
- cppreference — Member access – https://en.cppreference.com/w/cpp/language/access
- isocpp.org FAQ — Nested classes – https://isocpp.org/wiki/faq/classes-and-objects#nested-classes

---

This cheat sheet provides a complete, structured reference for C++ nested classes, covering nested classes, inner classes, nested class access, nested class scope, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.