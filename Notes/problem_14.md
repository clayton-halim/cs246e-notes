[Less Copying << ](./problem_13.md) | [**Home**](../README.md) | [>> Is vector exception safe?](./problem_15.md) 

# Problem 14: Memory management is hard!
**2017-10-12**

No it isn't.
- Vectors do everything arrays can
    - Grow as needed O(1) amortized time or better (reserve the sapce you need)
    - Clean up automatically when they go out of scope
    - Are tuned to minimize copying

Just use vectors, and you'll never have to manage arrays again.
C++ has enough abstraction facilites to make programming easier than C.

But what about single objects?

```C++
void f() {
    Posn *p = new Posn{1, 2};
    ...
    delete p;   // Must deallocate the posn!
}
```

First ask, do you really need to use the heap?

Could you have used the stack instead?

```C++
void f() {
    Posn p {1, 2};
    ...

}   // No cleanup necessary
```

But sometimes you do need the heap. Calling `delete` isn't so bad, but consider:

```C++
void f() {
    Posn *p = new Posn{1, 2};

    if (some condition) throw BadNews{};

    delete p;  
}
```

`p` is leaked if `f` throws (unacceptable).

Raising and handling an exception should not corrupt the program. 
- We desire exception safety.

Leaks are a corruption of your program's memory. This will eventualy degrade performance and crash the program.

If a program cannot recover from an expression without corrupting its memory, what's the point of recovering?

What constitues exception safety? 3 levels:
1. **Basic guarantee** - once an exception has been handled, the program is in some valid state, no leaked memory, no corrupted data structures, all invariants are maintained.
1. **Strong guarantee** - if an exception propogates out of a function `f`, then the state of the program will be as if `f` had not been called
    - `f` either succeeds completely or not at all.
1. **Nothrow guarantee** - a function `f` offers the nothrow guarantee if `f` never emits an exceptions and _always_ accomplishes its purpose


Will revist, but now coming back to `f`:

```C++
void f() {
    Posn *p = new Posn {1, 2};

    if (some condition) {
        delete p;
        throw BadNews{};
    }

    delete p;   // Duplicated effort! Memory management is even harder!
}
```

Want to guarantee that `delete p;` happens no matter what.
What guarantee does C++ offer?
- Only one: that destructors for stack-allocated objects will be called when the objects go out of scope.

So create a class with a destructor that deletes the pointer.

```C++
template<typename T> class unique_ptr {
        T *p;
    public:
        unique_ptr(T *p): p{p} {}
        ~unique_ptr() { delete p; }
        T *get() const { return p; }
        T *release() {
            T *q = p;
            p = nullptr;
            return q;
        }
};
```
```C++
void f() {
    unique_ptr<Posn> p {new Posn{1, 2}};

    if (some condition) throw BadNews{};

}
```

That's it! Less memory management effort than we started with!

Using `unique_ptr` can use `get` to fetch the raw pointer.

**Better** - make `unique_ptr` act like a pointer.

```C++
template<typename T> class unique_ptr {
        T *p;
    public:
        ...
        T &operator*() const { return *p; }

        T *operator->() const { return p; }

        explicit operator bool() const { return p; }    // Explicit prohibits bool b = p;
        void reset(T *p1) {
            delete p;
            p = p1;
        }

        void swap(unique_ptr<T> &x) {
            std::swap(p, x.p);  // Even though p is private, we are still inside the unique_ptr class, so we can access other unique_ptr's private fields
        }
};
```

Overloading `->` is weird.
- On the left is a pointer and on the right is a pointer
- No syntax for specifying something is a field name
- Thus you specify what the pointer is and C++ will automatically grab the field for you

But consider:

```C++
unique_ptr<Posn> p {new Posn{1, 2}}; 
unique_ptr<Posn> q = p;
```

Two pointers to the same object! Can't both delete it! (Undefined behaviour)


**Solution:** copying `unique_ptr`s are not allowed. Moving is ok though.

```C++
template<typename T> class unique_ptr {
    ...
    public:
        unique_ptr(const unique_ptr &other) = delete;
        unique_ptr &operator=(const unique_ptr &other) = delete;
        unique_ptr(unique_ptr &&other): p{other.p} { other.p = nullptr; }
        unique_ptr &operator=(unique_ptr &&other) {
            swap(other);
            return *this;
        }
};
```

**Note:** this is how copying of streams is prevented.

What happens when you pass a `unique_ptr` by value?

```C++
void f(unique_ptr<Posn> p);

unique_ptr<Posn> q = {...};
f(q);   // Will not compile!
f(std::move(q)) // However this will work, but then q loses what it points to
```

Passing a `unique_ptr` by value = transfer of ownership

**Small exception safety issue:**

```C++
class C {...};
void f(unique_ptr<C> x, int y) {...}
int g() {...}
f(unique_ptr<C> {new C;}, g());
```

C++ does not enforce order of argument evaluation
It could be:
1. `new C`
1. `g()`
1. `unique_ptr<c> {1.}`

Then what if `g` throws? 1. is leaked.

We can fix this by making 1. and 3. inseparable using a helper function

```C++
template<typename T, typename... Args> unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T> { new T(std::forward<Args> (args)...) };
}
```

`unique_ptr` is an example of the C++ idiom: **Resource Acquisition Is Initialization (RAII)**
- Any resource that must be properly released (memory, file handle, etc.) should be wrapped in a stack-allocated object whose destructor frees it
- Ex. `unique_ptr`, `ifstream`/`ofstream` aquire the resource when the object is initialized and release it when the object's destructor runs

---
[Less Copying << ](./problem_13.md) | [**Home**](../README.md) | [>> Is vector exception safe?](./problem_15.md) 
