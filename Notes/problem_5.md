[Copies <<](./problem_4.md) | [**Home**](../README.md) | [>> I want a constant vector](./problem_6.md)

# Problem 5: Moves
**2017-09-21**

Consider:

```C++
Node plusOne(Node n) {
    for (Node *p = &n; p; p = p->next) {
        ++p->data;
    }

    return n;
}

Node n {1, new Node {2, nullptr}};
Node m = plusOne(n);
```

In this case, "other" is a reference to the temporary object created to hold the result of plusOne.

- "Other" is a reference to this temporary
- Copy constructor deep-copies the data from this temporary

**But** the temporary is just going to be thrown out anyway, as soon as the statement `Node m = plusOne(n)` is done

It's wasteful to deep copy the temp, why not steal the data instead? - saves the cost of a copy
We need to be able to tell whether "other" is a reference to a temporary object, or a standalone object

**Rvalue references** - `Node &&` is a reference to a temporary object (rvalue) of type `Node`. We need a version of the constructor that takes a `Node &&`

**Move Constructors** - steals other's data

```C++
struct Node {
    ...
    Node(Node &&other): data{other.data}, next{other.next} {
        other.next = nullptr;
    }
};
```
Similarly:
```C++
Node m;
m = plusOne(n);  // assignment from temporary
```

**Move assignment operator**

```C++
struct Node {
    ...
    Node &operator=(Node &&other) {  // steal other's data
        using std::swap;            // destroy my old data
        swap(data, other.data);     // Easy: swap without the copy
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
        swap(data, other.data);
        swap(next, other.next);
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
    std::string name; // string is a class
    Student(const std::string &name): name{name} {  // copies name into field (uses copy ctor)
        ...
    }
};
```

What if `name` points to an rvalue?

```C++
struct Student {
    std::string name;

    Student (std::string name): name{name} {  // {name} may come from an rvalue, but it is an lvalue
        ... // in other words, name may refer to an rvalue but name itself is an lvalue
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

`name{std::move(name)}` forces `name` to be treated as an rvalue, now strings move constructor

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

## Copy/Move Elision

```C++
vector makeAVector() {
    return vector{} //   // Basic constructor
}

vector v = makeAVector();   // move ctor? copy ctor?
```

Try in g++, just the basic constructor, not copy/move

In some circumstances, the compiler is allowed to skip calling the copy/move constructors (but doesn't have to). `makeAVector()` writes its result directly into the space occupied by `v`, rather than copy/move it later.

Ex.
```C++
vector v = vector{};    // Formally a basic construction and a copy/move construction
                        // vector{} is a basic constructor
                        // Here though, the compiler is *required* to elide the copy/move
                        // So basic constructor here only 

vector v = vector{vector{vector{}}};    // Still one basic ctor only
```

Ex.
``` C++
void doSomething(vector v) {...};   // Pass-by-value - copy/move ctor

doSomething(makeAVector());   
```

Result of `makeAVector()` written directly into the param, there is no copy/move

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

---
[Copies <<](./problem_4.md) | [**Home**](../README.md) | [>> I want a constant vector](./problem_6.md)
