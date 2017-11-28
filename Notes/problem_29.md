[<< Polymorphic Cloning](./problem_28.md) | [**Home**](../README.md) | [>> Total Control](./problem_30.md)

# Problem 29: Logging
**2017-11-23**

We want to encapsulate logging functionality and "add" it to any class.

```C++
template<typename T, typename Data> class Logger {
    public:
        void loggedSet(Data x) {
            std::cout << "setting data to " << x << std::endl;
            static_cast<T*>(this)->set(x);  // No virtual call overhead
        }
};

class Box: public Logger<Box, int> {
        friend class Logger<Box, int>;
        int x;
        void set(int y) { x = -y; }
    public:
        Box(): x{0} { loggedSet(0); }
};

Box b;
b.loggedSet(1);
b.loggedSet(4);
// etc.
```

Another approach:

```C++
class Box {
        int x;
    public:
        Box(): x{0} {}
        void set(int y) { x = y ;}
};

                                    // Mixin Inheritance
template<typename T, typename Data> class Logger: public T {
    public:
        void loggedSet(Data x) {
            std::cout << "setting to" << x << std::endl;
            set(x); // No vtable overhead
        } 
};

using BoxLogger = Logger<Box, int>;
Boxlogger b;
b.loggedSet(1);
b.loggedSet(4);
//etc.
```

**Mixins** - can mix and match subclass functionality without writing new classes

Note: if `SpecialBox` is a subclass of `Box`, then `SpecialBox` has no relation to `Logger<Box, int>`. Nor is there any relationship between `Logger<SpecialBox, int>`, `Logger<Box, int>`.

But with CRTP, `SpecialBox` is a subtype of `Logger<Box, int>`
- Can specialize behaviour of virtual functions

---
[<< Polymorphic Cloning](./problem_28.md) | [**Home**](../README.md) | [>> Total Control](./problem_30.md)