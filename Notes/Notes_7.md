# Copy/Move Elision
**2017-09-26**

```C++
Vector makeAVector() {
    return Vector{} //   // Basic constructor
}

Vector v = makeAVector();   // move ctor? copy ctor?
```

Try in g++, just the basic constructor, not copy/move

In some circumstances, the compiler is allowed to skip calling the copy/move constructors (but doesn't have to). `makeAVector()` writes its result directly into the space occupied by `v`, rather than copy/move it later.

Ex.
```C++
Vector v = Vector{};    // Formally a basic construction and a copy/move construction
                        // Vector{} is a basic constructor
                        // Here though, the compiler is *required* to elide the copy/move
                        // So basic constructor here only 

Vector v = Vector{Vector{Vector{}}};    // Still one basic ctor only
```

Ex.
``` C++
void doSomething(Vector v) {...};   // Pass-by-value - copy/move ctor

doSomething(makeAVector());   
```

Result of makeAVector written directly into the param, there is no copy/move

This is allowed, even if dropping ctor calls would change the behaviour of the program (ex. if the constructors print something).

If you really need all of the constructors to run:

`g++14 -fno_elide_constructors ...`

**Note:** while possible, can slow down your program considerably

- Copying is an expensive procedure, the compiler skips these constructors as an optimization

In summary: Rule of 5 (Big 5)

- If you need to customize any one of
1. Copy constructor
1. Copy assignment
1. Destructor
1. Move constructor
1. Move assignment

then you usually need to customize all 5.

## Problem 6: I want a constant vector

Say we want to print a vector:

```C++
ostream &operator <<(ostream &out, const Vector &v) {
    for (size_t i = 0; i < v.size(), ++i) {
        out << v.itemAt(i) << " ";
    }
}
```

WON'T COMPILE!!! (lushman pls)

- Can't call `size()` and `itemAt()` on a const object. What if these methods change fields?
- Since they don't, declare them as `const`

```C++
struct Vector {
    ...
    size_t size() const;    // Means these methods will not modify fields
    int &itemAt(size_t i) const;    // Can be called on const objects
    ...
};

size_t Vector::size() const {return n};
int &Vector:itemAt(size_t i) const {return theVector[i];}
```

Now the loop will work.

BUT:

```C++
void f(const Vector &v) {
    v.itemAt(0) = 4;    // Works!! v not very const...
}
```

- `v` is a const object - cannot change `n`, `cap`, `theVector` (ptr)
- You can changed items pointed to by theVector

Can we fix this?

```C++
struct Vector {
    ...
    const int &itemAt(size_t i) const;
};

const int &itemAt(size_t i) const {
    return theVector[i];
}
```

Now `v.itemAt(0) = 4` won't compile if `v` is const, but it also won't compile if `v` is not const

To fix: **const overloading**

```C++
struct Vector {
    ...
    const int &itemAt(size_t i) const;  // Will be called if the object is const
    int &itemAt(size_t i);  // Will be called if object is non-const
};

inline const int &Vector::itemAt(size_t i) const {return theVector[i];}
inline int &vector::itemAt(size_t) {return theVector[i]};
```

Putting in `inline` tells the compile to replace the function call with the function body to save the cost of having to call a function

Merely a suggestion, compiler can choose to ignore it if it sees fit. Good idea for small functions

So now `v.itemAt(0) = 4;` will only compile if and only if `v` is non-const

Now let's make it prettier:

```C++
struct Vector {
    size_t size() const {return n;} // Method body inside class implcity declares the method inline
    const int &operator[](size_t i) const {return theVector[i]};
    int &operator[](size_t i) {return theVector[i];}    
};

ostream &operator<<(ostream &out, const vector &v) {
    for (size_t i = 0; i < v.size(); ++i) {
        out << v[i] << " ";
    }

    return out;
}
```

## Problem 7: Tampering

```C++
Vector v;
v.cap = 100;    // Sets cap without allocating memory
v.push_back(1);
v.push_back(10);
...  // Undefined behaviour - will likely crash
```
Interfering with ADTs (Abstract Data Types)

1. Forgery 
    - Creating an object without a constructor function
        - Not possible once we wrote constructors
1. Tampering
    - Accessing the internals without using provided interface functions

What's the big deal? _Invariants_

- Statement that will always be true about an abstraction

ADT's provide and rely on invariants

- **Stacks**
    - Provide the invariant that the last item pushed is the first item popped
- **Vectors**
    - Rely on the invariant that elements `O`, ..., `cap-1` denote valid locations

Cannot gaurantee invariants if the user can interfere, makes the program hard to reason behind

**Fix:** _encapsulation_, seal objects into "black boxes"

```C++
struct Vector {
    private:    // Fields are only accessible within the Vector class
        size_t n, cap;      
        int *theVector;
    public:     // Visible to all
        Vector();
        size_t size() const;
        void push_back(int n);
        ..
};
```

If no access specifer is given: default is public

In a previous lecture:

_vector.cc_
```C++
#include "vector.h"
namespace {
    void increaseCap(vector &v) {...}   // Doesn't work anymore! Doesn't have access to v's internals
}
```

Try again:

_vector.h_
```C++
struct Vector {
    private:
        size_t ...
        ...
    public:
        Vector();
        ...

    private:
        void increaseCap(); // Now a private method
};
```

NOW
_vector.cc_
```C++
namespace CS246E {
    Vector::Vector() {...}  
    // etc. as before
    void Vector::increaseCap() {...}  // Don't need anonymous namespaces anymore!
};
```

- **Structs** (`struct`)
    - Default public access
    - Would be better if we had default private
- **Classes** (`class`)
    - Exactly like struct, except private default access

_vector.cc_
```C++
class Vector {
        size_t n, cap;
        int *theVector;
    public:
        Vector();
        ...

    private:
        void increaseCap(); // Now a private method
};
```