# Rvalue references 
**2017-09-21**

Recall:
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

```C++
Vector v;
Vector w;

w = v;  // Copy, but not a construction
        // Copy assignment operator
        // Compiler supplied: copies each field (shallow), leaks w's old data
```

**Deep copy assignment**

```C++
struct Node {
    Node &operator=(const Node &other) {
        data = other.data;
        next = other.next ? new Node{*other.next} : nullptr;

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

## Problem 5: Moves

Consider:

```C++
Node plusOne(Node n) {
    for (Node *p = &n; p; p = p->next) {
        ++p->data;
    }

    return n;
}

Node n {1, new Node {2, nullptr}};
Node m = plusOne(n)
```

In this case `other` is a reference to this temporary object created to hold the result of plusOne.

- Other is a reference to this temporary
- Copy constructor deep-copies the data from this temporary

**but** the temporary is just going to be thrown out anyway, as soon as the statement `Node m = plusOne(n)` is done

It's wasteful to deep copy the temp, why not steal the data instead? - saves the cost of a copy
We need to be able to tell whether other is a reference to a temporary object, or a standalone object

**Rvalue references** - `Node &&` is a reference to a temporary object (rvalue) of type Node. Version of the constructor that takes a Node &&

**Move Constructors** - steals other's data

```C++
struct Node {
    ...
    Node(Node &&other): data{other.data}, next{other.next} {
        other.next = nullptr;
    }
};

// Similarily
Node m;
m = plusOne(n);  // assignment from temporary
```

**Move assignment operator**

```C++
struct Node {
    ...
    Node &operator=(Node &&other) {  // steal other's data
        using std::swap;            // destroy my old data
        swap(data, other.data);     // Easy: swap without the cop
        swap(next, other.next);

        return *this;
    }
};
```

Can combine copy/move assignment:

```C++
struct Node {
    ...
    Node &operator=(Node other) { 
        swap(other);
        return *this;
    }
};
```

- Unified assignment operator
    - Pass by value
    - Invokes copy constructor if an lvalue
    - Invokes move constructor if an rvalue

**Note:** copy/swap can be expensive, hand-coded operator may do less copying

But now consider:

```C++
struct Student {
    std::string name;
    Student(const std::string &name): name{name} {  // copies name into field (copy ctor)
        ...
    }
};
```

What if `name` points to an rvalue?

```C++
struct Student {
    std::string name;

    Student (std::string name): name{name} {  // {name} may come frm an rvalue, but it is an lvalue
        ...
    }
};
```

Will copy if `name` is an lvalue, moves if `name` is an rvalue

```C++
struct Student {
    std::string name;
    Student(std::string name): name{std::move(name)} {

    }
}
```

`name{std::move(name)}` forces `name` to be treated as an rvalue, now stirngs move constructor

```C++
struct Student {
    ...
    Student(Student &&other): //move constructor
        name{other.name} {
            ...
        }
}
```

If you don't define move operations, copy operations will be used

If you do define them, then replace copy operations whenever the arg is a temporary (rvalue)
