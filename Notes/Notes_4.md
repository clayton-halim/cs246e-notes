# Linear Collections and Modularity (cont.)
**2017-09-19**

**Readings:** 7.7.1, 14, 16.2 

_vector.cc_

```C++
#include "vector.h"

namespace {  // Anonymous namespace makes the function only visible to file (same as static in C)
    void increaseCap(CS246E::Vector &v) {
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

CS246E::Vector CS246E::make_vector() {
    Vector v {0, startSize, new int[startSize]};
    return v;
}

size_t CS246E::size(const Vector &v) {
    return v.size;
}

int &CS246E::itemAt(const Vector &v, size_t i) {
    return v.theVector[i];
}

void CS246E::push_back(Vector &v, int n) {
    increaseCap(v);
    v.theVector[v.size++] = n;
}

void CS246E::pop_back(Vector &v) {
    if (v.size > 0) {
        --v.size;
    }
}

void CS246E::dispose(Vector &v) {
    delete[] v.theVector;
}
```

_main.cc_

```C++
#include "vector.h"

using CS246E::vector;  # only allows you to use vector without CS246E::

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

**Object creation protocol**

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



