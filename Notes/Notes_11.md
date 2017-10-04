# Problem 11: Better Initialization
**2017-10-04**

```C++
a[] = {1, 2, 3, 4, 5};  // Array    :)

Vector<int> v;  // Vector   :(
v.push_back(1);
...
```

Long sequence of push backs can be very clunky

Goal: better initialization

```C++
template<typename T> class Vector {
    ...
    public:
        Vector(): ...
        Vector(size_t n, T i = T{}): n {n}, cap {n == 0 ? 1 : n}, theVector{new T[cap]} {
            for (size_t j = 0; j < n; ++j) {
                theVector[j] = i;
            }
        }
};
```

Now:

```C++
Vector<int> v;  // Empty
Vector<int> w{5};   // 0, 0, 0, 0, 0
Vector<int> z{3, 4};    // 4, 4, 4
```

**Notes:** `T{}` (default constructor) means 0 if T is a built-in type

Better - what about true array-style initialization?

```C++
#include <initializer_list>
template <typename T> class Vector {
    ...
    public:
        Vector() ...;
        Vector(size_t n, T i = T{}) ...
        Vector(std::initializer_list<T> init): 
            n{init.size()}, cap{init.size()}, theVector{new T[cap]} {
                size_t i = 0;
                for (auto &t: init) theVector[i++] = t;
            }
};

Vector<int> v {1, 2, 3, 4, 5};  // 1, 2, 3, 4, 5
Vector<int> v;  // Empty
Vector<int> v{5};   // 5
Vector<int> v{3, 4};    // 3, 4
```

Default constructors take precedence over initializer lists, which take precedence over other constructors

To get the other constructor to run: **round bracket intialization**

```C++
Vector<int> v(5);   // 0, 0, 0, 0, 0
Vector<int> v(3, 4);    // 4, 4, 4
```

A note on cost: item in an initializer list are stored in contiguous memory (begin method returns a pointer)
- So we are using one array to build another (2 compies in memory)

Also note:
- Initializer lists are meant to be immutable
- Do not try to modify their contents
- Do note use them as standalone data structures
- Only one allocation in vector, not several
- No doubling + reallocating
- If general, if you know how big your vector will be, you can save reallocation cost by requesting space up front

```C++
template<typename T> class Vector {
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

**Exercise:** rewrite Vector such that `push_back` uses `reserve` instead of `increaseCap`

```C++
Vector<int> v;
v.reserve(10);
v.push_back(__);    // Can do 10 push_backs without needing to reallocate
...
```

## Problem 12: I want a vector of posns

```C++
struct Posn {
    int x, y;
    Posn(int x, int y): x{x}, y{y} {}
};

int main() {
    Vector<Posn> v; // Won't compile, why not?
}
```

Take a look at Vector's constructor:

```C++
template<typename T> Vector<T>::Vector(): n{0}, cap{1}, theVector{new T[cap]} {}
```

Which `T` objects will be stored in the array?
- C++ always calls a constructor when creating an object.
- Which constructor gets called? The default constructor
- Posn doesn't have one
- Posn 

