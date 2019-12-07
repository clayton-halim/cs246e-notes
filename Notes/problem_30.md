[<< Logging](./problem_29.md) | [**Home**](../README.md) | [>> I want total control over vectors and lists](./problem_31.md) 

# Problem 30: Total Control
**2017-11-28**

Can control:
- Parameter passing
- Initialization
- Method call resolution
- Etc.

Control over memory allocation:

Memory allocators are tricky to write, so we have 2 questions:
- **Why write an allocator?**
    - Built-in one is too slow
    - General purpose, not optimized for specific use
        - Ex. If you know you will always allocate objects of the same size, a custom allocator may perform better
        - Ex. Optimize locaility
            - Maybe you want a separate heap, just for objects of some class `C`, keeps the objects close to each other. May improve performance.
        - Ex. You want to use "special memory"
            - Put objects in shared memory
        - Ex. You want to profile your program
            - Collect stats
- **How do you customize an allocation?**
    - Overload `operator new`
    - If you define a global non-member `operator new`, all allocations in your program will use your allocator
    - Also if you write `operator new`, you need to write `operator delete`. Else undefined behaviour
    - Ex.
    ```C++
    void *operator new(size_t size) {
        std::cout << "Request for " << size << "bytes\n";
        return malloc(size);
    }
    ```
    ```C++
    void operator delete(void *p) {
        std::cout << "Freeing " << p << std::endl;
        free(p);
    }
    ```
    ```C++
    int main() {
        int *x = new int;
        delete x;
    }
    ```
    - Works but is not correct. Doesn't adhere to convention
        - If `operator new` fails, it is supposed to throw `bad_alloc`
        - But actually if `operator new` fails, it is supposed to call the `new_handler` function
            - The `new_handler` can:
                - Free up space (somehow)
                - Install a different `new_handler`/uninstall the current
                - Throw `bad_alloc`
                - Abort/exit
            - `new_handler` should be called in an infinite loop
            - If `new_handler` is a `nullptr`, `operator_new` throws
            - Also `new` must return a valid pointer if `size == 0` and `delete` of a `nullptr` must be safe
    - Correct implementation:
    ```C++
    #include <new>
    void *operator new(size_t size) {
        std::cout << " " << 
        if (size == 0) size = 1;
        while (true) {
            void *p = malloc(size);
            if (p) return p;
            std::new_handler h = std::set_new_handler();
            std::set_new_handler(h);
            if (h) h();
            else throw std::bad_alloc{};
        }
    }
    void operator delete(void *p) {
        if (p == nullptr) return;
        std::cout << "Deleting" << p << std::endl;
        free(p);
    }
    ```
    - Replacing global `operator new`/`delete` affects your entire program
    - More likely - replace on a class-by-class basis
        - Especially if you are writing allocators specifically tuned to the sizes of your objects
    - To do this - make `operator new`/`delete` members
        - Must be `static` members
    ```C++
    class C {
        public:
            static void *operator new(size_t size) {
                std::cout << "Running class C's allocator" << std::endl;
                return operator new(size);
            }
            static void operator delete(void *p) noexcept {
                std::cout << "Freeing " << p << std::endl;
                ::operator delete(p);   // :: refers to global namespace
            }
    };
    ```
    - Generalize - log to an arbitrary stream
    ```C++
    class C {
        public:
            static void *operator new(size_t size, std::ostream &out) {
                out << "Running class C's allocator" << std::endl;
                return operator new(size);
            }
    };
    ```
    ```C++
    C *x = new(std::cout) C;  // log to cout
    ofstream f{...}
    C *y = new(f) C;
    ```
    - Note when you create a `new` that takes in parameters, it is called **placement new**, not to be confused with the other **placement new** where you intialize an object into already allocated memory
    - Must also write the corresponding "placement" `delete`.
    ```C++
    class C {
        public:
            ...
            static void operator delete(void *p, std::ostream &out) noexcept {
                out << "Placement delete: " << p << std::endl;
                ::operator delete(p);   
            }
    };
    ```
    - Won't compile! You also need "ordinary" delete
    ```C++
    class C {
        public:
            ...
            static void operator delete(void *p) noexcept {
                std::cout << "Ordinary delete (cout): " << p << std::endl;
                ::operator delete(p);   
            }
    };
    ```
    ```C++
    C *p = new(cout) C; // Running C's allocator
    delete p;   // Ordinary delete (cout)
    ```
    - If the client calls `delete p`, there needs to be a non-specialized `operator delete`, else compile-error
    - How can you call sepcialized delete? You can't
    - Then why do we need it?
    - If the constructor the called specialized `operator new` throws, it will call the specialized `operator delete` that matches `operator new`
        - If there isn't one, no `delete` gets called => leak
    - Ex.
    ```C++
    class C {
        ...
        public:
            C(bool b) { if (b) throw 0; }
    };
    ```
    ```C++
    try {
        C *p = new(std::cout) C{true};  // throws, calls specialized delete
        delete p; // Not reached
    } catch(...) {}
    C *q = new(std::cout) C{false}  // Does not throw
    delete q;   // Ordinary operator delete
    ```

Customizing array allocation - overload `operator new[]` and `operator delete[]`.

---
[<< Logging](./problem_29.md) | [**Home**](../README.md) | [>> I want total control over vectors and lists](./problem_31.md) 
