[Linear Collections and Memory Management << ](./problem_3.md) | [**Home**](../README.md) | [>> Moves](./problem_5.md) 

# Problem 4: Copies
**2017-09-20**

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
        data{other.data}, next{other.next ? new Node{*(other.next)} : nullptr} {  // Account for dereferencing a nullptr
            ...
        }
};
```

```C++
Vector v;
Vector w;

w = v;  // Copy, but not a construction
        // Copy assignment operator
        // Compiler supplied: copies each field (shallow), leaks w's old data
```

## Deep copy assignment

```C++
struct Node {
    Node &operator=(const Node &other) {
        data = other.data;
        next = other.next ? new Node{*(other.next)} : nullptr;

        return *this;
    }
};
```

**WRONG - dangerous**

Consider:
```C++
Node n {...};
n = n;
```

Destroys `n`'s data and then copies it. Must always ensure the operator = works in the case of self assignment

```C++
Node &Node::operator=(const Node &other) {
    if (this != &other) {
        data = other.data;
        next = other.next ? new Node{*other.next} : nullptr;
    }

    return *this;
}
```

**Alternative: copy-and-swap idiom**

```C++
#include <utility>

struct Node {
    ...
    void swap(Node &other) {
        using std::swap;
        swap(data, other.data);
        swap(next, other.next);
    }

    Node &operator=(const Node &other) {
        Node tmp = other;
        swap(tmp);

        return *this;
    }
};
```
---
[Linear Collections and Memory Management << ](./problem_3.md) | [**Home**](../README.md) | [>> Moves](./problem_5.md) 
