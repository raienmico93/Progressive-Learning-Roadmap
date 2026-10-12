# Java Comparable and Comparator: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `Comparable` and `Comparator` are two interfaces in Java that define how objects are compared and ordered. `Comparable` defines an object's **natural ordering** (the default way objects of that class are compared), while `Comparator` defines **custom orderings** that are external to the class being compared.

**Technical Definition:** The `Comparable<T>` interface imposes a total ordering on the objects of each class that implements it. This ordering is referred to as the class's **natural ordering**, and the class's `compareTo` method is referred to as its **natural comparison method**. The `Comparator<T>` interface is a separate object that defines a custom ordering for objects of another class, allowing multiple different orderings without modifying the original class. Both interfaces return a negative integer, zero, or a positive integer to indicate whether the first object is less than, equal to, or greater than the second object.

**Beginner-Friendly Explanation:** Think of `Comparable` as a built-in sorting rule that objects of a class carry with them — like how numbers know they can be sorted from smallest to largest. Think of `Comparator` as an external sorting tool you can create whenever you need a different sorting rule — like sorting a list of people by age instead of by name. `Comparable` says "I know how to compare myself to others." `Comparator` says "I know how to compare two other objects."

### Key Characteristics

- **Natural vs. Custom Ordering:** `Comparable` defines the single, default natural ordering for a class; `Comparator` defines any number of custom, external orderings.
- **Single vs. Multiple:** A class can implement `Comparable` only once (one natural ordering), but you can create as many `Comparator` implementations as needed.
- **Class Modification:** `Comparable` requires modifying the class itself; `Comparator` does not require any changes to the class being compared.
- **Consistency with equals:** It is strongly recommended (though not strictly required) that natural orderings be consistent with `equals` — that is, `x.compareTo(y) == 0` if and only if `x.equals(y)`.
- **Contract Requirements:** Both interfaces have strict contracts: the comparison must be antisymmetric, transitive, and reflexive.
- **Functional Interface:** Since Java 8, `Comparator` is a functional interface, enabling lambda expressions and method references.

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with generics (`Comparable<T>`, `Comparator<T>`)
- Knowledge of the Java Collections Framework (`List`, `Set`, `Map`)
- Understanding of the `equals()` and `hashCode()` contracts
- Awareness of sorting algorithms and the `Collections.sort()` and `Arrays.sort()` methods

### Related Programming Areas

- **Collections Framework:** Sorting and ordering of `List`, `Set`, and `Map` elements.
- **Stream API:** Comparators are essential for `Stream.sorted()` and `Collectors.toMap()`.
- **Tree-based Collections:** `TreeSet` and `TreeMap` rely on `Comparable` or `Comparator` for ordering.
- **Searching:** Binary search (`Collections.binarySearch()`) requires sorted collections.
- **Algorithm Design:** Priority queues, merge sort, and other algorithms depend on comparison logic.

---

## Core Concepts / Features

---

## 1. Comparable

### Definitions

**Core Definition:** `Comparable` is an interface that defines the natural ordering of objects of a class. A class that implements `Comparable` declares that its instances can be compared to each other and sorted automatically.

**Technical Definition:** `Comparable<T>` is a generic interface in the `java.lang` package. It declares a single method `compareTo(T o)`. "This interface imposes a total ordering on the objects of each class that implements it. This ordering is referred to as the class's natural ordering, and the class's `compareTo` function is referred to as its natural comparison method". Arrays of objects that implement this interface can be sorted automatically by `Arrays.sort()`. Objects of these classes can also be used as keys in `TreeMap` without specifying a `Comparator`. Classes that implement `Comparable` include `String`, `Integer`, `Double`, `Date`, `File`, and many others.

**Beginner-Friendly Explanation:** `Comparable` is like a built-in sorting rule that objects of a class carry with them. When you implement `Comparable`, you're saying: "Here's how to compare two objects of my class." This allows Java to sort your objects automatically using methods like `Collections.sort()` or `Arrays.sort()`.

### Purposes

- To define a natural, default ordering for a class
- To enable automatic sorting of collections and arrays
- To allow objects to be used as keys in sorted collections (`TreeMap`, `TreeSet`)
- To provide a consistent, single comparison method for a class

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class ClassName implements Comparable<ClassName> {
    @Override
    public int compareTo(ClassName other) {
        // return negative, zero, or positive
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Comparable<ClassName>` | The generic interface to implement; `ClassName` is the type being compared. |
| `compareTo(ClassName other)` | The single method to implement. |
| Return value | Negative if `this` < `other`; zero if equal; positive if `this` > `other`. |

**Syntax Rules:**

- The class must implement `Comparable<T>` where `T` is the class itself.
- `compareTo` must be antisymmetric: `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`.
- `compareTo` must be transitive: if `x > y` and `y > z`, then `x > z`.
- It is strongly recommended that `x.equals(y)` implies `x.compareTo(y) == 0`.

**Constraints and Limitations:**

- A class can implement `Comparable` only once — there is only one natural ordering.
- `compareTo` throws `ClassCastException` if the argument type prevents comparison.
- Violating the contract can cause unpredictable behaviour in sorted collections.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Comparable with a Student Class**

```java
// File: ComparableDemo.java
import java.util.*;

public class ComparableDemo {
    static class Student implements Comparable<Student> {
        private String name;
        private int age;

        public Student(String name, int age) {
            this.name = name;
            this.age = age;
        }

        // Natural ordering: by age
        @Override
        public int compareTo(Student other) {
            // Returns negative if this.age < other.age
            // Returns zero if equal
            // Returns positive if this.age > other.age
            return Integer.compare(this.age, other.age);
        }

        @Override
        public String toString() {
            return name + " (" + age + ")";
        }

        public static void main(String[] args) {
            List<Student> students = new ArrayList<>();
            students.add(new Student("Alice", 25));
            students.add(new Student("Bob", 20));
            students.add(new Student("Charlie", 30));

            // Collections.sort uses natural ordering (Comparable)
            Collections.sort(students);

            for (Student s : students) {
                System.out.println(s);
            }
        }
    }
}
```

**Expected Output:**
```
Bob (20)
Alice (25)
Charlie (30)
```

**Step-by-Step Explanation:**

1. `Student` implements `Comparable<Student>`.
2. `compareTo` compares students by age using `Integer.compare()`.
3. `Collections.sort(students)` uses the natural ordering defined by `compareTo`.
4. Students are sorted in ascending order of age.

**Example 2: Natural Ordering with Strings**

```java
// File: NaturalOrderingDemo.java
import java.util.*;

public class NaturalOrderingDemo {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Charlie", "Alice", "Bob");

        // String implements Comparable<String> — lexicographic order
        Collections.sort(names);

        System.out.println(names); // [Alice, Bob, Charlie]
    }
}
```

**Expected Output:**
```
[Alice, Bob, Charlie]
```

**Step-by-Step Explanation:**

1. `String` implements `Comparable<String>` with lexicographic order.
2. `Collections.sort()` uses the natural ordering of `String`.
3. Strings are sorted alphabetically: Alice, Bob, Charlie.

### Real-World Cases

- **Sorting Names:** A `Person` class implements `Comparable` to sort by last name.
- **Sorting Dates:** The `Date` class implements `Comparable` for chronological ordering.
- **Sorting Products:** A `Product` class implements `Comparable` to sort by price or ID.
- **Natural IDs:** An `Employee` class implements `Comparable` to sort by employee ID.

---

## 2. compareTo()

### Definitions

**Core Definition:** `compareTo()` is the single method declared in the `Comparable` interface. It compares the current object (`this`) with another object of the same type and returns an integer indicating their relative order.

**Technical Definition:** `int compareTo(T o)` compares this object with the specified object for order. "Returns a negative integer, zero, or a positive integer as this Object is less than, equal to, or greater than the given Object". The implementor must ensure `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`, the relation is transitive, and `x.equals(y)` implies `x.compareTo(y) == 0`.

**Beginner-Friendly Explanation:** `compareTo()` is like a referee comparing two players. It says: "You (the current object) are better, equal to, or worse than the other player." Negative means "I'm less," zero means "we're equal," positive means "I'm greater."

### Purposes

- To define the natural ordering of a class
- To enable automatic sorting by Java's built-in sorting methods
- To provide a consistent comparison mechanism for all instances of a class

### Syntax Rules and Structure

```java
@Override
public int compareTo(T other) {
    // Return negative, zero, or positive
}
```

**Component Breakdown:**

| Return Value | Meaning |
|--------------|---------|
| Negative (< 0) | `this` is less than `other` |
| Zero (0) | `this` is equal to `other` |
| Positive (> 0) | `this` is greater than `other` |

**Syntax Rules:**

- Must be antisymmetric: `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`.
- Must be transitive: if `x > y` and `y > z`, then `x > z`.
- Strongly recommended: consistent with `equals`.
- Throws `ClassCastException` if the argument type is incompatible.
- Throws `NullPointerException` if the argument is `null`.

**Constraints and Limitations:**

- `compareTo` must not rely on mutable state that can change during sorting.
- Violating the contract can cause `TreeSet` or `TreeMap` to behave incorrectly.

### Annotated Code Example

```java
// File: CompareToDemo.java
import java.util.*;

public class CompareToDemo {
    static class Person implements Comparable<Person> {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public int compareTo(Person other) {
            // Primary: compare by name
            int nameComparison = this.name.compareTo(other.name);
            if (nameComparison != 0) {
                return nameComparison;
            }
            // Secondary: compare by age
            return Integer.compare(this.age, other.age);
        }

        @Override
        public String toString() {
            return name + " (" + age + ")";
        }

        public static void main(String[] args) {
            List<Person> people = new ArrayList<>();
            people.add(new Person("Charlie", 30));
            people.add(new Person("Alice", 25));
            people.add(new Person("Bob", 20));
            people.add(new Person("Alice", 35));

            Collections.sort(people);

            for (Person p : people) {
                System.out.println(p);
            }
        }
    }
}
```

**Expected Output:**
```
Alice (25)
Alice (35)
Bob (20)
Charlie (30)
```

**Step-by-Step Explanation:**

1. `compareTo` first compares names lexicographically.
2. If names are equal, it compares ages numerically.
3. `Collections.sort()` uses this natural ordering.
4. "Alice" comes before "Bob", and the two Alices are sorted by age.

### Real-World Cases

- **Employee Sorting:** Sort employees by last name, then first name.
- **Product Sorting:** Sort products by category, then by price.
- **Book Sorting:** Sort books by author, then by title.

---

## 3. Natural Ordering

### Definitions

**Core Definition:** Natural ordering is the default, single ordering that a class defines for itself by implementing the `Comparable` interface.

**Technical Definition:** "This ordering is referred to as the class's natural ordering, and the class's `compareTo` function is referred to as its natural comparison method". The natural ordering for a class `C` is said to be consistent with equals if and only if `e1.compareTo(e2) == 0` has the same boolean value as `e1.equals(e2)` for every `e1` and `e2` of class `C`. Virtually all Java core classes that implement `Comparable` have natural orderings that are consistent with equals.

**Beginner-Friendly Explanation:** Natural ordering is the "default sort order" for a class. For example, `String` objects naturally sort alphabetically, and `Integer` objects naturally sort numerically. When you sort a list of these objects, Java knows exactly how to order them without any extra instructions.

### Purposes

- To provide a consistent, default ordering for a class
- To enable automatic sorting without specifying a comparator
- To allow objects to be used in `TreeSet` and `TreeMap` without a comparator

### Syntax Rules and Structure

**Natural Ordering Examples:**

| Class | Natural Ordering |
|-------|-----------------|
| `String` | Lexicographic (alphabetical) |
| `Integer` | Numeric ascending |
| `Double` | Numeric ascending |
| `Date` | Chronological (oldest first) |
| `Boolean` | `false` before `true` |
| `Character` | Unicode value order |

**Syntax Rules:**

- A class defines its natural ordering by implementing `Comparable<T>`.
- `Collections.sort(list)` uses natural ordering.
- `Arrays.sort(array)` uses natural ordering.
- `TreeSet` and `TreeMap` use natural ordering by default.

**Constraints and Limitations:**

- Only one natural ordering per class.
- Natural ordering should be consistent with `equals` for correct `SortedSet`/`SortedMap` behaviour.

### Annotated Code Example

```java
// File: NaturalOrderingDemo2.java
import java.util.*;

public class NaturalOrderingDemo2 {
    public static void main(String[] args) {
        // Integer natural ordering
        List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9);
        Collections.sort(numbers);
        System.out.println("Numbers: " + numbers); // [1, 2, 5, 8, 9]

        // String natural ordering
        List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
        Collections.sort(names);
        System.out.println("Names: " + names); // [Alice, Bob, Charlie]

        // Date natural ordering
        List<Date> dates = new ArrayList<>();
        dates.add(new Date(3000));
        dates.add(new Date(1000));
        dates.add(new Date(2000));
        Collections.sort(dates);
        System.out.println("Dates: " + dates); // chronological
    }
}
```

**Expected Output:**
```
Numbers: [1, 2, 5, 8, 9]
Names: [Alice, Bob, Charlie]
Dates: [Thu Jan 01 00:00:01 UTC 1970, Thu Jan 01 00:00:02 UTC 1970, Thu Jan 01 00:00:03 UTC 1970]
```

**Step-by-Step Explanation:**

1. `Integer` implements `Comparable` with numeric ordering.
2. `String` implements `Comparable` with lexicographic ordering.
3. `Date` implements `Comparable` with chronological ordering.
4. `Collections.sort()` uses natural ordering for each type.

### Real-World Cases

- **File Sorting:** `File` implements `Comparable` for path-based ordering.
- **BigDecimal:** Implements `Comparable` for numeric ordering.
- **Enums:** Enum constants have a natural ordering based on declaration order.

---

## 4. Comparator

### Definitions

**Core Definition:** `Comparator` is an interface that defines a custom ordering for objects of a class, separate from the class itself.

**Technical Definition:** `Comparator<T>` is a functional interface in the `java.util` package. It declares a single abstract method `compare(T o1, T o2)`. Unlike `Comparable`, which is implemented by the class itself, `Comparator` is a separate class or object that defines the order of objects. It allows you to create multiple custom orderings without modifying the original class.

**Beginner-Friendly Explanation:** `Comparator` is like a ruler you can use to measure and compare objects in any way you choose. You can have different rulers for different purposes — one for sorting by name, another for sorting by age. You don't need to modify the objects themselves to use a different ruler.

### Purposes

- To define custom orderings that differ from the natural ordering
- To create multiple different orderings for the same class
- To sort objects from third-party libraries you cannot modify
- To provide context-specific sorting (e.g., sort employees by department, then by name)

### Syntax Rules and Structure

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
    // Plus many static and default methods (Java 8+)
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `compare(T o1, T o2)` | Compares two objects. Returns negative if `o1 < o2`, zero if equal, positive if `o1 > o2`. |

**Syntax Rules:**

- `Comparator` is a functional interface — can be implemented with a lambda expression.
- The comparison must be antisymmetric and transitive.
- It is recommended (but not required) that `compare` be consistent with `equals`.
- Since Java 8, `Comparator` provides many static and default methods for building comparators.

**Constraints and Limitations:**

- `compare` must handle `null` values or explicitly throw `NullPointerException`.
- A `Comparator` should not modify the objects it compares.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Comparator with a Lambda Expression**

```java
// File: ComparatorDemo.java
import java.util.*;

public class ComparatorDemo {
    static class Student {
        private String name;
        private int age;

        public Student(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() { return name; }
        public int getAge() { return age; }

        @Override
        public String toString() {
            return name + " (" + age + ")";
        }
    }

    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("Alice", 25));
        students.add(new Student("Bob", 20));
        students.add(new Student("Charlie", 30));

        // Comparator: sort by name
        Comparator<Student> byName = (s1, s2) -> s1.getName().compareTo(s2.getName());
        Collections.sort(students, byName);
        System.out.println("By name: " + students);

        // Comparator: sort by age
        Comparator<Student> byAge = (s1, s2) -> Integer.compare(s1.getAge(), s2.getAge());
        Collections.sort(students, byAge);
        System.out.println("By age: " + students);
    }
}
```

**Expected Output:**
```
By name: [Alice (25), Bob (20), Charlie (30)]
By age: [Bob (20), Alice (25), Charlie (30)]
```

**Step-by-Step Explanation:**

1. `byName` is a lambda comparator that compares students by name.
2. `Collections.sort(students, byName)` sorts using the custom comparator.
3. `byAge` is another comparator that sorts by age.
4. The same list is sorted two different ways without modifying the `Student` class.

**Example 2: Comparator with an Anonymous Inner Class**

```java
// File: ComparatorAnonymousDemo.java
import java.util.*;

public class ComparatorAnonymousDemo {
    static class Product {
        private String name;
        private double price;

        public Product(String name, double price) {
            this.name = name;
            this.price = price;
        }

        public String getName() { return name; }
        public double getPrice() { return price; }

        @Override
        public String toString() {
            return name + " ($" + price + ")";
        }
    }

    public static void main(String[] args) {
        List<Product> products = new ArrayList<>();
        products.add(new Product("Laptop", 1200.00));
        products.add(new Product("Phone", 800.00));
        products.add(new Product("Tablet", 500.00));

        // Anonymous inner class Comparator
        Collections.sort(products, new Comparator<Product>() {
            @Override
            public int compare(Product p1, Product p2) {
                return Double.compare(p1.getPrice(), p2.getPrice());
            }
        });

        System.out.println(products);
    }
}
```

**Expected Output:**
```
[Tablet ($500.0), Phone ($800.0), Laptop ($1200.0)]
```

**Step-by-Step Explanation:**

1. An anonymous inner class implements `Comparator<Product>`.
2. `compare` compares products by price.
3. `Collections.sort()` uses this comparator to sort by price ascending.

### Real-World Cases

- **E-commerce:** Sorting products by price, rating, or popularity.
- **Human Resources:** Sorting employees by salary, hire date, or department.
- **File Management:** Sorting files by size, name, or modification date.

---

## 5. compare()

### Definitions

**Core Definition:** `compare()` is the single abstract method declared in the `Comparator` interface. It compares two objects and returns an integer indicating their relative order.

**Technical Definition:** `int compare(T o1, T o2)` compares its two arguments for order. "Returns a negative integer, zero, or a positive integer as the first argument is less than, equal to, or greater than the second". The implementor must ensure that `sgn(compare(x, y)) == -sgn(compare(y, x))` for all `x` and `y`.

**Beginner-Friendly Explanation:** `compare()` is like a judge comparing two contestants. It says: "Contestant 1 is better, equal to, or worse than Contestant 2." Negative means "1 is less," zero means "they're equal," positive means "1 is greater."

### Purposes

- To define the comparison logic for a custom ordering
- To enable sorting of objects that do not implement `Comparable`
- To allow multiple different comparison strategies for the same class

### Syntax Rules and Structure

```java
@Override
public int compare(T o1, T o2) {
    // Return negative, zero, or positive
}
```

**Component Breakdown:**

| Return Value | Meaning |
|--------------|---------|
| Negative (< 0) | `o1` is less than `o2` |
| Zero (0) | `o1` is equal to `o2` |
| Positive (> 0) | `o1` is greater than `o2` |

**Syntax Rules:**

- Must be antisymmetric: `sgn(compare(x, y)) == -sgn(compare(y, x))`.
- Must be transitive: if `compare(x, y) > 0` and `compare(y, z) > 0`, then `compare(x, z) > 0`.
- Must be consistent: repeated calls with the same arguments return the same result.
- Should handle `null` gracefully or throw `NullPointerException`.

**Constraints and Limitations:**

- Violating the contract can cause `Collections.sort()` or `Arrays.sort()` to throw `IllegalArgumentException`.
- `compare` should not modify the objects being compared.

### Annotated Code Example

```java
// File: CompareMethodDemo.java
import java.util.*;

public class CompareMethodDemo {
    static class Book {
        private String title;
        private String author;
        private int year;

        public Book(String title, String author, int year) {
            this.title = title;
            this.author = author;
            this.year = year;
        }

        public String getTitle() { return title; }
        public String getAuthor() { return author; }
        public int getYear() { return year; }

        @Override
        public String toString() {
            return title + " by " + author + " (" + year + ")";
        }
    }

    public static void main(String[] args) {
        List<Book> books = new ArrayList<>();
        books.add(new Book("Java Basics", "Smith", 2020));
        books.add(new Book("Advanced Java", "Jones", 2018));
        books.add(new Book("Java Patterns", "Smith", 2019));

        // Comparator: by author, then by year
        Comparator<Book> byAuthorThenYear = new Comparator<Book>() {
            @Override
            public int compare(Book b1, Book b2) {
                int authorComparison = b1.getAuthor().compareTo(b2.getAuthor());
                if (authorComparison != 0) {
                    return authorComparison;
                }
                return Integer.compare(b1.getYear(), b2.getYear());
            }
        };

        Collections.sort(books, byAuthorThenYear);

        for (Book b : books) {
            System.out.println(b);
        }
    }
}
```

**Expected Output:**
```
Advanced Java by Jones (2018)
Java Patterns by Smith (2019)
Java Basics by Smith (2020)
```

**Step-by-Step Explanation:**

1. `compare` first compares authors.
2. If authors are equal, it compares years.
3. Books by Jones come first, then Smith's books sorted by year.

### Real-World Cases

- **Multi-Field Sorting:** Sorting by last name, then first name.
- **Reverse Sorting:** Creating a comparator that reverses the natural ordering.
- **Null-Safe Sorting:** Using `Comparator.nullsFirst()` or `nullsLast()`.

---

## 6. Custom Ordering

### Definitions

**Core Definition:** Custom ordering is any ordering of objects that differs from the natural ordering, defined through a `Comparator` implementation.

**Technical Definition:** "A `Comparator<T>` defines an alternate order—a way of comparing objects that differs from the natural order. Unlike `Comparable`, which is built into the class itself, a `Comparator` is a separate class whose sole purpose is to define a particular ordering". Custom orderings allow you to sort objects by different fields, in different directions, or using different criteria, without modifying the original class.

**Beginner-Friendly Explanation:** Custom ordering is like having a set of different sorting rules for the same collection of items. You might sort your music by artist, then by album, then by track number — each is a different custom ordering.

### Purposes

- To sort objects by different attributes (name, age, price, etc.)
- To sort in different directions (ascending or descending)
- To sort objects from classes you cannot modify
- To provide context-specific sorting logic

### Syntax Rules and Structure

```java
// Custom comparator for sorting by a specific field
Comparator<Type> customOrder = (a, b) -> {
    // comparison logic
};
```

**Common Custom Ordering Patterns:**

| Pattern | Example |
|---------|---------|
| By field | `Comparator.comparing(Person::getName)` |
| By field (primitive) | `Comparator.comparingInt(Person::getAge)` |
| Reverse order | `comparator.reversed()` |
| Multiple fields | `comparator1.thenComparing(comparator2)` |
| Null-safe | `Comparator.nullsFirst(comparator)` |

### Annotated Code Example

```java
// File: CustomOrderingDemo.java
import java.util.*;

public class CustomOrderingDemo {
    static class Employee {
        private String name;
        private String department;
        private double salary;

        public Employee(String name, String department, double salary) {
            this.name = name;
            this.department = department;
            this.salary = salary;
        }

        public String getName() { return name; }
        public String getDepartment() { return department; }
        public double getSalary() { return salary; }

        @Override
        public String toString() {
            return name + " (" + department + ", $" + salary + ")";
        }
    }

    public static void main(String[] args) {
        List<Employee> employees = new ArrayList<>();
        employees.add(new Employee("Alice", "Engineering", 90000));
        employees.add(new Employee("Bob", "Sales", 70000));
        employees.add(new Employee("Charlie", "Engineering", 110000));
        employees.add(new Employee("Diana", "Sales", 80000));

        // Custom ordering: by department, then by salary descending
        Comparator<Employee> byDeptThenSalary = Comparator
            .comparing(Employee::getDepartment)
            .thenComparing(Comparator.comparingDouble(Employee::getSalary).reversed());

        Collections.sort(employees, byDeptThenSalary);

        for (Employee e : employees) {
            System.out.println(e);
        }
    }
}
```

**Expected Output:**
```
Charlie (Engineering, $110000.0)
Alice (Engineering, $90000.0)
Diana (Sales, $80000.0)
Bob (Sales, $70000.0)
```

**Step-by-Step Explanation:**

1. `Comparator.comparing(Employee::getDepartment)` sorts by department alphabetically.
2. `.thenComparing(...)` adds a secondary sort by salary.
3. `Comparator.comparingDouble(Employee::getSalary).reversed()` sorts salary descending.
4. The result: Engineering employees first (highest salary first), then Sales employees.

### Real-World Cases

- **E-commerce:** Sort by price (low to high), then by rating (high to low).
- **Social Media:** Sort posts by timestamp, then by likes.
- **Log Analysis:** Sort log entries by severity, then by timestamp.

---

## 7. Comparator Methods

### comparing()

**Definition:** Creates a comparator that compares objects by a extracted key.

**Syntax:**
```java
static <T, U extends Comparable<? super U>> Comparator<T> comparing(
    Function<? super T, ? extends U> keyExtractor)
```

**Example:**
```java
Comparator<Person> byName = Comparator.comparing(Person::getName);
```

**Purpose:** To create a comparator from a key extraction function, enabling concise sorting by a field.

### comparingInt() / comparingLong() / comparingDouble()

**Definition:** Creates a comparator for primitive keys, avoiding boxing overhead.

**Syntax:**
```java
static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor)
static <T> Comparator<T> comparingLong(ToLongFunction<? super T> keyExtractor)
static <T> Comparator<T> comparingDouble(ToDoubleFunction<? super T> keyExtractor)
```

**Example:**
```java
Comparator<Person> byAge = Comparator.comparingInt(Person::getAge);
```

**Purpose:** To sort by primitive fields without the performance cost of autoboxing.

### thenComparing()

**Definition:** Chains two comparators: the second is used only when the first considers the elements equal.

**Syntax:**
```java
default Comparator<T> thenComparing(Comparator<? super T> other)
default <U extends Comparable<? super U>> Comparator<T> thenComparing(
    Function<? super T, ? extends U> keyExtractor)
```

**Example:**
```java
Comparator<Person> byNameThenAge = Comparator
    .comparing(Person::getName)
    .thenComparingInt(Person::getAge);
```

**Purpose:** To create multi-level sorting criteria.

### reversed()

**Definition:** Returns a comparator that reverses the order of the original comparator.

**Syntax:**
```java
default Comparator<T> reversed()
```

**Example:**
```java
Comparator<Person> descendingByName = Comparator.comparing(Person::getName).reversed();
```

**Purpose:** To reverse any comparator's ordering.

### naturalOrder()

**Definition:** Returns a comparator that compares `Comparable` objects in natural order.

**Syntax:**
```java
static <T extends Comparable<? super T>> Comparator<T> naturalOrder()
```

**Example:**
```java
Comparator<String> natural = Comparator.naturalOrder();
```

**Purpose:** To obtain a comparator that uses natural ordering.

### reverseOrder()

**Definition:** Returns a comparator that imposes the reverse of natural ordering.

**Syntax:**
```java
static <T extends Comparable<? super T>> Comparator<T> reverseOrder()
```

**Example:**
```java
Comparator<String> reverse = Comparator.reverseOrder();
```

**Purpose:** To obtain a comparator that reverses natural ordering.

### nullsFirst() / nullsLast()

**Definition:** Returns a null-friendly comparator that considers `null` as less than (or greater than) non-null values.

**Syntax:**
```java
static <T> Comparator<T> nullsFirst(Comparator<? super T> comparator)
static <T> Comparator<T> nullsLast(Comparator<? super T> comparator)
```

**Example:**
```java
Comparator<String> nullSafe = Comparator.nullsFirst(Comparator.naturalOrder());
```

**Purpose:** To handle `null` values gracefully in sorting.

---

## 8. Sorting Collections

### Definitions

**Core Definition:** Sorting collections is the process of arranging the elements of a `List`, array, or other collection in a specific order, using either natural ordering (`Comparable`) or custom ordering (`Comparator`).

**Technical Definition:** Java provides several methods for sorting: `Collections.sort(List<T>)` uses natural ordering, `Collections.sort(List<T>, Comparator)` uses a custom comparator, `Arrays.sort(T[])` uses natural ordering, and `Arrays.sort(T[], Comparator)` uses a custom comparator. Since Java 8, `List.sort(Comparator)` is also available.

**Beginner-Friendly Explanation:** Sorting is like arranging a deck of cards in order. You can sort by number (natural ordering), by suit (custom ordering), or by any other rule you choose.

### Purposes

- To arrange data in a meaningful order
- To enable efficient searching (binary search requires sorted data)
- To improve the presentation of data
- To facilitate comparison and analysis

### Syntax Rules and Structure

```java
// Natural ordering
Collections.sort(list);
Arrays.sort(array);

// Custom ordering
Collections.sort(list, comparator);
Arrays.sort(array, comparator);
list.sort(comparator);  // Java 8+
```

**Syntax Rules:**

- `Collections.sort()` modifies the list in place.
- `Arrays.sort()` modifies the array in place.
- `List.sort()` is the preferred method since Java 8.
- The list must be mutable (not `List.of()`).

**Constraints and Limitations:**

- Natural ordering requires elements to implement `Comparable`.
- Custom ordering requires a `Comparator`.
- Sorting is O(n log n) for most implementations.

### Annotated Code Example

```java
// File: SortingDemo.java
import java.util.*;

public class SortingDemo {
    static class Person implements Comparable<Person> {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public int compareTo(Person other) {
            return this.name.compareTo(other.name);
        }

        public String getName() { return name; }
        public int getAge() { return age; }

        @Override
        public String toString() {
            return name + " (" + age + ")";
        }
    }

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("Charlie", 30));
        people.add(new Person("Alice", 25));
        people.add(new Person("Bob", 20));

        // Natural ordering (by name)
        Collections.sort(people);
        System.out.println("Natural: " + people);

        // Custom ordering (by age)
        people.sort(Comparator.comparingInt(Person::getAge));
        System.out.println("By age: " + people);

        // Custom ordering (by age descending)
        people.sort(Comparator.comparingInt(Person::getAge).reversed());
        System.out.println("By age desc: " + people);
    }
}
```

**Expected Output:**
```
Natural: [Alice (25), Bob (20), Charlie (30)]
By age: [Bob (20), Alice (25), Charlie (30)]
By age desc: [Charlie (30), Alice (25), Bob (20)]
```

**Step-by-Step Explanation:**

1. `Collections.sort(people)` uses natural ordering (by name).
2. `people.sort(Comparator.comparingInt(Person::getAge))` sorts by age ascending.
3. `.reversed()` reverses the age ordering to descending.

### Real-World Cases

- **E-commerce:** Sorting products by price, rating, or popularity.
- **Social Media:** Sorting posts by timestamp or likes.
- **File Systems:** Sorting files by name, size, or date.
- **Databases:** Sorting query results by various columns.

---

## 9. Sorting Maps

### Definitions

**Core Definition:** Sorting maps involves arranging the entries of a `Map` by key or value, using a `TreeMap` with a comparator or by converting to a list and sorting.

**Technical Definition:** `HashMap` and `LinkedHashMap` do not maintain sorted order. `TreeMap` maintains entries sorted by key using natural ordering or a `Comparator` provided at construction time. To sort a `HashMap` by value, you must convert its `entrySet()` to a `List` and sort it with a `Comparator`.

**Beginner-Friendly Explanation:** Maps are like dictionaries — they don't have a natural order. To sort a map, you either use a special map that keeps keys sorted (`TreeMap`) or convert the entries to a list and sort that list.

### Purposes

- To present map data in a sorted order
- To enable range queries on map keys
- To find the highest or lowest values in a map
- To sort configuration or reporting data

### Syntax Rules and Structure

```java
// Sort by key using TreeMap
Map<String, Integer> sortedByKey = new TreeMap<>(map);

// Sort by key using a custom comparator
Map<String, Integer> sortedByKeyDesc = new TreeMap<>(Comparator.reverseOrder());

// Sort by value using streams
List<Map.Entry<String, Integer>> entries = new ArrayList<>(map.entrySet());
entries.sort(Map.Entry.comparingByValue());
```

**Syntax Rules:**

- `TreeMap` sorts by key automatically.
- To sort by value, you must extract entries and sort them externally.
- `Map.Entry.comparingByKey()` and `comparingByValue()` provide convenient comparators.

**Constraints and Limitations:**

- `HashMap` cannot be sorted directly.
- `TreeMap` requires keys to be `Comparable` or a `Comparator` must be provided.
- Sorting by value requires extra memory for the list of entries.

### Annotated Code Example

```java
// File: SortingMapsDemo.java
import java.util.*;

public class SortingMapsDemo {
    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        scores.put("Diana", 88);

        // Sort by key using TreeMap
        Map<String, Integer> sortedByKey = new TreeMap<>(scores);
        System.out.println("By key: " + sortedByKey);

        // Sort by value using streams
        List<Map.Entry<String, Integer>> entries = new ArrayList<>(scores.entrySet());
        entries.sort(Map.Entry.comparingByValue());
        System.out.println("By value: " + entries);

        // Sort by value descending
        entries.sort(Map.Entry.comparingByValue(Comparator.reverseOrder()));
        System.out.println("By value desc: " + entries);
    }
}
```

**Expected Output:**
```
By key: {Alice=95, Bob=87, Charlie=92, Diana=88}
By value: [Bob=87, Diana=88, Charlie=92, Alice=95]
By value desc: [Alice=95, Charlie=92, Diana=88, Bob=87]
```

**Step-by-Step Explanation:**

1. `new TreeMap<>(scores)` creates a sorted map by key.
2. `new ArrayList<>(scores.entrySet())` converts entries to a list.
3. `Map.Entry.comparingByValue()` sorts by value ascending.
4. `.reversed()` sorts by value descending.

### Real-World Cases

- **Scoreboards:** Sorting players by score (value).
- **Inventory:** Sorting products by quantity (value).
- **Configuration:** Sorting properties alphabetically (key).
- **Leaderboards:** Sorting users by points (value).

---

## 10. Comparable vs Comparator

### Definitions

**Core Definition:** `Comparable` defines the natural ordering within a class, while `Comparator` defines custom orderings external to the class.

**Technical Definition:** "Comparable is implemented by the class itself to define its natural ordering. Comparator is a separate class or object that defines the order of objects". A class can implement `Comparable` only once, but you can create many `Comparator` implementations for different orderings.

**Beginner-Friendly Explanation:** `Comparable` is like a built-in sorting rule that objects carry with them. `Comparator` is like an external sorting tool you can create whenever you need a different rule.

### Comparison Table

| Feature | Comparable | Comparator |
|---------|-----------|------------|
| **Package** | `java.lang` | `java.util` |
| **Method** | `compareTo(T o)` | `compare(T o1, T o2)` |
| **Purpose** | Natural ordering | Custom ordering |
| **Implementation** | Class itself | Separate class/object |
| **Number of Orderings** | One per class | Unlimited |
| **Modifies Class** | Yes | No |
| **Consistent with equals** | Strongly recommended | Recommended |
| **Functional Interface** | No (but can be lambda) | Yes (since Java 8) |
| **Used By** | `Collections.sort(list)` | `Collections.sort(list, comparator)` |
| **TreeMap/TreeSet** | Default ordering | Specified at construction |

### When to Use Comparable

- When there is one obvious, natural ordering for the class
- When you are writing the class and can modify it
- When objects need to be sorted automatically by Java's built-in methods
- When using objects as keys in `TreeMap` or elements in `TreeSet`

### When to Use Comparator

- When you need multiple different orderings
- When you cannot modify the class (e.g., third-party library)
- When sorting logic is context-specific or temporary
- When you want to sort by different fields at different times

### Annotated Code Example

```java
// File: ComparableVsComparatorDemo.java
import java.util.*;

public class ComparableVsComparatorDemo {
    // Comparable: natural ordering by name
    static class Person implements Comparable<Person> {
        private String name;
        private int age;
        private double salary;

        public Person(String name, int age, double salary) {
            this.name = name;
            this.age = age;
            this.salary = salary;
        }

        @Override
        public int compareTo(Person other) {
            return this.name.compareTo(other.name);
        }

        public String getName() { return name; }
        public int getAge() { return age; }
        public double getSalary() { return salary; }

        @Override
        public String toString() {
            return name + " (" + age + ", $" + salary + ")";
        }
    }

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("Charlie", 30, 90000));
        people.add(new Person("Alice", 25, 110000));
        people.add(new Person("Bob", 20, 70000));

        // Natural ordering (Comparable)
        Collections.sort(people);
        System.out.println("Natural: " + people);

        // Custom ordering by age (Comparator)
        people.sort(Comparator.comparingInt(Person::getAge));
        System.out.println("By age: " + people);

        // Custom ordering by salary (Comparator)
        people.sort(Comparator.comparingDouble(Person::getSalary));
        System.out.println("By salary: " + people);
    }
}
```

**Expected Output:**
```
Natural: [Alice (25, $110000.0), Bob (20, $70000.0), Charlie (30, $90000.0)]
By age: [Bob (20, $70000.0), Alice (25, $110000.0), Charlie (30, $90000.0)]
By salary: [Bob (20, $70000.0), Charlie (30, $90000.0), Alice (25, $110000.0)]
```

**Step-by-Step Explanation:**

1. `Person` implements `Comparable` for natural ordering by name.
2. `Collections.sort(people)` uses natural ordering.
3. `Comparator.comparingInt(Person::getAge)` creates a custom ordering by age.
4. `Comparator.comparingDouble(Person::getSalary)` creates a custom ordering by salary.

### Real-World Cases

- **Comparable:** A `Product` class naturally sorts by product ID.
- **Comparator:** An e-commerce site sorts products by price, rating, or popularity — each is a custom ordering.

---

## 11. Best Practices

### Definitions

**Core Definition:** Best practices for `Comparable` and `Comparator` are guidelines for using comparison and ordering correctly and efficiently in Java applications.

**Technical Definition:** Best practices include using `Comparable` for the single natural ordering and `Comparator` for all other orderings, keeping `compareTo` simple, comparing only one field in `Comparable`, and using `Comparator` for additional sorting criteria. When sorting objects, it is better to provide a specific `Comparator` to `Collections.sort()` than to have the class implement `Comparable`, as it gives more flexibility and does not require implementing `hashCode()` and `equals()`.

**Beginner-Friendly Explanation:** Best practices help you choose the right interface for the job, write comparison logic correctly, and avoid common pitfalls.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Use `Comparable` for natural ordering** | Implement `Comparable` when there is one obvious, immutable natural ordering for the class. |
| **Use `Comparator` for multiple orderings** | Create `Comparator` implementations for context-specific sorting. |
| **Keep `compareTo` simple** | Compare only one field in `Comparable`; use `Comparator` for multi-field comparisons. |
| **Ensure consistency with `equals`** | `x.compareTo(y) == 0` should imply `x.equals(y) == true` when possible. |
| **Use `Integer.compare()` and `Double.compare()`** | Avoid subtraction-based comparisons (e.g., `a - b`) which can overflow. |
| **Use `Comparator` factory methods** | Use `Comparator.comparing()`, `thenComparing()`, etc. for concise and correct comparators. |
| **Handle `null` values** | Use `Comparator.nullsFirst()` or `nullsLast()` to handle `null` gracefully. |
| **Prefer `Comparator` over `Comparable` for sorting** | Provides more flexibility and does not require modifying the class. |
| **Test comparison logic thoroughly** | Verify antisymmetry, transitivity, and consistency with `equals`. |
| **Document the ordering** | Clearly document the natural ordering of a class. |

### Annotated Code Example

```java
// File: BestPracticesDemo.java
import java.util.*;

public class BestPracticesDemo {
    static class Employee implements Comparable<Employee> {
        private String name;
        private int age;
        private double salary;

        public Employee(String name, int age, double salary) {
            this.name = name;
            this.age = age;
            this.salary = salary;
        }

        // GOOD: Natural ordering by name (single field, simple)
        @Override
        public int compareTo(Employee other) {
            return this.name.compareTo(other.name);
        }

        public String getName() { return name; }
        public int getAge() { return age; }
        public double getSalary() { return salary; }

        @Override
        public String toString() {
            return name + " (" + age + ", $" + salary + ")";
        }
    }

    public static void main(String[] args) {
        List<Employee> employees = new ArrayList<>();
        employees.add(new Employee("Charlie", 30, 90000));
        employees.add(new Employee("Alice", 25, 110000));
        employees.add(new Employee("Bob", 20, 70000));

        // GOOD: Use Comparator for custom orderings
        employees.sort(Comparator.comparingInt(Employee::getAge));

        // GOOD: Use thenComparing for multi-field sorting
        employees.sort(Comparator
            .comparing(Employee::getName)
            .thenComparingDouble(Employee::getSalary));

        // GOOD: Use nullsFirst for null-safe sorting
        List<String> names = new ArrayList<>(Arrays.asList("Charlie", null, "Alice"));
        names.sort(Comparator.nullsFirst(Comparator.naturalOrder()));
        System.out.println(names); // [null, Alice, Charlie]

        // BAD: Subtraction-based comparison (overflow risk)
        // employees.sort((a, b) -> a.getAge() - b.getAge());

        // GOOD: Integer.compare for safe comparison
        employees.sort(Comparator.comparingInt(Employee::getAge));
    }
}
```

**Expected Output:**
```
[null, Alice, Charlie]
```

**Step-by-Step Explanation:**

1. `Employee` implements `Comparable` for natural ordering by name (single field).
2. `Comparator.comparingInt(Employee::getAge)` creates a custom ordering by age.
3. `Comparator.comparing(Employee::getName).thenComparingDouble(Employee::getSalary)` creates a multi-field comparator.
4. `Comparator.nullsFirst(Comparator.naturalOrder())` handles `null` values gracefully.
5. Subtraction-based comparison is avoided to prevent integer overflow.

### Real-World Cases

- **API Design:** Use `Comparator` factory methods to provide flexible sorting options.
- **Data Processing:** Use `Comparable` for natural ordering, `Comparator` for report-specific ordering.
- **Multi-Threading:** Ensure comparators are thread-safe (stateless).

---

## References Links

- Comparable (Java SE 21 & JDK 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Comparable.html
- Comparator (Java SE 21 & JDK 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Comparator.html
- Object Ordering — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/interfaces/order.html
- Comparable (OpenJDK Source) - https://cr.openjdk.org/~iris/se/21/latestSpec/api/java.base/java/lang/Comparable.html
- Introduction to the Comparator Interface — University of Maryland - http://www.cs.umd.edu/class/spring2025/cmsc132-010X-020X/labs/Week3/Comparator.pdf
- Sorting Collections — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/collections/algorithms/
- SEI CERT Oracle Coding Standard — MET10-J: Follow the general contract when implementing the compareTo() method - https://wiki.sei.cmu.edu/confluence/display/java/MET10-J.+Follow+the+general+contract+when+implementing+the+compareTo%28%29+method
- Comparable vs Comparator in Java — Baeldung - https://www.baeldung.com/java-comparable-comparator
- Java 8 Comparator: How to Sort a List — DZone - https://dzone.com/articles/java-8-comparator-how-to-sort-a-list
- Autocomplete Assignment — Princeton University - https://www.cs.princeton.edu/courses/archive/spring26/cos226/assignments/autocomplete/specification.php