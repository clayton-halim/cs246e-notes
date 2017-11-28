[I want to know what kind of Book I have << ](./problem_22.md) | [**Home**](../README.md) | [>> Shared Ownership](./problem_23.md)

# A Big Unit on Object Oriented Design
**2017-11-02**

**System Modelling - UML**
- Unified Modelling Language
    - Make ideas easy to communicate
    - Aid design discussions

<pre>
                                +---------------------+
                                |BOOK                 | - Class Name [italics = absract]
                                +---------------------+
                                |- title: String      |
                                |- author: String     | - Fields (optional)
                                |# length: Integer    |
                                +---------------------+
                                |+ getTitle: String   |
                                |+ getAuthor: String  | - Methods (optional) [italics = virtual]
                                |+ getLength: Integer |
                                |+ <em>isHeavy:</em> Boolean   | 
                                +---------------------+
                                           ^
                                           |
                                           | - (is-a relationship: inheritance)
                                           |
                   +-----------------------+--------------------------+
                   |                                                  |
        +---------------------+                             +---------------------+
        |Text                 |                             |Comic                | 
        +---------------------+                             +---------------------+
        |- topic: String      |                             |- hero: String       |
        +---------------------+                             +---------------------+
        |+ getTopic: String   |                             |+ getTopic: String   |
        |+ isHeavy: Boolean   |                             |+ isHeavy: Boolean   | 
        +---------------------+                             +---------------------+ 
</pre>

- `-` private
- `#` protected
- `+` public

<pre>
+-----+   m +-------+
| Car |◆--->| Motor |
+-----+   2 +-------+
</pre>

- "Owns-a" relationship (composition)
- Means the motor is part of the car
    - Does not have an independent exisitance
    - Copy/destroy the car => copy/destroy the motor (deep copies)
- Typical implementation - class composition, ie. object fields
- `m` on arrow indicates the name of the field
    - `class Car { Motor m; };`
- Number under arrow indicates how many fields

<pre>
+------+  0..*  +-------+
| Pond |◇------>| Ducks |
+------+        +-------+
</pre>

- "has-a" relationship (aggregation)
- Duck has its own independent existence
- Copy/destroy the pond =/=> copy/destroy the ducks
- Typical implementation: pointer field
- Ex. `class Pond { vector<Duck*> ducks; };`

The concept of _ownership_ is central to object oriented design in C++.

Every resource (memory, file, window) should be owned by an object that will release it (RAII). A unique pointer _owns_ the memory it points to. A raw pointer should be regarded as _not_ owning the memory it points.

If you need to point at the same object with several pointers, one pointer should own it and be a unique pointer. The rest should be raw pointers. Moving a unique pointer = transferring ownership.

If you need true shared ownership - we'll see later.

## Measures of Design Quality

- **Coupling**
    - How strongly different modules depend on each other
    - **Low:** 
        - Function calls with params/results of basic type 
        - Function calls with array/struct params
        - Modules affect each other's control flow
        - Modules share global data 
    - **High:**
        - Modules access each other's implementation (friends)
    - I can achieve perfect coupling by putting everything in one class, so there must be a balancing force
- **Cohesian**
    - How closely are the elements of a module related to each other
    - **Low:**
        - Arbitrary grouping (eg. `<utility>`)
        - Common theme, otherwise unrelated, maybe some common base code (eg. `<algorithm>`)
        - Elements manipulate state over the lifetime of an object
            - Ex. open/read/close files
        - Elements pass data to each other
    - **High:**
        - elements operate to perform exactly one task
    - I can achieve perfect cohesian by putting each method into its own class, but then they'll all depend on each other (high coupling).

- **High coupling**
    - Changes to a module affect other modules
    - Harder to reuse individual modules
    - Ex. `function whatIsIt(dynamic_cast)` tightly coupled to the `Book` hierachy, must change this function if you create another `Book` subclass
- **Low Cohesian**
    - Poor organized code, hard to understand and maintain
- We want high cohesian and low coupling

## SOLID principles of OO Design
- **Single Responsibility Principle (SRP)**
    - A class should only have one reason to change
    - i.e. a class should do one thing, not several
    - Any change to the problem spec requires a change to the program
    - If changes to >= 2 different parts of the spec cause changes to the same class, SRP is violated
    - Ex. Don't let you (main) classes print things
        - Consider: 
        ```C++
        class ChessBoard {
            std::cout << "Your move"
        };
        ```
        - Bad design, inhibits code reuse
        - What if you want a version of your program that:
            - Communicates over different streams (file/network)
            - Works in another language?
            - Uses graphics instead of text?
        - Major changes to `ChessBoard` class, instead of reuse
        - Violates SRP, must change the class if there is any specification for:
            - Game rules
            - Strategy
            - Interface
            - Etc.
        - Low cohesian
        - Split these up
        - One modules (not main! can't reuse main) responsible for comunnication
            - If a class wants to say something, do it via parameters/results/exceptions
            - Pass info to communications object and let it do the talking
        - On the other hand - specifications that are unlikely to change may not need their own class - avoid needless complexity
            - Judgement call
- **Open/Closed Principle (OCP)**
    - Classes/modules/functions/etc. should be open for extension + closed for modification
    - Changes in a program's behaviour should happen by writing new code, not by changing old code
    - Ex.

        ```
        +-----------+     +---------+
        | Carpenter |◆--->| Handsaw |
        +-----------+     +---------+

        ```
        - What if a carpenter buys a table saw
        - This design is not open for extension (must change carpenter code)

        ```
        +-----------+     +-----+
        | Carpenter |◆--->| Saw |
        +-----------+     +-----+
                             ^
                             |
                    +--------+----------+
                    |                   |
               +---------+        +-----------+
               | Handsaw |        | Table Saw |
               +---------+        +-----------+
               
        ```
    - Also note: `countHeavy` function:
        ```C++
        int countHeavy(const vector<Book *> &v) {
            int count = 0;
            for (auto &p: v)
                if (p->heavy()) ++ count;
            return count;
        }
        ```
    - vs. `whatIsIt`, when we used dynamic casting (not closed for modiciation)
    - **Note:** can't really be 100% closed, some changes may require source modification
        - Plan for the _most likely_ changes and make your code closed with respect to those changes
- **Liskov Substitution Principle (LSP)**
    - Simply put: public inheritance must indicate an "IS-A" relationship
    - But there's more to it: If `B` is a subtype (subclass) of `A`, then we should be able to use an object `b` of type `B` in any context that requires an object of type `A` _without affecting the correctness of the program (*)_
    - C++'s inheritance rules already allow us to use subclass objects in place of superclass objects
    - (\*) Very important point: a program should "not be able to tell" if it is using a superclass object or a subclass object 
        - Formally: if an invariant `I` is true of class `A`, then it is true of class `B`
        - If an invariant `I` is true of method `A::f`, and `B::f` overrides `A::f`, then `I` must hold for `B::f`
        - If `A::f` has a precondition `P` and a postcondition `Q`, then `B::f` must have a precondition `P' <= P` and a post condition `Q' => Q`
        - If `A::f` and `B::f` behave differently, the difference in behaviour must fall within what is allowed by the program's correctness specification
    - Ex.
        1. **Contravariance Problem**
            - Arises anytime you have a binary operator (ie. a method with an "other" parameter) of the same type as `*this`
                ```C++
                class Shape {
                    public:
                        virtual bool operator==(const Shape &other) const;
                        ...
                };
                class Circle: public Shape {
                    public:
                        bool operator==(const Circle &other) const override; // (*)
                }
                ```
            - (\*) violates the Liskov Substitution
                - A `Circle` _is_ a `Shape`
                - A `Shape` can be compared with _any_ other `Shape`
                - Therefore a `Circle` can be compared with _any_ other `Shape`
                - C++ will flag this problem with a compiler error
                - **FIX:**
                    ```C++
                    bool Circle::operator==(const Shape &other) const {
                        if (typeid(other) != typeid(Circle)) return false;
                        const Circle &other = static_cast<const Circle &>(other);
                        // Compare fields of other with fields of *this;
                    }
                    ```
                - `dynamic_cast` vs `typeid`
                    - `dynamic_cast<Circle &>(other);`: is `other` a `Circle` or a subclass of `Circle`
                    - `typeid(other) == typeid(Circle)`: is `other` percisely a `Circle`?
                    - `typeid` returns an object of type `typeinfo`
        1. Is a square a rectangle?
            - A square has all the properties of a rectangle
                ```C++
                class Rectangle {
                    private:
                        int length, width;
                    public:
                        Rectangle(___);
                        int getLength() const;
                        virtual void setLength(int);
                        int getWidth() const;
                        virtual void setWidth(int);
                        int area() const { return length * width; }
                };
                ```
                ```C++
                class Square: public Rectangle {
                    public:
                        Square(int side): Rectangle(side, side) {}
                        void setLegnth(int l) override {
                            Rectangle::setLength(l);
                            Rectangle::setWidth(l);
                            // length == width in a square
                        }
                        void setWidth(int w) { ... } // Similar
                };
                ```
                ```C++
                int f(Rectangle &r) {
                    r.setLength(10);
                    r.setWidth(20);
                    return r.area();  // Expect 200
                }
                ```
                ```C++
                Square s{1};
                f(s);   // 400
                ```
            - `Rectangle`s have the property that their length and width can vary independently; `Square`s do not. So this violates LSP
            - On the other hand, an immutable `Square` could substitute for an immutable `Rectangle`
            - What can be done:
            <pre>
                                                  +-------+
                                                  | Shape |
                                                  +-------+
                                                      ^
                                                      |
                                                      |
                                    +-----------------+------------- ...
                                    |                           
                +--------------------------+
                | <em>RightAngledQuadrilateral</em> |
                +--------------------------+
                                     ^
                                     |
                            +--------+----------+
                            |                   |
                       +-----------+       +--------+
                       | Rectangle |       | Square |
                       +-----------+       +--------+

            </pre>
        - Constraining what subclasses can do:
            ```C++
            class Turtle {
                public:
                    virtual void draw() = 0;
            };
            ```
            ```C++
            class RedTurtle: public Turtle {
                public:
                    void draw() override {
                        drawHead();
                        drawRedShell();
                        drawFeet();
                    }
            };
            ```
            ```C++
            class GreenTurtle: public Turtle {
                public:
                    void draw() override {
                        drawHead();
                        drawGreenShell();
                        drawFeet();
                    }
            };    
            ```
        - Code duplication!
        - How can we ensure that overrides of `draw()` will always do these things?
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
                    virtual void drawShell() = 0;
                    void drawFeet();
            };
            ```
            ```C++
            class RedTurtle: public Turtle {
                void drawShell() override { ... };
            };
            ```
            ```C++
            class GreenTurtle: public Turtle {
                void drawShell() override { ... };
            };
            ```
        - Subclasses cannot control the steps of drawing a turtle, not the drawing of head + feet
        - Can only control the drawing of a shell (called the **Template Method Pattern**)
    - Extension: **Non-Virtual Interface (NVI) Idiom**
        - `public virtual` methods are simultaneously:
            - Part of a class' interface
                - Pre/post conditions
                - Respect invariants
            - "Hooks" for customizations by subclasses, overriding code could be anything
        - All virtual methods should be private
        - All public methods should be non-virtual
        - Ex. Non-NVI class
            ```C++
            class DigitalMedia {
                public:
                    virtual void play() = 0;
            };
            ```
        - Now with NVI
            ```C++
            class DigitalMedia {
                public:
                    void play() {
                        doPlay();
                    }
                private:
                    virtual void doPlay() = 0;
            };
            ```
            - In the future, can add before/after code
                - Ex. call `checkCopyright()` before, call `updatePlayCount()` afterwards
        - Generalizes the Template Method Pattern
        - Puts every virtual method function inside a template method
- **Interface Segregation Principle**
    - Many small interfaces is better than one large interface
    - If a class has many functionalities, each client of the class should only see the functionality that it needs
    - Ex. Video Game (will ignore NVI to keep example short)
        ```C++
        class Enemy {
            public:
                virtual void strike();  // Needed by game logic
                virtual void draw();    // Needed by UI'
        };
        ``` 
        ```C++
        class UI {
            vector<Enemy *> v;
        };
        ```
        ```C++
        class Battlefield {
            vector<Enemy *> v;
        };  
        ```
        - If we need to change the drawing interface, `Battlefield` must recompile for no reason
        - Creates needless coupling between `UI` and `Battlefield`
        - One solution: **Multiple Inheritance**
        ```C++
        class Enemy: public Draw, public Combat {};
        ```
        ```C++
        class Draw {
            public:
                virtual void draw() = 0;
        };
        ```
        ```C++
        class UI {
            vector<Draw *> v;
        };
        ```
        ```C++
        class Combat {
            public:
                virtual void strike() = 0;
        }
        ```
        ```C++
        class Battlefield {
            vector<Combat *> v;
        }
        ```
        - Example of the **Adapter Pattern**
    - General use of the Adapter Pattern: when a class provides an interface different from the one you need
    - Ex.

        ```
        +------------------+        +----------------+
        | Needed Interface |        | Provided Class |
        +------------------+        +----------------+
        |+ g()             |        |+ f()           |
        +------------------+        +----------------+
                 ^                          ^   
                 |                          |   // This inheritance could be private, 
                 +-------------+------------+   // depends on whether you want the adapter 
                               |                // to still support the old interface
                           +---------+
                           | Adapter |
                           +---------+    +----------+
                           |+ g() ---|----| { f(); } |  // Show's implementation
                           +---------+    +----------+

        ```
    - _Detour:_ Issues with multiple inheritance

        ```
        +------+        +------+    
        |  A1  |        |  A2  |
        +------+        +------+
        |+ a() |        |+ a() |
        +------+        +------+
            ^               ^
            |               |
            +-------+-------+
                    |
                 +-----+
                 |  B  |    // Has 2 a() methods
                 +-----+
        ```

        ```
                 +-----+
                 |  A  |
                 +-----+
                 |+ a()|
                 +-----+
                    ^
                    |
           +--------+--------+
           |                 |
        +-----+           +-----+           
        |  B  |           |  C  |
        +-----+           +-----+
           |                 |
           +--------+--------+
                    |
                 +-----+        
                 |  D  |    // Has two a() methods, and they're different
                 +-----+        
        ```
        ```C++
        class D: public B, public C {
            void f() { ... a() ... }  // Ambiguous, use B::a() or C::a()
        };
        D d;    
        d.a();  // Ambiguous, use d.B::a() or d.C::a()
        ```
    - OR maybe there should be only one `A` base, and therefore only one `a()`
        ```C++
        class B: virtual public A { ... };
        class C: virtual public A { ... };
        ```
    - Now `d.a()` is no longer ambiguous
    - Ex. iostream hiearchy

        ```
                            ios_base
                               |
                               |
                       .------ios-----.
                      /                \
                     /                  \
                    /                    \
                   /                      \
               istream -------------.    ostream----ofstream
              /       \              \    /    \ 
             /         \              \  /      ostringstream
            /           \              iostream
           /             \                |    \
          /               \               |     \
         /                 \              |      \
     ifstream       iostringstream      fstream  stringstream    
         ```
    - _Problem:_ How will a class like `D` be laid out in memory (implementation specific)
        - Consider:
        ```C++
        +----------+    <-- Should look like an A*, B*, C*, D*
        |   vptr   |      
        +----------+
        | A fields |
        +----------+
        | B fields |
        +----------+
        | C fields |
        +----------+
        | D fields |
        +----------+          
        ```
    - What does `g++` do?
        ```C++
        +----------+
        |   vptr   |
        +----------+
        | B fields |
        +----------+
        |   vptr   |
        +----------+
        | C fields |
        +----------+
        |   vptr   |
        +----------+
        | D fields |
        +----------+
        |   vptr   |
        +----------+
        | A fields |
        +----------+
        ```
    - `B` and `C` need to be laid out so that we can find the `A` partm but the distance is not known (depends on the runtime of the object)
    - _Solution:_ location of the base object stored in vtable
        - Also note the diagram doesn't simultaneously look like `A`, `B`, `C`, `D`, but slices of it do
        - Therefore pointer assignment among `A`, `B`, `C`, `D` pointers may change the address stored in the pointer
        ```C++
        D *d = ___;
        A *a = d;   // Changes the address
        ```
        - `static_cast`, `const_cast`, `dynamic_cast`, under multiple inheritance will also adjust the value of the pointer (`reinterpret_cast` will not)
- **Dependency Inversion Principle**
    - High level modules should not depend on low-level modules. Both should depend on abstractions
    - Abstract classes should never depend on concrete classes
    - Traditional top-down design
        - High level modules _uses_ low level module
        - Ex. `Word count` _uses_ `keyboard reader`
        - What if I want to use a file reader?
        - Changes to details affect the higher level word count module
        - Dependency inversion
        <pre>
        +-------------------+   +--------------------------------+
        | High Level Module |-->|      <em>Low Level Abstraction</em>     | 
        + ------------------+   +--------------------------------+
                                                ^
                                                |
                                        +-------------------+
                                        | Low Level Modules |
                                        +-------------------+
        +------------------+         +--------------------+
        |    WordCount     |-------->|  <em>Input Interface</em>   |
        +------------------+         +--------------------+
                                                ^
                                                |
                                         +------+------+
                                         |             |
                                +------------+      +----------+
                                |  Keyboard  |      |   File   |
                                |   Reader   |      |  Reader  |
                                +------------+      +----------+
        </pre>
        - Ex.
            ``` 
            +-------+    +-----------+
            | Timer |◇-->| Bell      |
            +-------+    +-----------+
                         |+ notify() |
                         +-----------+
            ```
            - When the timer hits some specified time, it rings the `Bell` (calls `Bell:notify`, which rings the bell)
            - What if we want to trigger other events? Maybe more than one:
            <pre>
            +-------+  * +-----------+
            | Timer |◇-->| <em>Responder</em> |
            +-------+    +-----------+
                         |+ notify() |
                         +-----------+
                               ^
                               |
                        +------+-------+
                        |              |
                  +-----------+   +-----------+     
                  | Bell      |   | Lights    |     
                  +-----------+   +-----------+     
                  |+ notify() |   |+ notify() |     
                  +-----------+   +-----------+
            </pre>
            - Maybe we want a dynamic set of responders
            <pre>
            +-----------------------+        *
            | Timer                 |◇--------->+-----------+
            +-----------------------+           | <em>Responder</em> |
            |+ register(Responder)  |&lt;---------◇+-----------+
            |+ unregister(Resonder) |           |+ notify() |
            +-----------------------+           +-----------+
                                                       ^    
                                                       |      
                                                +------+-------+
                                                |              |
                                          +-----------+   +-----------+
                                          | Bell      |   | Lights    |
                                          +-----------+   +-----------+
                                          |+ notify() |   |+ notify() |
                                          +-----------+   +-----------+
            </pre>
            - Now _`Responder`_ is depending on the concrete `Timer` class: apply Dependency Inversion again
            <pre>
            +------------------------+      +-----------------+ 
            | <em>Source</em>                 |⬦---->| <em>Responder</em>       |
            +------------------------+      +-----------------+ 
            |+ register(Responder)   |      |+ notify()       |
            |+ unregister(Responder) |      +-----------------+
            +------------------------+               ^
                    ^                     +----------+
                    |                     |          |
                +-------+             +------+     +-------+
                | Timer |&lt;-----------⬦| Bell |     | Light |
                +-------+             +------+     +-------+
                    ^                                   ⬦
                    |                                   |
                    +-----------------------------------+
           
            </pre>
            - Could dependency invert this again if you wanted
            - **General Solution:** known as the Observer Pattern
            <pre>
            +-----------------+             +-----------------+ 
            | <em>Subject</em>         |⬦----------->| <em>Observer</em>        |
            +-----------------+             +-----------------+ 
            |+ notifyObservers|             |+ notify()       |
            |+ attatch(Obs)   |             +-----------------+ 
            |+ detatch(Obs)   |                      ^
            +----------------+                       |
                    ^                                |
                    |                                |
            +-----------------+             +-------------------+
            | ConcreteSubject |&lt;-----------⬦| Concrete Observer |
            +-----------------+             +-------------------+
            |+ getState()     |             |+ notify()         |
            +-----------------+             +-------------------+
            </pre>
            - Sequence of calls:
                1. `Subject`'s state changes
                1. `Subject::notifyObservers` (either by the `Subject` itself OR by some external controller)
                    - Calls each `Observer`'s `notify` 
                1. Each `Observer` calls `concreteSubject::getState` to query the state + react accordingly

## Some More Design Patterns

### Factory Method Pattern

When you don't know exactly what kind of object you want, and your preferences may vary
- Also called the **Virtual Constructor Pattern**
- Strategy pattern applied to object construction

Ex. 
<pre>
                +-------+
                | <em>Enemy</em> |
                +-------+
                    ^
                    |
         +----------+------------+
         |                       |
     +--------+             +--------+
     | Turtle |             | Bullet |
     +--------+             +--------+

                +-------+
                | <em>Level</em> |
                +-------+
                    ^
                    |
         +----------+------------+
         |                       |
      +------+               +------+
      | Easy |               | Hard |
      +------+               +------+    
</pre>

- Randomly generated
- More turtles in easy levels
- More bullets in hard levels

```C++
class Level {
    public:
        virtual Enemy *getEnemy() = 0;
};
```

```C++
class Easy: public Level {
    public:
        Enemy *getEnemy() override {
            // mostly turtles
        }
};
```

```C++
class Hard: public Level {
    public:
        Enemy *getEnemy() override {
            // mostly bullets
        }
};
```

```C++
Level *l = new Easy;
Enemy *e = l->getEnemy();
```

### Decorator Pattern
Add/remove functionality to/from objects at runtime

Ex. add menus/scrollbars to windows - either or both without a combinatorial explosion of subclasses

<pre>
                      +-----------+
                      | <em>Component</em> |<------------+
                      +-----------+             |
                            ^                   |
                            |                   |
          +----------------------------+        |
          |                            |        |
          | // Plain window            |        |
    +--------------------+       +-----------+  |
    | Concrete Component |       | <em>Decorator</em> |◇-+
    +--------------------+       +-----------+
    |+ operation         |             ^
    +--------------------+             |
                                       +------------------------------------+
                                       |                                    |
                            +----------------------+            +----------------------+
// (Window w/ scrollbar)    | Concrete Decarator A |            | Concrete Decarator B | // Window w/ menu
                            +----------------------+            +----------------------+
                            | operation            |            | operation            |
                            +----------------------+            +----------------------+
</pre>

Every `Decorator` IS a component AND HAS a `Component`
- `Window w/ scrollbar` is a kind of window, _and_ has a pointer to the underlying plain window
- `Window w/ scrollbar + menu` is a window and has a pointer to a `window w/ scrollbar`, which has a pointer to a `plain window`  

Ex.

```C++
WindowInterface *w = new WindowWithMenu{
                            new WindowWithScollBar{
                                new Window{}}};
```

### Vistor Pattern

For implementing _double dispatch_
- Method chosen based on the runtime types of 2 objects, not just one

```C++
class Enemy {
    public:
    virtual void beStruckBy(Weapon &w) = 0;
};

class Turtle: public Enemy {
    public:
    void beStruckBy(Weapon &w) override {w.strike(*this);}
};

class Bullet: public Enemy {
    public:
    void beStruckBy(Weapon &w) override {w.strike(*this);}
};

class Weapon {
    public:
    virtual void strike(Turtle &t) = 0;
    virtual void strike(Bullet &b) = 0;
};

class Stick: public Weapon {
    public:
    void strike(Turtle &t) override {
        //strike turtle with stick
    }

    void strike(Bullet &b) override {
        //strike bullet with stick
    }
}
```

```C++
Enemy *e = new Bullet{...};
Weapon *w = new Rock{...};
e->beStruckBy(*w); 
```

What happens?
- `Bullet::beStruckBy` runs (virtual method dispatch)
- Which calls `Weapon::strike(Bullet &b)` since `*this` is a Bullet
- This fact is known at compile time, overload resolution
- Virtual method resolves to `Rock::Strike(Bullet &)`

Visitor can also be used to add functionality to a class hierarchy without adding new virtual methods.

Add a visitor to the book hierarchy:
```C++
class Book {
    ...
    public:
    ...
    virtual void accept(BookVisitor &v) {v.visit(*this);}
};

class Text: public Book {
    ...
    public:
    void accept(BookVisitor &v) override {
        v.visit(*this);
    }
};

class BookVisitor {
    public:
    virtual void visit(Book &b) = 0;
    virtual void visit(Text &t) = 0;
    virtual void visit(Comic &c) = 0;
};
```

Example: Categorize and Count.
- For `Book`s: - by Author
- `Text`s: - by Topic
- `Comic`s: - by Hero

Could do this with a virtual method, or write a visitor.
```c++
class Catalog: public BookVisitor {
    public:
    map<string, int> theCat;
    void visit(Book &b) override {++theCat[b.getAuthor()];}
    void visit(Text &t) override {++theCat[b.getTopic()];}
    void visit(Comic &c) override {++theCat[c.getHero()];}
};
```

### But it won't compile!
- Circular include dependency
- _book.h_, _BookVisitor.h_ include each other.
- include guard prevents multiple inclusion
- whichever ends up occurring first will refer to things not yet defined.
- needless includes create artificial compilation dependencies, and slow down compilation, or prevent compilation altogether.

Sometimes a forward class declaration is not good enough.

Consider:
```c++
class A {...}; // A.h

class B {
    A a;
};

class C {
    A *a;
};

class D: public A {
    ...
};

class E {
    A f(A);
};

class F {
    A f(A a) {a. someMethod();}
};

class G {
    t<A> x;
};
```
Which need includes? `B`,`D`,`F` need includes

`C`,`E` forward declare ok.

`G` - it depends on how the template t uses A.
- should collapse to one of the other cases.

Note: class `F` only needs an include because method `f`'s implementation is present.
- a good resason to keep implementation in .cc
- where possible: forward declare in .h, include in .cc

Also notice: `B` needs an include; `C` does not.
- If we want to break the compilation dependency of `B` on `A`, we could make `B` like `C`.

More generally:
```C++
class A1{}; class A2{}; class A3{};

class B {
    A1 a1;
    A2 a2;
    A3 a3;
};
```

_b.h:_
```C++
class BImpl;

class B {
    unique_ptr<BImpl> pImpl;
};
```

_bimpl.h:_
```C++
#include "a1.h"
...

struct BImpl {
    A1 a1;
    A2 a2;
    A3 a3;
};
```

_b.cc:_
```C++
#include "b.h"
#include "bimpl.h"

methods reference pImpl -> a1, a2, a3
```

_b.h_ no longer compilation-dependent on _a1.h_, etc.
- called the pImpl idiom

Another advantage of pImpl - pointers have a non-throwing swap.

Can provide the strong guarantee on a B method by
- Copying the Impl into a new BImpl structure (heap-allocated)
- Method modifies the copy
- If anything throws, discard the new structure (easy and automatic with `unique_ptr`s)
- If all succeeds swap impl structs (pointer swap - `nothrow`)
- Previous impl automcatically destroyed by the smart pointer.

Example:
```C++
class B {
    unique_ptr<BImpl> pImpl;
    ...
    void f() {
        auto temp = make_unique<BImpl>(*pImpl);
        temp->doSomething();
        temp->doSomethingElse();
        std::swap(pImpl,temp); // nothrow
    } // strong guarantee
};
```

---
[I want to know what kind of Book I have << ](./problem_22.md) | [**Home**](../README.md) | [>> Shared Ownership](./problem_23.md)