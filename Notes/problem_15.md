[Memory management is hard <<](./problem_14.md) | [**Home**](../README.md) | [>> Insert/Remove in the middle](./problem_16.md)

# Problem 15: Is vector exception safe?
**2017-10-12**

Consider:

```C++
template<typename T> class vector {
        size_t n, cap;
        T *theVector;
    public:
        vector(size_t n, const T &x): n{n}, cap{n}, theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
            for (size_t i = 0; i < n; ++i) {
                new(theVector + i) T(x);    // copy constructor for T, could throw - then what?
            }
        }
};
```

- Partially constructed vector - destructor will not run
    - Broken invariant
- **Note:** if `operator new` throws - nothing has been allocated
    - No problem - strong guarantee

**Fix:**
```C++
template<typename T> class vector {
        size_t n, cap;
        T *theVector;
    public:
        vector(size_t n, const T &x): n{n}, cap{n}, theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
            size_t progress = 0;
            try {
                for (size_t i = 0; i < n; ++i) {
                    new(theVector + i) T(x);
                    ++progress;
                }
            } catch(...) {  // ... supresses all type checking (accept whatever)
                for (size_t i = 0; i < progress; ++i) theVector[i] ~T();
                operator delete(theVector);
                throw;  // rethrow
            }
        }
};
``` 

Abstract the filling part into its own function:
```C++
template<typename T> void uninitialized_fill(T *start, T *finish, const T &x) {
    T *p;
    try {
        for (p = start; p != finish; ++p) {
            new(static_cast<void *>(p)) T(x);
        }
    } catch(...) {
        for (T *q = start; q != p; ++q) q->~T();
        throw;
    }
}
```

This is an all or nothing function (strong guarantee).

```C++
template<typename T> class vector {
        size_t n, cap;
        T *theVector;
    public:
        vector(size_t n, const T &x): n{n}, cap{n}, theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
            size_t progress = 0;
            try {
                uninitialized_fill(theVector, theVector + n; x);
            } catch (...) {
                operator delete(theVector);
                throw;
            }
        }
};
```

Can clean this up using RAII on the array:

```C++
template<typename T> struct vector_base {
    size_t n, cap;
    T *v;
    vector_base(size_t n): 
        n{n}, cap{n == 0 ? 1 : n}, v{static_cast<T*>(operator new(cap * sizeof(T)))} {}
    ~vector_base() { operator delete(v); }
};

template<typename T> class vector {
        vector_base<T> vb;  // Cleaned up implicitly when vector is destroyed
    public:
        vector(size_t n, const T &x): vb{n} {
            uninitialized_fill(vb.v, vb.v + vb.n, x);
        }
        ~vector() { destroy_elements(); }
};
```  

**Copy Constructor:**
```C++
template<typename T> vector<T>::vector(const vector &other): vb{other.size()} {
    uninitialized_copy(other.begin(), other.end(), vb.v);   // Similar to uninitialized_fill
}
```

Assignment: copy & swap is exception-safe because swap is no-throw
Pushback:
```C++
void push_back(const T&x) {
    increaseCap();
    new(vb.v + (vb.n++)) T{x};  // If this throws, have the same vector
                    // Don't increment n before you know the construction succeded
}
``` 

What about `increaseCap`? 

```C++
void increaseCap() {
    if (vb.n == vb.cap) {
        vector_base vb2 {2 * vb.cap};   // RAII
        uninitialized_copy(vb.v, vb.v + vb.n, vb2.v);   // Strong guarantee
        destroy_elements();
        std::swap(vb, vb2); // no throw
    }
}
```

**Note:** only `try` blocks are in `uninitialized_copy` + `uninitialized_fill`.

But we have an efficiency issue - copying from old array to the new one knowing that the old array is going to be destroyed. Moving would be better.
- But moving destroys the old array, so if an exception is thrown during moving, our vector is destroyed
- Therefore we can only move if we are sure that the move operation is nothrow.

```C++
void increaseCap() {
    if (vb.n == vb.cap) {
        vector_base vb2 {2 * vb.cap};
        uninitialized_copy(vb.v, vb.v + vb.n, vb2.v);   
        destroy_elements();
        std::swap(vb, vb2); 
    }
}

template<typename T>
void uninitialized_copy_or_move(T *start, T *finish, T *target) {
    T *p;
    try {
        for (p = start; p != finish; ++p, ++target) {
            new(static_cast<void*>(target)) T{std::move_if_noexcept(*p)};
        }
    } catch(...) {
        for (T *q = start; q != p; ++q) (target + (q - p))->~T();
    }
}
```

`std::move_if_noexcept(x)` produces `std::move(x)` if `x` has a non-throwing move constructor, produces `x` otherwise.

But how should the compiler know if `T`'s move constructor is non-throwing? You have to tell it:

```C++
class C {
    public:
        C(C &&other) noexcept;
        ...
}
```

In general: moves and swaps should be non-throwing. Declare them so - will allow more optimized code to run.

Any function you are sure will never throw or propogate an exception, you should declare `noexcept`.

**Q:** Is `std:swap` `noexcept`?

```C++
template<typename T> void swap(T &a, T &b) {
    T c (std::move(a))
    a = std::move(b);
    b = std::move(c);
}
```

**Answer:** Only if T has a `noexcept` move constructor and a `noexcept` move assignment. How do we specify this?

```C++
template<typname T> void swap(T &a, T &b) 
    noexcept(std::is_nothrow_move_constructible<T>::value &&
             std::is_nothrow_move_assignable<T>::value) {
    ...
}
``` 

**Note:** `noexcept` = `noexcept(true)`

---
[Memory management is hard <<](./problem_14.md) | [**Home**](../README.md) | [>> Insert/Remove in the middle](./problem_16.md)
