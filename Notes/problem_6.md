[Moves << ](./problem_5.md) | [**Home**](../README.md) | [>> Tampering](./problem_7.md) 

# Problem 6: I want a constant vector
**2017-09-26**

Say we want to print a vector:

```C++
ostream &operator <<(ostream &out, const Vector &v) {
    for (size_t i = 0; i < v.size(), ++i) {
        out << v.itemAt(i) << " ";
    }
}
```

WON'T COMPILE!!! (lushman pls)

- Can't call `size()` and `itemAt()` on a const object. What if these methods change fields?
- Since they don't, declare them as `const`

```C++
struct vector {
    ...
    size_t size() const;    // Means these methods will not modify fields
    int &itemAt(size_t i) const;    // Can be called on const objects
    ...
};

size_t vector::size() const {return n};
int &vector:itemAt(size_t i) const {return theVector[i];}
```

Now the loop will work.

BUT:

```C++
void f(const vector &v) {
    v.itemAt(0) = 4;    // Works!! v not very const...
}
```

- `v` is a const object - cannot change `n`, `cap`, `theVector` (ptr)
- You can changed items pointed to by `theVector`

Can we fix this?

```C++
struct vector {
    ...
    const int &itemAt(size_t i) const;
};

const int &itemAt(size_t i) const {
    return theVector[i];
}
```

Now `v.itemAt(0) = 4` won't compile if `v` is const, but it also won't compile if `v` is not const

To fix: **const overloading**

```C++
struct vector {
    ...
    const int &itemAt(size_t i) const;  // Will be called if the object is const
    int &itemAt(size_t i);  // Will be called if object is non-const
};

inline const int &Vector::itemAt(size_t i) const {return theVector[i];}
inline int &vector::itemAt(size_t) {return theVector[i]};
```

Putting in `inline` tells the compiler to replace the function call with the function body to save the cost of having to call a function

Merely a suggestion, compiler can choose to ignore it if it sees fit. Good idea for small functions

So now `v.itemAt(0) = 4;` will only compile if and only if `v` is non-const

Now let's make it prettier:

```C++
struct vector {
    size_t size() const {return n;} // Method body inside class implcity declares the method inline
    const int &operator[](size_t i) const {return theVector[i]};
    int &operator[](size_t i) {return theVector[i];}    
};

ostream &operator<<(ostream &out, const vector &v) {
    for (size_t i = 0; i < v.size(); ++i) {
        out << v[i] << " ";
    }

    return out;
}
```

---
[Moves << ](./problem_5.md) | [**Home**](../README.md) | [>> Tampering](./problem_7.md) 
