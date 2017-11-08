[I want a vector of chars <<](./problem_10.md) | [**Home**](../README.md) | [>> I want a vector of Posns](./problem_12.md)

# Problem 11: Better Initialization
**2017-10-04**

```C++
a[] = {1, 2, 3, 4, 5};  // Array    :)

vector<int> v;  // Vector   :(
v.push_back(1);
⋮
```

Long sequence of push backs can be very clunky

Goal: better initialization

```C++
template<typename T> class vector {
    ⋮
    public:
        vector(): ...
        vector(size_t n, T i = T{}): n {n}, cap {n == 0 ? 1 : n}, theVector{new T[cap]} {
            for (size_t j = 0; j < n; ++j) {
                theVector[j] = i;
            }
        }
};
```

Now:

```C++
vector<int> v;  // Empty
vector<int> w{5};   // 0, 0, 0, 0, 0
vector<int> z{3, 4};    // 4, 4, 4
```

**Notes:** `T{}` (default constructor) means `0` if `T` is a built-in type

Better - what about true array-style initialization?

```C++
#include <initializer_list>
template <typename T> class vector {
    ⋮
    public:
        vector() ...;
        vector(size_t n, T i = T{}) ...
        vector(std::initializer_list<T> init): 
            n{init.size()}, cap{init.size()}, theVector{new T[cap]} {
                size_t i = 0;
                for (auto &t: init) theVector[i++] = t;
            }
};
```
```C++
vector<int> v {1, 2, 3, 4, 5};  // 1, 2, 3, 4, 5
vector<int> v;  // Empty
vector<int> v{5};   // 5
vector<int> v{3, 4};    // 3, 4
```

Default constructors take precedence over initializer lists, which take precedence over other constructors

To get the other constructor to run: **round bracket intialization**

```C++
vector<int> v(5);   // 0, 0, 0, 0, 0
vector<int> v(3, 4);    // 4, 4, 4
```

A note on cost: item in an initializer list are stored in contiguous memory (begin method returns a pointer)
- So we are using one array to build another (2 copies in memory)

Also note:
- Initializer lists are meant to be immutable
- Do not try to modify their contents
- Do not use them as standalone data structures
- Only one allocation in vector, not several
- No doubling + reallocating
- If general, if you know how big your vector will be, you can save reallocation cost by requesting space up front

```C++
template<typename T> class vector {
    ...
    public:
    ...
    void reserve(size_t newCap) {
        if (cap < newCap) {
            T *newVec = new T[newCap];
            for (size_t i = 0; i < n; ++i) newVec[i] = theVector[i];
            delete[] theVector;
            theVector = newVec;
            cap = newCap;
        }
    }
};
```

**Exercise:** rewrite `vector` such that `push_back` uses `reserve` instead of `increaseCap`

```C++
vector<int> v;
v.reserve(10);
v.push_back(__);    // Can do 10 push_backs without needing to reallocate
...
```

---
[I want a vector of chars <<](./problem_10.md) | [**Home**](../README.md) | [>> I want a vector of Posns](./problem_12.md)
