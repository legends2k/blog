+++
title = "C++ Primer"
description = "solving “you don’t know what you don’t know”"
date = "2018-09-06T17:00:46-07:00"
tags = ["tech", "language", "c++"]
toc = true
+++

> Within C++, there is a much smaller and cleaner language struggling to get out. — Bjarne Stroustrup

C++ is vast.  If you’re beginning to learn it, the internet has enough discouraging words.  Just start learning instead of reading about it i.e. don’t learn _about_ C++, learn C++.  The innumerable topics might be daunting, but don’t worry.  Knowing a few concepts will get you productive.  More than the actual content, this article aims to give you keywords of _that_ set, so that you can pursue them further.

_[Spirit of C++][]_ is a beginner presentation I’d given to a 50+ team; it [received good comments][sprit-c++-comments] too.  See that for a tutorial-style, more in-depth treatment of the basics.

[Spirit of C++]: https://legends2k.github.io/spirit-of-cpp
[sprit-c++-comments]: https://www.reddit.com/r/cpp/comments/da4xrd/spirit_of_c/

# Overview

- Based on C
    - More popular sibling of Objective-C
    - Intermediate-level: can operate at both low and high-levels
    - Native: compiles to object, not byte, code
    - Rich standard library
    - Fairly modern with new standards: C++11, 14, 17 and 20
- Static, strongly typed
    - **Static** Variables are tied to types
    - **Strong** Type system is rigid and *tries* to prevent
      cross-overs
    - Calling on nil crashes; can’t call a non-existent method
    - A set of type safety properties for all possible inputs
      guaranteed
- Key Differences to C\#, Objective-C and Java
    - Multi-paradigm language
        - Imperative, OOP, generic and functional
    - Allows static and virtual methods
    - Supports multiple inheritance
    - No Reflection
        - Queries about type system at run-time unsupported
        - NO type system once compiled

Lots of dark corners not discussed; we want to build a good mental model
of a language; corner cases build a weak model.

# Structure

- Comment: `//`, `/**/`
- Token: `int`, `a`
- Expression: `a + b`
    - Operators: `*`, `static_cast<T>()`, `sizeof`, `new`, …
- Statement: `c = a + b;`
- Block: `{ c = a + b; }`
- Function: `int add(int a, int b) { return a + b; }`
- Function template `template <typename T> T add(T a, T b) { return a + b; }`
- Class
- Class template
- Inclusions (header *and* source)

**Bonus**: Inherits C’s header issues.

# Data Types

## Built-in Types

- **Nullity** `void`
- **Integral**
    - `bool`
    - `char`
        - `wchar_t`
        - `char16_t`
        - `char32_t`
    - `int`, `unsigned int`
        - `short`, `unsigned short`
        - `long`, `unsigned long`
        - `long long`, `unsigned long long`
        - `uint8_t`, `int8_t`, … from `cstdint`
- **Floating Point**
    - `float`
    - `double`
        - `long double`
- **Arrays** `int[3]`, `float[5]`
- **Function types** `int (float, char)`
- **Pointers** `int*`, `float*`, `int (*addptr)(int, int)` …
- **References**
- **Class and struct types**

None have guaranteed sizes except size-named ones.

# Sample 1

{{< highlight cpp >}}
#include <iostream>

// function definition
// pass by value
int add(int x, int y) {
  return x + y;
}

// pass by reference; can also be achieved with pointers
void myswap(int &x, int &y) {
  int t = x;
  x = y;
  y = t;
}

// function declaration
int add(int x, int, int /*z*/);

int main() {
  int a = 1, b = 2;
  myswap(a, b);
  float f = add(a, b);    // widening assignment, cast unneeded
  // really a call to std::ostream& std::ios::operator<<(std::ostream&, float)
  // with std::cout and f as parameters; notice the return type that enables expression chaining
  std::cout << f << '\n';
  const double PI = 3.14;
  // clang++ 9.1.0
  // warning: implicit conversion from 'double' to 'int' changes value from 3.14 to 3
  std::cout << add(1, 2, PI);
  // static_cast<int>(3.14) states programmer intention explicitly; no warning
}

// pass by value
int add(int x, int y, int z) {
  return x + y + z;
}
{{< /highlight >}}

## Notice

- Inclusion
- Variables, types
- Function definition
- Function declaration
- Function call
- Function overloading
- Standard functions

## Compilation

Compile with GCC/Clang’s C++ frontend

{{< highlight bash >}}
g++ -Wall -std=c++14 -pedantic test.cpp -o test
{{< /highlight >}}

Add `-g -O0` for debug symbols.

Compile with VC++ (use `/D_DEBUG` for debug symbols)

{{< highlight text >}}
cl /EHsc test.cpp
{{< /highlight >}}

# Conditionals and Loops

- If… else
- For
    - Range for
- While
    - Do.. while
- Switch
- Go to

Loops additionally support `break` and `continue`.

## Sample 2

{{< highlight cpp >}}
#include <iostream>
#include <string>

int main() {
  int i = 0;
  int arr[10] = { };
  do {
    arr[i] = i;
    ++i;
  } while (i < 10);
  // idomatic way of doing this: std::iota
  std::cout << i << '\n';

  // boolean context
  if (!i)  // checks if i is 0
    std::cout << "What!\n";
  else if (i == 10)
    std::cout << "Cool\n";
  else
    std::cout << "Whatever\n";

  goto vowels;

  for (; i > 0; --i) {
    std::cout << i << '\t';
  }

vowels:
  std::string s = "\nHello";
  // range for
  for (char c : s) {
    // switch only operates on integral types
    switch (c) {
      case 'a':
        std::cout << "o\n";
        break;
      case 'e':
        std::cout << "o\n";
        break;
      case 'i':
        std::cout << "o\n";
        break;
      case 'o':
        std::cout << "o\n";
        break;
      case 'u':
        std::cout << "o\n";
        break;
      default:
        std::cout << "x\n";
    }
  }
}
{{< /highlight >}}

# Scope, Storage Duration and Linkage

- **Scope**: Region in which a name is visible to compiler
    - Namespaces are used for better organising
- **Linkage**: Name visibility for linker
- [Variable Scopes](https://stackoverflow.com/a/13415659/183120)
    - **Local**: Visible only in the current block
    - **Global**: Visible to the compilation unit
- Storage Duration
    - **Automatic / Local**: Lifetime ends as execution leaves scope
    - **Static**: Alive throughout the program
- [Linkage](https://stackoverflow.com/q/1358400)
    - **None**: No linkage
    - **Internal**: Translation unit
    - **External**: Beyond compilation unit
    - Global variables are by default internal; change with `extern`
    - Function are by default external; change with `static`

# Object Construction and Memory Management

- **Two types**:
  [POD](https://stackoverflow.com/questions/146452/what-are-pod-types-in-c)s (no magic) and non-PODs (magic)
- **Construction**: memory allocation + initialization
- **Destruction**: deinitialization + memory deallocation
- Memory spaces
    - Stack
        - A stack frame / function (memory) of limited size
        - Hosts function parameters and local variables
        - Frame popped out when function returns to caller or an exception occurs
        - Current frame and the ones above can *access* data
        - Fast and short-lived; **preferred**
    - Free store (heap)
        - A large area free to allocate memory
        - Never deleted until done explicitly
        - Greater visibility and flexibility in life time
        - Greater chances of fragmentation
        - Manual management; cumbersome
- No garbage collector; mostly manual
    - Writing [custom memory allocators are not uncommon](https://github.com/mtrebi/memory-allocators)
    - Overloading `new` isn’t uncommon but highly discouraged
- Modern C++ with RAII relieves one from *most* manual rote work (sometimes called “eagerly managed”)
- Avoid `new` and `delete` like the plague!
- Use `std::unique_ptr` and sparingly `std::shared_ptr`
- Use containers or the many higher-level constructs available in `std`

## Sample 3

{{< highlight cpp >}}
#include <iostream>
#include <memory>

int main() {
  int *intPtr = new int{0};  // initialize value to 0
  delete intPtr;             // call dtor and release memory
  intPtr = nullptr;          // initialize pointer to 0

  // array has its own operators
  int *arrPtr = new int[4]{0, 1, 2, 3};  // <- initializer list
  arrPtr[0] = 1;
  delete [] arrPtr;
  arrPtr = nullptr;

  std::unique_ptr<int> ip = std::make_unique<int>(3);
  std::unique_ptr<int[]> ap = std::make_unique<int[]>(3);
  std::cout << *ip << '\n' << ap[2];
}
{{< /highlight >}}

## Sample 4

Shows the difference between traditional and modern C++ dialects.

{{< highlight cpp >}}
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <iterator>
#include <stdexcept>
#include <limits>

class mySafeInt
{
public:
    mySafeInt(int x) : i{x} { }

    int get() const { return i; }

private:
    int i = 0;
};

mySafeInt** conv_old(int argc, char **argv, int* count) {
    mySafeInt **p = nullptr;
    int processed = 0;
    if (argc >= 2) {
        p = new mySafeInt*[argc];
        for (int i = 1; i < argc; ++i) {
            int x = strtoul(argv[i], nullptr, 10);
            if (x != 0)
                p[processed++] = new mySafeInt{x};
        }
    }
    *count = processed;
    return p;
}

std::vector<mySafeInt> conv_new(int argc, char **argv) {
    std::vector<std::string> vs(argv + 1, argv + argc);
    std::vector<mySafeInt> v;
    std::for_each(vs.cbegin(), vs.cend(), [&](const std::string& s){
                                              try {
                                                  v.emplace_back(mySafeInt{std::stoi(s, nullptr, 10)});
                                              }
                                              catch (std::invalid_argument&) {
                                                  std::cerr << "Invalid argument '" << s << "' ignored.\n";
                                              }
                                              catch (std::out_of_range&) {
                                                  std::cerr << s << " beyond MAX_INT(" << std::numeric_limits<int>::max() << "); ignored.\n";
                                              }
                                          });
    return v;
}

std::ostream& operator<< (std::ostream& os, const mySafeInt &i) {
    return os << i.get();
}

int main(int argc, char **argv) {
    int outLen = 0;
    mySafeInt** ip = conv_old(argc, argv, &outLen);
    if ((ip == nullptr) || (outLen == 0))
         std::cerr << "Insufficient data\n";
    else {
        for (int i = 0; i < outLen; ++i)
            std::cout << ip[i]->get() << ", ";

        for (int i = 0; i < outLen; ++i) {
            delete ip[i];
        }
        delete [] ip;
    }
    std::cout << '\n';

    auto v = conv_new(argc, argv);
    if (v.empty())
        std::cerr << "Insufficient data\n";
    else {
        std::copy(v.cbegin(), v.cend(), std::ostream_iterator<mySafeInt>{std::cout, ", "});
    }
    std::cout << '\n';
}
{{< /highlight >}}

# Structs and Classes

- User-defined types built atop primitives
- Class and object-level data members and member functions
    - Beware of [static initialization order fiasco](https://isocpp.org/wiki/faq/ctors#static-init-order)!
- Composition and inheritance; [prefer former over latter](https://blog.codinghorror.com/inherits-nothing/)

> [Inheritance isn’t for code reuse](https://isocpp.org/wiki/faq/objective-c#objective-c-and-inherit); it’s for interface conformance.

- Allows fine-grained control over
    - Allocation
    - Initialization
    - Copy
    - Move
    - Assignment (both copy and move)
    - Deinitialization
    - Deallocation
    - Operators
- Popularly called **Rule of Three**, then **Rule of Five** but now really **[Rule of Zero](https://en.cppreference.com/w/cpp/language/rule_of_three)**
- Refer [cheat sheet for when to supply which special member function](https://foonathan.net/2019/02/special-member-functions/)
- Order is important
    - Construction happens from top to bottom e.g. inheritance,
      variables of a class, function, etc.
    - Destruction happens from bottom to top

## Sample 5

{{< highlight cpp >}}
#include <memory>
#include <iostream>

// HEADER: shape.hpp

struct Point
{
  float x = 0, y = 0;
};

struct Shape2D
{
  // data members
  Point pos;

  // member functions

  // constructors
  Shape2D() = default;
  Shape2D(float posX, float posY);

  // accessors and modifiers; former is usually const
  Point GetPos() const;
  void SetPos(int posX, int posY);

  // class data and functions
  static constexpr float PI = 3.14f;
  static int GetResID();
};

// IMPLEMENTATION: shape.cpp

Shape2D::Shape2D(float posX, float posY) : pos{posX, posY} {
}

void Shape2D::SetPos(int posX, int posY) {
  pos.x = posX;
  pos.y = posY;
}

Point Shape2D::GetPos() const {
  return pos;
}

// can’t access members from a static function
// call as Shape2D::GetResID() or obj.GetResID()
int Shape2D::GetResID() {
  return 0x12;
}

// HEADER: circle.hpp

class Circle : public Shape2D
{
public:

  Circle();
  Circle(float x, float y, float rad);
  Circle(Point pt, float rad);
  Circle(const Circle& that);

  Circle& operator=(const Circle& that);
  float operator+(const Circle& that) const;

  ~Circle() = default;

private:
  float r = 1.0f;            // in-place initilizers
};

// IMPLEMNTATION: circle.cpp

Circle::Circle() : r{1.0f} {
}

// initialization lists
Circle::Circle(float x, float y, float rad) : Shape2D(x, y), r{rad} {
}

Circle::Circle(Point pt, float rad) : Shape2D(pt.x, pt.y) {
  r = rad;        // perhaps slower way
}

// copy constructor; take by reference
Circle::Circle(const Circle& that) : Shape2D{that.pos.x, that.pos.y}, r{that.r} {
}

float Circle::operator+(const Circle& that) const {
  return r + that.r;
}

Circle& Circle::operator=(const Circle& that) {
  if (this != &that) {
    this->r = that.r;
    // base class method call syntax is the same as static function call interface
    Shape2D::operator=(that);
  }
  // return this for expression chaining
  return *this;
}

int main() {
  Circle c1;
  auto c2 = std::make_unique<Circle>(1, 1, 6);
  std::cout << c1 + *c2 << '\n';
}
{{< /highlight >}}

# Interfaces or ABCs

- Abstract base classes are interfaces in C++
- Concrete classes implement ABCs
- Overridable methods are marked `virtual`
- Must-override methods are made pure virtual `= 0`
- Polymorphism with dynamic binding
    - Works only with pointers and references
    - Polymorphism: `animal.eat()` might be `squirrel.nibble()` or
      `cow.chew()` depending on the object at runtime
    - [Dynamic Binding](https://isocpp.org/wiki/faq/virtual-functions#dyn-binding): the mechanism that enables polymorphism
        - Call to actual function resolved based on `vtbl` (virtual table)
    - Never call a virtual function during construction!

## Sample 6

{{< highlight cpp >}}
class Shape  // abstract
{

public:
  virtual ~Shape() { }   // <- without this it’s not an interface!!

  virtual Point GetPosition() const = 0;  // pure virtual
  virtual void  SetPosition(float x, float y) = 0;
};

class Square : public Shape
{
public:
  Point GetPosition() const override {
  }
};
{{< /highlight >}}

# Asserts, Errors and Exceptions

- Asserts are validation for programmer whether internal systems work
  correctly
    - e.g. assert if an internal invariant has remained unchanged
- Errors are when at runtime something goes wrong
    - e.g. user input is invalid, file system is not longer available
    - Fail fast / early
- [Assertions: planned, errors: unplanned](https://stackoverflow.com/a/21084587/183120)
- Use static asserts for compile-time validation/conformance
- C/COM-style: return error code
- C++ allows both error codes and exceptions (`try { throw(...); } catch(...) { }`)
- Internal errors, if you’re throwing an exception, be prepared to
  catch it too
- External errors that are manageable catch else throw
    - Uncaught exceptions bubble up and abort the program when the
      stack is empty
- Don’t use exceptions as a control mechanism
- When a [constructor fails throw an exception](https://isocpp.org/wiki/faq/exceptions#ctors-can-throw);
  notice that the return type is missing
- When a destructor fails? [Call Aunt Tilda](https://isocpp.org/wiki/faq/exceptions#dtors-shouldnt-throw)
    - Seriously, never throw an exception from a destructor
    - Disrupts stack unwind sequence and aborts the program
- Some projects turn off exceptions for various reasons including
    - Legacy _C_ code interfacing that doesn’t have exceptions
    - Performance (common in [game engines](https://stackoverflow.com/a/943169/183120))


# Commonly used `std::` facilities

C++ ships with a very versatile standard library. Take advantage of it!
Explore now; find gems!

## [Containers](https://en.cppreference.com/w/cpp/container)

### Sequences

- `std::vector` all-purpose, auto-expanding array type when size unknown at compile-time
- `std::array` when you know the size; wrapper over simple arrays; avoids ugliness
- `std::deque` double-ended queue that’s almost as fast as vector; Herb Sutter recommends highly
- `std::list`, `std::forward_list` doubly and singly linked list

### Associative

- `std::set` collection of unique items; sorted and searchable in O(log<sub>2</sub> N) time[^1]
- `std::map` collection of key-value pairs; same as `set` with a value associated to the key
- `std::multiset`, `std::multimap` multiple keys allowed; not just unique

### Unordered Associative

- `std::unordered_set` hash map with no values
- `std::unordered_map` hash map[^2]
- `std::unordered_multiset`, `std::unordered_multimap`

### Container Adopters

- `std::stack`
- `std::queue`
- `std::priority_queue`

## [Algorithms](https://en.cppreference.com/w/cpp/algorithm)

- `std::for_each`
- `std::copy`
- `std::lower_bound`, `std::upper_bound` (binary search on sorted sequences)
- `std::transform`
- `std::find_if`
- `std::any_of`, `std::all_of`, `std::none_of`

## [Concurrency](https://en.cppreference.com/w/cpp/thread)

- `std::thread`
- `std::packaged_task`
- `std::async`
- `std::future`, `std::promise`
- `std::mutex`
- `std::lock_guard`

Atomics, filesystem library and many more, not covered here due to
scope.

# Further Reading

1. [ISO C++ FAQ](https://isocpp.org/wiki/faq)
2. [C++ and standard library reference](https://en.cppreference.com)
3. [StackOverflow C++ FAQ](https://stackoverflow.com/tags/c++-faq)
4. C++ Primer, 5th Edition
5. The C++ Programming Language, Bjarne Stroustrup
6. [RIP Tutorial](https://riptutorial.com/cplusplus) includes eccentric
   topics like alignment, ADL, metaprogramming, etc.
7. [The Definitive C++ Book Guide and List](https://stackoverflow.com/questions/388242/the-definitive-c-book-guide-and-list)
8. [C++ Idioms](https://en.wikibooks.org/wiki/More_C%252B%252B_Idioms)
9. [C++ Papyrus](https://caiorss.github.io/C-Cpp-Notes/): extensive notes on various C and C++-related topics including tool chains, shared libraries, Unix, Linux and WinAPI programming, OpenGL, etc.

[Swiss Tables]: https://abseil.io/blog/20180927-swisstables
[Skarupe]: https://github.com/skarupke/flat_hash_map
[skarupe-blog]: https://probablydance.com/2018/05/28/a-new-fast-hash-table-in-response-to-googles-new-fast-hash-table/
[skarupe-recommends]: https://youtu.be/M2fKMP47slQ?t=3915


[^1]: [Use sorted vector instead of set](http://lafstern.org/matt/col1.pdf), if `find` calls outnumber `insert`; it’d be lot faster — Matt Austern.

[^2]: [Robin Hood hashing](https://www.sebastiansylvan.com/post/robin-hood-hashing-should-be-your-default-hash-table-implementation/) is a good technique when using linear probing.  `absl::flat_hash_map`, part of Abseil’s [Swiss tables][], is recommended over `std::unordered_map`.  It is also Rust’s `HashMap` implementation.  [Malte Skarupe’s `bytell_hash_map`][Skarupe] is the [fastest][skarupe-blog] as of writing this article.  Check [Malte’s recommendations][skarupe-recommends] on choosing one.
