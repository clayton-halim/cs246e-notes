[<< Resolving Method Overrides at Compile-Time](./problem_28.md) | [**Home**](../README.md) | [>> Logging](./problem_29.md)

# Problem 28: Polymorphic Cloning
**2017-11-23**

```C++
Book *pb = ...;
Book *pb2 = // I want an exact copy of *pb;
```

Can't call constructor directly (we don't know what `*pb` is, ie. don't know which constructor to call).

**Standard Solution:** virtual clone method

```C++
class Book {
        ...
    public:
        virtual Book *clone() { return new Book{*this}; }
};

class Text: public Book {
        ...
    public:
        Text *clone() override { return new Text{*this} ;}
};

// Comic - similar
```

Boilerplate code - can we reuse it?

Works better with an abstract base class:

```C++
class AbstractBook {
    public:
        virtual AbstractBook *clone() = 0;
        virtual ~AbstractBook();
};

template<typename T> class Book_cloneable: public AbstractBook {
    public:
        T *clone() override { return new T{static_cast<T&>(*this)}; }
};

class Book: public Book_cloneable<Book> { ... };
class Text: public Book_cloneable<Text> { ... };
class Comic: public Book_cloneable<Comic> { ... };
```

---
[<< Resolving Method Overrides at Compile-Time](./problem_28.md) | [**Home**](../README.md) | [>> Logging](./problem_29.md)