# Iterators (cont.)
**2017-10-03**

Recall: Encapsulation + Iterators for linked list

Can do the same for vectors:

```C++
class Vector {
    size_t size, cap;
    int *theVector;
    
    public:
        class iterator {
            int *p;
            ...
        };

        class const_iterator {
            ...
        };

        iterator begin() {
            return iterator{theVector};
        }

        iterator end() {
            return iterator{theVector + n};
        }

        const_iterator begin() const {
            return const_iterator{theVector};
        }

        const_iterator end() const {
            return const_iterator{theVector + n};
        }
};
```

Could do this, OR:

```C++
typedef int *iterator;
typedef const int *const_iterator;

---

using iterator = int*;
using const_iterator = const int*;

iterator begin() {return theVector;}
iterator end() {return theVector + n;}
```

## Problem 9: Staying in Bounds

```C++
Vector v;
v.push_back(4);
v.push_back(5);

v[2];   // Out of bounds! (undefined behaviour) - may or may not crash
```

Can we make this safer?

Adding bounds checks to operator`[]` may be needlessly expensive.

Could have a second, safer fetch operator:

```C++
class Vector {
    ...
    public:
        int &at (size_t i) {
            if (i <= n) return theVector[i];
            // else what?
        }
};
```

`v.at(2)` still wrong - what should happen?
- Return any `int`, looks like non-error
- Returning a non-`int`, not type correct
- Crash the program - can we do better? Don't return. don't crash

**Solution:** raise an `exception`

```C++
class range_error {};

class Vector {
    ...
    public:
        int &at(size_t i) {
            if (i < n) return theVector[i];
            else throw range_error{};   // Construct an object of range_error & "throw" it
        } 
};
```

- Client's options
1. Do nothing
    ```C++
    Vector v;
    v.push_back(0);
    v.at(1) // The exception will crash the program
    ```
1. Catch it
    ```C++
    try {
        Vector v;
        v.push_back(0);
        v.at(1);
    } catch (range_error &r) {  // r is the thrown object, catch by reference saves a copy operation
        // Do something
    }
    ```
1. Let your caller catch it
    ```C++
    int f() {
        Vector v;
        v.push_back(0);
        v.at(1);
    }
    int g() {
        try{
            f();
        } catch(range_error &r) {
            // Do something
        }
    }
    ```
    - Exception will propogate through the callchain until a handler is found.
    - Called **unwinding** the stack
    - If no handler is found, program aborts (`std::terminate` gets called)
    - Control resumes after the catch block (problem code is not retried)

What happens if a consturctor throws an exception?
- Object is considered **partially constructed**
- Destructor will not run on partiall constructed object

Ex.

```C++ 

class C {...};

class D {
    C a;
    C b;
    int *c;

    public:
        D() {
            c = new int[100];
            ...
            if (...) throw something {};  // (*)
        }

        ~D() {
            delete[] c;
        }
};

D d;
```

- (\*) the `D` object is not fully constucted, so `~D()` will not run on d
- But `a` and `b` are fully constucted so their destructors will run
- So if a constructor wants to throw, it must clean itself

```C++
class D {
    ...
    public:
        D() {
            c = new int[100];
            ...
            if (...) {
                delete[] c;
                throw something {};
            }
        }
    }
} 
```

What happens if a destructor throws? 
> Trouble

- By default, program aborts immediately
- `std::terminate`
- If you really want a throwing destructor, tag it with `noexcept(false)` 

```C++
class myexception{};

class C {
    ...
    public:
        ~C(_) noexcept(false) {
            throw myexception{};
        }
};
```
But watch out

```C++
void h() {
    C c1;
}

void g() {
    C c2;
    h();
};

void f() {
    try {
        g();
    } catch (myException &ex) {
        ...
    }
}
```

1. Destructor for `c1` throws at the end of `h`
1. Unwind through `g`
1. Destructor for `c2` throws as we leave `g`
    - No handler yet
    - Now two unhandled exceptions
    - Immediate termination guaranteed, `std::terminate` is called

Never let destructors throw, swallow the exception if necessary

Also note that you can throw _any value_, not just objects

## Problem 10: I want a vector of chars

Start over? No

Introduce a major abstraction mechanism, **templates**
- Generalize overtypes

_Vector.h_

```C++
namespace CS246E {
    template<typename T> class Vector {
            size_t n, cap;
            T* theVector;

        public:
            Vector();
            ...
            void push_back(T n);
            T &operator[](size_t i);
            const T &operator[] const(size_t)

            using iterator = T*;
            using const_iterator = const T*;
            // etc.
    };

    template<typename T> Vector<T>::vector() n{0}, cap{1}, theVector{new T[cap]} {}
    template<typename T> void push_back(T n) {...}
    /// etc.
}
```

**Note:** Must put implementation in file

_main.cc_

```C++
int main() {
    Vector<int> v;  // Vector of ints
    v.push_back(1);
    ...
    Vector<char> w; // Vector of chars
    v.push_back('a');
    ...
}
```

- **Semantics:**
    - The first time the compile encounters `Vector<int>`, it creates a version of the vector code where `int` reaplces `T` and compiles that new class
    - Can't do that unless it has all the details about the class
    - So implementation must be available in `.h`
    - Can also write bodies inline
