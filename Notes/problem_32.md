[<< I want total control over vectors and lists](./problem_31.md) | [**Home**](../README.md) | [>> I want a (tiny bit) smaller vector class](./problem_33.md) 

# Problem 32: A fixed-size object allocator
**2017-11-29**

A custom allocator can be fast.
**Fixed size allocator:** all allocated "chunks" are the same size (ie. customaized code for one class) - no need to keep track of sizes

(aside - many many traditional allocators store the size of the block before the pointer so that the allocator knows how much space is allocated to that pointer.)

**Fixed size:**
- Saves space (no hidden size field)
- Saves time (no hunting for a block of the right size)

**Approach:** create a pool of memory  - an array large enough to hold `n` `T` objects.

When the client has a slot: `T` object

When we have it: node in a linked list

Each slot stores the index of the next slot in the list

```
FIRST
+---+       +---+---+---+---+-----+----+
| 0 |       | 1 | 2 | 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1
```

Allocation: from the front

```
          Allocated
+---+       +---+---+---+---+-----+----+
| 1 |       |///| 2 | 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1

+---+       +---+---+---+---+-----+----+
| 2 |       |///|///| 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1
```

Deallocation: 

```
Free item 0
+---+       +---+---+---+---+-----+----+
| 0 |       | 2 |///| 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1

Free item 1
+---+       +---+---+---+---+-----+----+
| 2 |       | 2 | 0 | 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1
```

_Implementation:_

```C++
template<typename T, int n> class fsAlloc {
    private:
        union Slot {
            int next;
            T data;
            Slot(): next{0} {}
        };

        Slot theSlots[n];
        int first = 0;
    public:
        fsAlloc() {
            for (int i = 0; i < n - 1; ++i) {
                theSlots[i].next = i + 1;
            }
            theSlots[n - 1].next = -1;
        }

        T *allocate() noexcept {
            if (first == -1) return nullptr;
            T *result = &(theSlots[first].data);
            first = theSlots[first].next;
            return result;
        }

        void deallocate(void *item) noexcept {
            int index = (static_cast<char*>(item) - reinterpret_cast<char*>(theSlots)) / sizeof(Slot);
            theSlots[index].next = first;
            first = index;
        }
};
```

Use in a class:

```C++
class Student final {
        int assns, mt, final;     
        static fsAlloc<Student, SIZE> pool; // SIZE - how many you want
    public:
        ...
        static void* operator new(size_t size) {
            if (size != sizeof(Student)) throw std::bad_alloc;
            while (true) {
                void *p = pool.allocate();
                ...
            }
        }

        static void* operator delete(void *p) noexcept {
            if (p == nullptr) return;
            pool.deallocate(p);
        }
};

fsAlloc<Student, SIZE> Student::pool;
```

_Example main:_
```C++
int main() {
    Student *s1 = new Student;  // Uses custom allocator
    Student *s2 = new Student;  // Uses custom allocator
    delete s1;
    delete s2;
}
```

**Q:** Where do `s1` and `s2` reside?
**A:** Static memory (NOT the heap)
- Could arrange for stack/heap memory

**More notes:** 
- We used a union to hold both `int *T` - wastes less space
- We could have used a struct `[ next | T ]`
    - Disadvantage: if you access a dangling `T` pointer, you can corrupt the linked list
    ```C++
    Student *s = new Student;
    delete s;
    s->setAssns(...);
    ```
- Lesson: following a dangling pointer can be VERY dangerous
- With a struct, `next` is before the `T` object, so you have to work hard to corrupt it.
```C++
reinterpret_cast<int *>(s)[-1] = ...
```

On the other hand - with a struct - problems if `T` doesn't have a default constructor

```C++
struct Slot {
    int next;
    T data;
};

Slot theSlots[n];   // X - if T has no default constructor
```

Can't do `operator new`/`placement new`

```C++
union SlotChar {
    char dummy; // As before
    slot s;
    SlotChar(): dummy{0} {}
};
```

Also:
- Why store indices instead of pointers?
- Smaller than pointers on this machine
- So waste no memory as long as `T` >= size of an `int`
- Would waste if `T` smaller than an `int`
- Could use a smaller index than an `int`
- could use a smaller index type, ex. `short`, `char`
    - (as long as you don't want more items than the type can hold)
- Could make the index type a parameter of the template
- `Student final` - fixed-size allocator - subclasses might be larger
- Options: 
    - Have no subclasses
    - Check size, throw if it isn't the right size
    - Derived class can have its own allocator

---
[<< I want total control over vectors and lists](./problem_31.md) | [**Home**](../README.md) | [>> I want a (tiny bit) smaller vector class](./problem_33.md) 
