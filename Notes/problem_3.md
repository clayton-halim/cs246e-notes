[Linear Collections and Modularity <<](./problem_2.md) | [**Home**](../README.md) | [>> Copies](./problem_4.md) 

# Problem 3: Linear Collections and Memory Management
**2017-09-14**  
**Readings:** 7.7.1, 14, 16.2 

**Arrays**
`int a[10];`

- On the stack, fixed size

On the heap:
`int *p = new int[10];`

To delete:
`delete[] p;`

Use `new` with `delete`, and `new [...]` with `delete[]`

Mismatching these is undefined behaviour

**Problem:** what if our array isn't big enough
Note: no `realloc` for `new`/`delete`  

Use abstraction to solve the problem:

_vector.h_

```C++
#ifndef VECTOR_H
#define VECTOR_H

namespace CS246E {
    struct vector {
        size_t size, cap;
        int *theVector;
    }
};

const size_t startsize = 1;

vector make_vector();

size_t size(const vector &v);

int &itemAt(const vector &v, size_t i);

void push_back(const vector &v, int x);

void pop_back(const vector&v);

void dispose(vector &v);

#endif
```

_vector.cc_

```C++
#include "vector.h"

namespace {  // Anonymous namespace makes the function only visible to file (same as static in C)
    void increaseCap(CS246E::vector &v) {
        if (v.size == v.cap) {
            int *newVec = new int[2 * v.cap];

            for (size_t i = 0; i < v.cap; ++i) {
                newVec[i] = v.theVector[i]
            }

            delete[] v.theVector;
            v.theVector = newVec;
            v.cap *= 2;
        }
    }
}

CS246E::vector CS246E::make_vector() {
    vector v {0, startSize, new int[startSize]};
    return v;
}

size_t CS246E::size(const vector &v) {
    return v.size;
}

int &CS246E::itemAt(const vector &v, size_t i) {
    return v.theVector[i];
}

void CS246E::push_back(vector &v, int n) {
    increaseCap(v);
    v.theVector[v.size++] = n;
}

void CS246E::pop_back(vector &v) {
    if (v.size > 0) {
        --v.size;
    }
}

void CS246E::dispose(vector &v) {
    delete[] v.theVector;
}
```

_main.cc_

```C++
#include "vector.h"

using CS246E::vector;  // only allows you to use vector without CS246E::

int main() {
    vector v = CS246E::make_vector();
    push_back(v, 1);
    push_back(v, 10);
    push_back(v, 100);  // Can add as many items as we want without worrying about allocation
    itemAt(v, 0) = 2;
    dispose(v);
}
```

**Question:** why don't we have to say `CS246E::push_back`, `CS246E::itemAt`, `CS246E::dispose?`

**Answer:** Argument-Dependent Lookup (ADL) - aka König lookup

- If the type of a function f's argument belongs to a namespace n, then C++ will search the namespace n,
as well as the current scope, for a function matching f

This is the reason why we can say
```C++
std::cout << x
// rather than
std::operator<< (std::cout, x)
```

- **Problems** 
    - What if we forget to call `make_vector`? (uninitialized object)
    - What if we forget to call `dispose`? (memory leak)
- How can we make this more robust?

## Introduction to Classes
First concept in OOP - functions inside structs

```C++
struct Student {
    int assns, mt, final;
    
    float grade() {
        return assns*0.4 + mt*0.2 + final*0.4;
    }
};
```

- Structs that can contain functions - **classes**
- Functions inside structs - **methods**
- Instances of a class - **objects**

```C++
Student bob {90, 70, 80};
cout << bob.grade();
```

`bob` is an object, `.grade()` is a method.

What do `assns`, `mt`, `final`, mean with `grade() {...}`?

- Fields of the _current_ object, the receiver of the method call (ie. `bob`)

Formally, methods differ form functions in that methods have an implicit parameter called `this`, that is 
a pointer to the receiver object.

`this == &bob`

Could hae written (equivalent):

```C++
struct Student {
    ...

    float grade() {
        return this->assns * 0.4
                + this->mt * 0.2
                + this->final * 0.4;
    }
};
```

## Initializing objects
```C++
Student bob {90, 70, 80}
```

- C style struct initialization
- Field by field
- ok, but limited

Better initializtion method: a constructor

```C++
struct Student {
    int assns, mt, final;

    Student(int assns, int mt, int final) {
        this->assns = assns;
        this->mt = mt;
        this->final = final;
    }
};

// Now we can call
Student bob {90, 70, 80};
// Now calls the constructor with args 90, 70, 80
```

**Note:** once the constructor is defined, the C style field-by-field initialization is no longer available

**Equiv:** 
```C++ 
Student bob = Student{90, 70, 80};
```

**Heap:**
```C++
Student *p = new Student{90, 80, 70};
delete p;
```
- **Advantages of constructors:**
    - Default parameters
    - Overloading
    - Sanity checks

ex.

```C++
struct Student {
    student(int assns = 0, int mt = 0, int final = 0) {
        this->assns = assns;
        ...
    }
};

Student laura {70};  // 70, 0 , 0
Student newKid; // 0, 0, 0 
```

**Note:** Every class comes with a **default constructor** (zero argument constructor)

ex.

```C++
Node n;  // Default constructor (constructs all fields that are objects), does nothing in this case
```

This goes away if you write any constructor

Ex.

```C++
struct Node {
    int data;
    Node *next;

    Node (int data, Node *next = nullptr) {
        ...
    }
};

Node n {3};  // GOOD
Node n;  // BAD - no default constructor
```

## Object creation protocol

When an object is created, there are 4 steps:

1. Space is allocated
2. (later)
3. Fields are constructeed in declaration (field constructors called for fields that are objects)
4. Constructor body runs

Field initialization should happen in step 3, but constructor body happens in step 4

Consequence: object fields are intialized twice:

```C++
#include <string>

struct Student {
    int assns, mt, final;
    std::string name;

    Student (std::string name, int assns, int mt, int final) {
        this->name = name;  // etc.
    }
};

Student mike {"Mike", 90, 70, 60};
// name default-initialized in step 3 ("")
// then reassigned in step 4 ("Mike")
```

**To fix:** the Member Initialization List (MIL)

```C++
struct Student {
    Student (string name, int assns, int mt, int final): 
        name{name}, assns{assns}, mt{mt}, final{final}  // Step 3
    {  // Step 4

    }
}
```

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
    struct vector {
        size_t n, cap;
        int *theVector;

        vector();
        size_t size();
        int &itemAt(size_t i);
        void push_back();
        void pop_back();
        ~vector();
    };
}
#endif
```

_vector.cc_

```C++
#include "vector.h"

namespace {
    void increaseCap(vector &v) {
        ...
    }
}

const size_t startSize = 1;

CS246E::vector::vector(): 
    n{0}, cap{startSize}, theVector{new int[cap]} {
}

size_t CS246E::vector::size() {
    return n;
}

// Etc.

CS246E::vector::~vector() {
    delete[] theVector;
}
```

_main.cc_

```C++
int main() {
    vector v;   // Constructor is already called - no make_vector
    v.push_back(1);
    v.push_back(10);
    v.push_back(100);
    v.itemAt(0) = 2; 
}   // No dispose - destructor cleans v up
```

---
[Linear Collections and Modularity <<](./problem_2.md) | [**Home**](../README.md) | [>> Copies](./problem_4.md) 
