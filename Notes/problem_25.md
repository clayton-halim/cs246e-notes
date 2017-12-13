[<< Abstraction over Iterators](./problem_24.md) | [**Home**](../README.md) | [>> Collecting Stats](./problem_26.md)

# Problem 25: I want an even faster vector
**2017-11-21**

In the good old days of C, you could copy an array (even an array of structs!) very quickly by calling a function `memcpy` (similar to `strcpy`, but for arbitrary memory, not just strings).

`memcpy` was probably written in assembly, and was as fast as the machine could possibly be.

Nowadays in C++, copies invoke copy constructors, which are costly function calls.

_Good news!_

In C++, a type is considered POD (plain old data) if it:
- has a trivial default constructor (equiv. to `= default`)
- is triviable copiable
    - copy/move operations, destructor has default implementations
- is standard layout
    - no virtual methods or bases
    - no reference members
    - no fields in both base class & subclass, or in multiple base classes

For POD types, semantics is compatible with C, and `memcpy` is safe to use.

How can we use it? - Only safe to use if `T` is a POD type

_One option:_

```C++
template<typename T> class vector {
    private:
        size_t size, cap;
        T *theVector;
    public:
        vector(const vector &other): size{other.size}, cap{other.cap} {
            if (std::is_pod<T>::value) {
                memcpy(theVector, other.theVector, n * sizeof(T));
            } else {
                // as before
            }
        }
}
```

Works... But condition is evaluated at run-time, but the result is known at compile-time

_Second option (no run-time cost):_

```C++
template<typename T> class vector {
    ...
    public:
        template<typename X = T>
        vector(enable_if<std::is_pod<X>::value, const T&>::type other): ... {
            memcpy(...);
        }

        template<typename X = T>
        vector(enable_if<!std::is_pod<X>::value, const T&>::type other): ... {
            // orignal implementation
        }
}
```

How does it work?

```C++
template<bool b, typename> struct enable_if;
template<typename T> struct enable_if<true, T> {
    using type = T;
};
```

With metaprogramming, what you don't say is as important as what you do say.

If `b` is `true`, `enable_if` defines a struct whose 'type' member typedef is `T`. So if `std::is_pod<T>::value, const vector<T>&>::type => const vector<T> &`.

If `b` is `false`, the struct is declared but not defined. So `enable_if<b, T>` will not compile.

So one of the two versions of the copy constructor won't compile (the one with the `false` condition).

Then how is this a valid program?

**C++ rule:** SFINAE (Substitution Faliure Is Not An Error)

In other words - if `t` is a type,

```C++
template<typename T> __ f(___) { ____ }
```

if a template function, and substituting `T = t` results in an invalid function, the compiler does _not_ signal an error - it just removes that instansitation from consideration during overload resolution.

On the other hand, if _no_ version of the function is in scope and substitutes validly, that is an error.

Question: why is this wrong?

```C++
template<typename T> class vector {
    ...
    public:
        vector(typename enable_if<std::is_pod<T>::value, const vector<T>&>::type other): ... {
            ...
        }
};
```

That is, why do we need the extra template out front? 

Because SFINAE applies to template functions and these  methods are ordinary functions (constructors), not templates.
- They depend on `T`, but `T`'s value was deternmined when you decided what to put in the vector
- If substituting `T=t` fails, it invalidates the entire `vector` class, not just the method

So make a seperate template, with a new arg `X`, which can be defaulted to `T`, and do `is_pod<X>`, not `is_pod<T>`. 

... It compiles, but when we run it, it crashes

**Why? Hint:** if you put debug statements into both of these constructors, they don't print.

**Ans:** We're getting the compiler-supplied copy constructor, which is doing shallow copies.

These templates are not enough to suppress the auto-generated copy constructor. A non-templated match is always preferred to a templated one.

What do we do about it?

Could try: disabling the copy constructor

```C++
template<typename T> class vector {
    ...
    public:
        vector(const vector &other) = delete;
}
```

Not allowed, can't disable the copy constructor and then create another copy constructor.

Solution the works: overloading

```C++
template<typename T> class vector {
        ...
        struct dummy{};
    public:
        vector(const vector &other): vector{other, dummy{}} {}

        template<typename X = T> 
        vector(typename enable_if<...>::type other, dummy) { ... }

        template<typename X = T> 
        vector(typename enable_if<...>::type other, dummy) { ... }
};
```

- Overload the constructor with an unused "dummy" arg
- Have the copy consturctor delegate to the overloaded constructor
- Copy constructor is in line, so no function call overhead
- This works

Can write some "helper" definitions to make `is_pod` and `enable_if` easier to use.

```C++
template<typename T> constexpr bool is_pod_v = std::is_pod<T>::value
template<bool b, typename T> using enable_if_t = typename enable_if<b, T>::type
```

```C++
template <typename T> class vector {
        ...
    public:
        ...
        template<typename X = T> vector(enable_if_t<is_pod_v<X>, const vector<T>& other, dummy) { ... }

        template<typename X = T> vector(enable_if_t<!is_pod_v<X>, const vector<T>& other, dummy)
};
```

## Move / Forward implementation

We now have enough machinery to implement `std::move` and `std::forward`.

_`std::move`_ - first attempt
```C++
template<typename T> T &&move(T & x) {
    return static_cast<T &&>(x);
}
```

Doesn't quite work, `T&&` is a universal reference, not an rvalue reference. If `x` was an lvalue reference, `T&&` is an lvalue reference.
- Need to make sure `T` is not an lvalue reference
    - If `T` is an lvalue reference, get rid of the reference

```C++
template<typename T> inline typename std::remove_reference<T>::type && move(T &&x) {
    return static_cast<typename std::remove_reference<T>::type &&>(x);
    // turns T&, T&& into T
}
```

**Exercise:** write `remove_reference`

**Q:** can we save typing and use `auto`? Ex.
    ```C++
    template<typename T> auto move(T &&x) { ... }
    ```
**A:** No! By-value auto throws away reference and outer consts

```C++
int z;
int &y = z;
auto x = y; // x is an int

const int &w = z;
auto v = w; // int
```

By reference, `auto &&` is a universal reference

Need a type definition rule that doesn't discard references.

```C++
decltype(...)  // returns the type that ... was declared to have
decltype(var)  // returns the declared type of the variable
decltype(expr) // returns lvalue or rvalue, depending on whether the expr 
               //  is an lvalue or rvalue

int z;
int &y = z;
decltype(y) x = z;  // x is an int&
x = 4;  // Affects z

/* Path/Example 1 */
auto z;
x = 4;  // Does not affect z

/* Path/Example 2 */
decltype(z) s = z;  // s is an int
s = 5; // Does not affect z

/* Path/Example 3 */
decltype((z)) r = z;    // r is in int&&, since (z) is a ref.
r = t;  // Does affect z

decltype(auto) - perform type deduction, like auto, but use the decltype rules
```

```C++
template<typename T> decltype(auto) move(T &&x) {
    return static_cast<std::remove_reference_t<T>&&>(x);
}
```

_`std::forward`_
```C++
template<typename T> inline T&& move(T &&x) {
    return static_cast<T&&>(x);
}
```

**Reasoning:**
- If `x` is an lvalue, `T&&` is an lvalue reference
- If `x` is an rvalue, `T&&` is an rvalue reference

Doesn't work, `forward` is called on expressions that are lvalues, that may point at rvalues.

```C++
template<typename T> void f(T&& y) {
    ... forward(y) ...  // y is an lvalue
}
```

`forward(y)` is will _always_ yield an lvalue referernce.

In order to work, `forward` must know what type (including l/rvalue) was deduced for `y`, ie. needs to know `T`.

So in principle, `forward<T>(y)` would work.

**2 Problems:**
- Supplying `T` means `T&&` is no longer universal
- Want to prevent the user from omitting `<T>`

**Instead:** separate lvalue/rvalue cases

```C++
template<typename T> 
inline constexpr T&& forward(std::removed_reference_t<T>&x) noexcept {
    return static_cast<T&&>(x);
}

template<typename T> 
inline constexpr T&& forward(std::removed_reference_t<T>&&x) noexcept {
    return static_cast<T&&>(x);
}
```

---
[<< Abstraction over Iterators](./problem_24.md) | [**Home**](../README.md) | [>> Collecting Stats](./problem_26.md)
