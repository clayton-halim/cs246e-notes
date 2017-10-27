[Heterogeneous Data << ](./problem_18.md) | [**Home**](../README.md) | [>> I want a class with no objects](./problem_20.md)

# Problem 19 - I'm leaking!
**2017-10-26**

```C++
class X {
        int *a;
    public:
        X(int n): a{new int[n]} {}
        ~X() { delete[] a; }
};

class Y: public X {
        int *b;
    public:
        Y(int n, int m): X{n}, b{new int[m]} {}
        ~Y() { delete[] b; }    // Note: Y's dtor will call X's dtor (step 3)
};

X *px = new Y{3, 4};
delete px;  // Leaks
```

This calls `X`'s destructor, but not `Y`'s

**Solution:** make the destructor _virtual_

```C++
class X {
        ...
    public:
        ...
        virtual ~X() { delete[] a; }   
};
```

Now there is no more leak.

__Always__ make the destructor virtual in classes that are meant to be superclasses, even if the destructor does nothing.
- You never know what the subclass' destructor might do, so you need to make sure its destructor gets called

If a class is not meant to be a superclass, then no need to incur the cost of virtual methods needlessly/
- Leave the destructor non-virtual

```C++
class X final { // Cannot be subclassed
    ...
};
```

Like `override`, `final` is another contextual keyword (right before the brace). 

---
[Heterogeneous Data << ](./problem_18.md) | [**Home**](../README.md) | [>> I want a class with no objects](./problem_20.md)