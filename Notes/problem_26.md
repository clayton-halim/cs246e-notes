[<< I want an even faster vector](./problem_25.md) | [**Home**](../README.md) | [>> Resolving Method Overrides at Compile-Time](./problem_27.md)

# Problem 26: Collecting Stats
**2017-11-23**

I want to know how many `Student`s I create. 

```C++
class Student {
        int assns, mt, finals;
        static int count;   // Associated with the class, not one per object
    public:
        Student(...) { ++count; }
        static int getCount() { return count; } // static methods
};
```

- `static` methods have no `this` parameter
    - Thus not really a method, more like scoped function

_`.cc`_
```C++
int Student::count = 0; // must define the variable
```

Now

```C++
Student s1{...}, s2{...}, s3{...};

std::cout << Student::getCount() << std::endl;
```

Now I want to count objects in other classes. How do we abstract the solution into reusable code.

```C++
template<typename T> struct Count {
        static int count;
        Count() { ++count; }
        Count(const Count &) { ++count; }
        Count(const &&) { ++count; }
        ~Count() { --count; }
        static int getCount() { return count; }
};

template<typename T> int Count<t>::count = 0;
```

```C++
class Student: count<Student> {
        int assns, mt, final;
    public:
        Student(...): ...
        // accessors
        using Count<Student>::getCount; // Make this function visible 
}
```

- **Private Inheritance**
    - inherits `Count`'s implementation without creating an "is-a" relationship
    - Members of  `Count` become private in `Student`

Now we can easily add it to other classes:
```C++
class Book:Count<Book> {
        ...
    public:
        using Count<Book>::getCount;
};
```

Why is `Count` a template?
- So that for each class `C`, `class C: Count<C>` creates a new unique, instantiation of `Count` for each `C`. This gives `C` its own counter vs. sharing one counter over all subclasses

This technique (inheriting from a template specialized by yourself)
- Looks weird, but happens enough to have its own name: **The Curiously Recurring Template Pattern (CRTP)**

---
[<< I want an even faster vector](./problem_25.md) | [**Home**](../README.md) | [>> Resolving Method Overrides at Compile-Time](./problem_27.md)
