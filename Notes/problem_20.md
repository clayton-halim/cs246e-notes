[I'm leaking! << ](./problem_17.md) | [**Home**](../README.md) | [>> The copier is broken](./problem_21.md)

# Problem 20 - I want a class with no objects
**2017-10-26**

```C++
class Student {
    public:
        virtual float fees() const;
};

class RegularStudent: public Student {
    public:
        float fees() const override;    // Regular student frees
}

class CoopStudent: public Student {
    public:
        float fees() const override;    // Co-op student frees
}
```

What should `Student::fees` do?

Don't know - every `Student` should either be `RegularStudent` or `CoopStudent`.

**Solution:** explicitly give `Student::fees` no implementation.

```C++
class Student {
    public:
        virtual float fees() const = 0;
};
```

`Student` is an **abstract class**  
`fees()` is a **pure virtual method**

Abstract classes cannot be instantiated:

```C++
Student s;  // ERROR
Student *s = new Student;   // ERROR
```
Can point to instances of **concrete classes** (non-abstract classes):

```C++
Student *s = new RegularStudent;
```

Subclasses of abstract classes are abstract are abstract, unless they implement every pure virtual method in the superclass.

Abstract classes 
- used to organize concrete classes
- can contain common fields and default methods (not need to be overrided)

---
[I'm leaking! << ](./problem_17.md) | [**Home**](../README.md) | [>> The copier is broken](./problem_21.md)