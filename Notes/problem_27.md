[<< Collecting Stats](./problem_26.md) | [**Home**](../README.md) | [>> Polymorphic Cloning](./problem_28.md)

# Problem 27 - Resolving Method Overrides at Compile-Time
**2017-11-23**

**Recall:** Template Method Pattern

```C++
class Turtle {
    public:
        void draw() {
            drawHead();
            drawShell();
            drawFeet();
        }
    private:
        void drawHead();
        virtual void drawShell() = 0;   // vtable lookup
        void drawFeet();
};

class RedTurtle: public Turtle {
    void drawShell() override;
};
```

**Consider:**

```C++
template<typename T> class Turtle {
    public:
        void draw() {
            drawHead();
            static_cast<T *>(this)->drawShell();
            drawFeet();
        }
    private:
        void drawHead();
        void drawFeet();
};

class RedTurtle: public Turtle<RedTurtle> {
    friend class Turtle;
    void drawShell();
};

class GreenTurtle: public Turtle<GreenTurtle> {
    friend class Turtle;
    void drawShell();
};
```

No virtual method methods, no vtable lookup
- Drawback: no relationship between `RedTurtle` & `GreenTurtle`
    - Can't store a mix of them in a container

Can give `Turtle` a parent:

```C++
template<typename T> class Turtle: public Enemy { ... };
```

Then can store `RedTurtles` and `GreenTurtles`
- But then can't access the `draw` method
- Could give Enemy a virtual `draw` method

```C++
class Enemy {
    public:
        virtual void draw() = 0;
};
```

But then there will be a vtable lookup
- On the other hand, if `Turtle::draw` calls several would-be virtual helpers, could trade away several vtable lookups for one

---
[<< Collecting Stats](./problem_26.md) | [**Home**](../README.md) | [>> Polymorphic Cloning](./problem_28.md)