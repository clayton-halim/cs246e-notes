[Staying in bounds << ](./problem_9.md) | [**Home**](../README.md) | [>> Better Initialization](./problem_11.md) 

# Problem 10: I want a vector of chars
**2017-10-03**

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

**Note:** For templates, the implementation goes in `.h` file

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
    - The first time the compile encounters `Vector<int>`, it creates a version of the vector code where `int` replaces `T` and compiles that new class
    - Can't do that unless it has all the details about the class
    - So implementation must be available in `.h`
    - Can also write bodies inline

---
[Staying in bounds << ](./problem_9.md) | [**Home**](../README.md) | [>> Better Initialization](./problem_11.md) 
