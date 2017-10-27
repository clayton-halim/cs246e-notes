[I want a class with no objects << ](./problem_20.md) | [**Home**](../README.md)

# Problem 21 - The copier is broken

How do copies and moves interact with inheritance?

**Copy constructor:** 
```C++
Text::Text(const Text &other): Book{other}, topic{other.topic} {}
```

**Move constructor:** 
```C++
Text::Text(const Text &other): 
    Book{std::move(other)}, 
    topic{std::move(other.topic)} {}
```

We can still call `std::move(other.topic)` even though we already moved other because `std::move(other)` only took the "`Book` part", leaving `other.topic` behind.

**Copy/Move assignment:**
```C++
Text &Text::operator=(Text other) {
    Book::operator=(std::move(other));
    topic = std::move(other.topic); 
}
```

But consider:
```C++
Book *b1 = new Text{...};  // Author1 writes about BASIC
Book *b2 = new Text{...};  // Author2 writes C++

*b1 = *b2;
```
Now Author2 writes about BASIC and Author1 writes about C++
- Essentially only the book part gets copied over

...