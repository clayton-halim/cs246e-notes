[Abstraction over containers << ](./problem_17.md) | [**Home**](../README.md)

# Problem 18 - Heterogenous Data
**2017-10-19**

I want a mixture of types in my vector.

Can't do this with a template.

```C++
vector<template<typename T> T> v;
```

- Not allowed - templates are compile time entities, don't exist at runtime

Ex. Fields of a struct

```C++
class MediaPlayer {
    template<typename T> T nowPlaying;   // Can't do this
};
```

What's available in C:

**unions** 

```C
union Media {Song s; Movie m};
Media nowPlaying;
```

Nice but you don't know what it might be exactly.

**void\***

```C
void *nowPlaying;
```

Even worse, can point to anything

These are not type safe.

Items in a heterogeneous collection usually have something in common, ex. Provide a common interface.

Can be viewed as different kinds of a more general "thing".

We'll use the standard CS 246 example.

```C++
class Book {    // Superclass or Base class
        string title, author;
        int length;
    public:
        Book(string title, string author, int length):
            title{title},
            author{author},
            length{length} {}

        bool isHeavy() const { return length > 100; }

        string getTitle() const { return title; }

        // etc.
};

BOOK
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+

// Some books are special though

class Text: public Book {   // Subclass or Derived class
        string topic;   // No need to mention title, etc. because it comes from book
    public:
        Text(string title, string author, int length, string topic): 
            Book{title, author, length}, 
            topic{topic} {}

        bool isHeavy() const { return length > 500; }
        string getTopic() const { return topic; }
};

TEXT
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+
| Topic  |
+--------+

class Comic: public Book {
        string hero;
    public:
        Comic(string title, string author, int length, string hero):
            Book{title, author, length},
            hero{hero} {}

        bool isHeavy() const { return length > 50; }
        string getHero() const { return hero; }
};

COMIC
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+
| Hero   |
+--------+
```
Subclasses inherit all members (fields & methods) from their superclass.  
All three classes have `title`, `author`, and `length`, methods `getTitle`, `getAuthor`, `getLength`, `isHeavy`, ... except this doesn't work.

`length` is a private method in `Book`, `Text` cannot access it.

**2 options:**

1. _Use protected_
```C++
class Book {
        string title, author;
    protected:  // Accessible only to this class and its subclasses
        int length;
    public:
        ...
};
```

2. _Call public method_
```C++
bool Text::isHeavy() const { return getLength() > 500; }
```

Recommended option is 2.
- You have no control over what subclasses might do
- Protected weakens encapsulation (cannot enforce invariants on protected fields)

If you want subclasses to have priviledged access
- Keep fields private
- Provide protected `get_` and `set_` methods

## Updated object creation/destruction protocols

**Creation:**
1. Space is allocated
1. Superclass part is constructed
1. Fields constructed in declaration order
1. Constructor body runs

**Destruction:**
1. Destructor body runs
1. Fields are destructed in reverse declaration order
1. Superclass part destructed
1. Space deallocated

Must revist everything to see the effect of inheritance

## Type compatibility
`Text`s and `Comic`s are special kinds of `Book`s - should be usable in place of `Book`s

```C++
Book b = Comic{___, ___, 75, ___};

// method calls:
b.isHeavy();


``` 
This is a light `Book`, but a heavy `Comic`. What does this return? -> Returns false
If `b` is a `Comic`, why is it acting like a `Book`? -> Because it is a `Book`!

Consequence of stack-allocated objects:

```C++
// Set aside enough space to hold a book
       +----+                +----+
       |    |                |    |
       +----+                +----+
Book b |    |  = Comic {...} |    |
       +----+                +----+
       |    |                |    |
       +----+      <---      +----+
                             |    |
                             +----+
```

Keeps only the `Book` part - `Comic` part is chopped off - **slicing**
- So it really is just a Book now
- Therefore it is `Book::isHeavy` that runs

Slicing happens even if superclass & subclass are the same size.

Similarily, if you want to collect your books:

```C++
vector<Book> library;
library.push_back(Comic {...}); 
```
only the book part will be pushed - _not_ a heterogeneous collection.

Also note:
```C++
void f(Book book[]);
Comic comics[] = {...};
f(comics);  // Will compile but never do this!
```

- Array will be misaligned
- Will not act like an array Books
- Undefined behaviour!

Slicing does not happen through pointers

So if I do this instead:

```C++
Book *p = new Comic{___, ___, 75, ___};

p
+--+     +---+   
|  | --> |   |  
+--+     +---+
         |   |
         +---+
         |   |
         +---+
         |   |
         +---+
```

But `p->isHeavy();` is still false!

**Rules:** the choice of which isHeavy is based on the type of the pointer (static typeou want to ), not the object (dynamic type).

Why? Because it's cheaper

**C++ Design Principle:** If you don't use it, you shouldn't have to pay for it.

That is if you want something more expensive, you have to ask for it. To make `*p` act like a comic when it is a comic:

```C++
class Book {
        ...
    public:
        ...
        virtual bool isHeavy() const { ... }

};

class Comic {
        ...
    public:
        ...
        bool isHeavy() const override { ... }
};

// Assume isHeavy is virtual
p->isHeavy();   // true!
```

`override` is a contextual keyword, is only a keyword in that specific location.


