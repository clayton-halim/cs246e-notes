[I want a constant vector <<](./problem_6.md) | [**Home**](../README.md) | [>> Efficient Iteration](./problem_8.md)

# Problem 7: Tampering
**2017-09-21**

```C++
vector v;
v.cap = 100;    // Sets cap without allocating memory
v.push_back(1);
v.push_back(10);
...  // Undefined behaviour - will likely crash
```
**Interfering with ADTs (Abstract Data Types)**

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
struct vector {
    private:    // Fields are only accessible within the vector class
        size_t n, cap;      
        int *theVector;
    public:     // Visible to all
        vector();
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
struct vector {
    private:
        size_t ...
        ...
    public:
        vector();
        ...

    private:
        void increaseCap(); // Now a private method
};
```

NOW
_vector.cc_
```C++
namespace CS246E {
    vector::vector() {...}  
    // etc. as before
    void vector::increaseCap() {...}  // Don't need anonymous namespaces anymore!
};
```

- **Structs** (`struct`)
    - Default public access
    - Would be better if we had default private
- **Classes** (`class`)
    - Exactly like struct, except private default access

_vector.cc_
```C++
class vector {
        size_t n, cap;
        int *theVector;
    public:
        vector();
        ...

    private:
        void increaseCap(); // Now a private method
};
```

A similar problem with linked lists:

```C++
Node n {3, nullptr};    // Stack allocated
Node m {4, &n}; // m's dtor will try to delete &n (undefined)
```

There was an invariant that - `next` is `nullptr` or was allocated by `new`

How can we enforce this? 
- Encapsulate Node inside a "wrapper" class

```C++
class list {
    struct Node {    // Private nested class - not available outside
        int data;
        Node *next; // ... methods
    };

    Node *theList;
    
    public:
        list(): theList{nullptr} {}
        ~list() {delete theList;}
        size_t size() const;

        void push_front(int n) {
            theList = new Node{n, theList};
        }

        void pop_font() {
            if (theList) {
                Node *tmp = theList;
                theList = theList->next;
                tmp->next = nullptr;
                delete tmp;
            }
        }

        const int &operator[](size_t i) const {
            Node *cur = theList;
            for (size_t j = 0; j < i && cur; ++j, cur=cur->next);
            return curr->daata;
        }

        int &operator[](size_t i) {
            Node *cur = theList;
            for (size_t j = 0; j < i && cur; ++j, cur=cur->next);
            return curr->data;
        }   
};
```
Client cannot manipulate the list directly
- No access to next pointers
- Invariant is maintained

---
[I want a constant vector <<](./problem_6.md) | [**Home**](../README.md) | [>> Efficient Iteration](./problem_8.md)