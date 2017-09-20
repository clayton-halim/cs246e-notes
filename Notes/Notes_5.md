# Member Initialization List (cont.)
**2017-09-20**

MIL _must_ be used for fields that are 

- Constants
- References
- Objects

In general, it should be used as much as possible.

Careful: single argument constructors
```C++
struct Node {
    Node(int data, Node *next = nullptr): ... {}
}
```

- Single argument constructors create implicit constructors

```C++
Node n {4};     // OK
Node n = 4;     // OK - implicity converted from int to Node

void f(Node n);
f(4); // OK - maybe trouble
```

However you can add an `explicit` keyword to disable the implicit conversion

```C++
explicit struct Node {
    Node(int data, Node *next = nullptr): ... {}
}

Node n {4};  // OK
Node n = 4;  // BAD

f(4) // BAD
f(Node {4}) // OK
```

## Object Destructor

A method called the **destructor** (dtor) runs automatically

- Built-in dtor: calls dtor on all fields that are objects
- Object destruction protocol:
    1. Dtor body runs
    2. Fields destructed (dtors called on fields that are objs) in reverse declaration order
    3. (Later)
    4. Space deallocated

```C++
struct Node {
    int data;
    Node *next;
};
```

In this case the built-in destructor does nothing because neither field is an object

If we have:

```C++
Node *n = new Node {3, new Node {4, new Node {5, nullptr}}}
delete n;  // Only deletes the first node (memory leak!)
```

We can fix this by writing our own destructor:

```C++
struct Node {
    ...

    ~Node() {
        delete next;
    }
};

delete n;  // Now frees the whole list
```

Also:

```C++
{
    Node n {1, new Node {2, new Node {3, nullptr}}};
}  // Scope of n ends; whole list is freed
```

Objects:

- A constructor always runs when they are created
- A destructor always runs when they are destroyed

_vector.h_

```C++
#ifndef VECTOR_H
#define VECTOR_H

namespace CS246E {
    struct Vector {
        size_t n, cap;
        int *theVector;

        Vector();
        size_t size();
        int &itemAt(size_t i);
        void push_back();
        void pop_back();
        ~Vector();
    };
}
#endif
```

_vector.cc_

```C++
#include "vector.h"

namespace {
    void increaseCap(Vector &v) {
        ...
    }
}

const size_t startSize = 1;

CS246E::Vector::Vector(): 
    n{0}, cap{startSize}, theVector{new int[cap]} {
}

size_t CS246E::Vector::size() {
    return n;
}

// Etc.

CS246E::Vector::~Vector() {
    delete[] theVector;
}
```

_main.cc_

```C++
int main() {
    Vector v;   // Constructor is already called - no make_vector
    v.push_back(1);
    v.push_back(10);
    v.push_back(100);
    v.itemAt(0) = 2; 
}   // No dispose - destructor cleans v up
```

# Problem 4: Copies

```C++
Vector v;

v.pushback(100);
...
Vector w = v;  // Allowed - constructs w as a copy of v
w.itemAt(0);  // 100
v.itemAt(0); = 200;
w.itemAt(0);  // 200 - **shallow copy**, v and w share data 
```

For `Vector w = v;`

- Constructs `w` as a copy of `v`
- Invokes the **copy constructor**

```C++
struct Vector {
    Vector(const Vector &other) {...}  // Copy constructor
    // Compiler supplied copy-ctor, copies all fields, shallow copy
};
```

If you want a deep copy, write your own copy constructor

```C++
struct Node {  // Vector: exercise
    int data;
    Node *next;
    ...
    Node (const Node &other): 
        data{other.data}, next{other.next ? new Node{*other.next} : nullptr} {  // Account for dereferencing a nullptr
            ...
        }
};
```
