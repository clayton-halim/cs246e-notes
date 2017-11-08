[Better Initialization << ](./problem_11.md) | [**Home**](../README.md) | [>> Less Copying](./problem_13.md) 

# Problem 12: I want a vector of posns
**2017-10-04**

```C++
struct Posn {
    int x, y;
    Posn(int x, int y): x{x}, y{y} {}
};

int main() {
    vector<Posn> v; // Won't compile, why not?
}
```

Take a look at Vector's constructor:

```C++
template<typename T> vector<T>::vector(): n{0}, cap{1}, theVector{new T[cap]} {}
```
`T[cap]` creates an array of T objects. Which `T` objects will be stored in the array?
- C++ always calls a constructor when creating an object.
- Which constructor gets called? The default constructor
- But `Posn` doesn't have one

Need to separate memory allocation (Object creation step 1) from initialization (steps 2-4)

**Allocation:** `void *operator new(size_t)`
- Allocates `size_t` bytes
- No initialization
- Returns `void*`

**Note:** 
- In C, `void*` implicity converts to any pointer type
- In C++, the conversion requires a cast

**Initialization:** "Placement new"
- `new (address) type`
- Constructs a "type" object at "address"
- Does not allocate memory (memory should already be allocated at "address")

```C++
template<typename T> class vector {
    ...
    public:
        vector(): n{0}, cap{1}, theVector{static_cast<T*>(operator new(sizeof(T)))} {}
        vector(size_t n, T x = T{}): 
            n{n}, cap{n}, theVector{static_cast<T*>(operator new(n *sizeof(T)))} {
            
            for (size_t i = 0; i < n; ++i)
                new(theVector + i) T(x);
        }

        ...

        void push_back(T x) {
            increase_cap();
            new(theVector + (n++)) T(x);
        }

        void pop_back() {
            if (n) {
                theVector[n-1].~T() // Must explicitly invoke destructor
                --n;
            }
        }

        ~vector() {
            destroy_items();
            operator delete(theVector);
        }

        void destroy_items() {
            for (auto &x: *this)
                x.~T();
        }
};
```

---
[Better Initialization << ](./problem_11.md) | [**Home**](../README.md) | [>> Less Copying](./problem_13.md) 
