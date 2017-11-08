[I want to know what kind of Book I have << ](./problem_22.md) | [**Home**](../README.md)

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
                   ------------------------+---------------------------
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
                        void setWidth(int w) { ... } // Simiar
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
                        drawHead()
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





