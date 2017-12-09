[Tampering << ](./problem_7.md) | [**Home**](../README.md) | [>> Staying in bounds](./problem_9.md) 

# Problem 8: Efficient Iteration
**2017-09-28**

Consider the two implementations Vector and List

```C++
vector v;
v.push_back(___);
...

for (size_t i = 0; i < v.size(); ++i) {
    std::cout << v[i] << std::endl;     // O(1)
}
```

- Array access - efficient
- O(n) traversal

```C++
list l;
l.push_front(___);
...

for (size_t i = 0; i < l.size(); ++i) {
    std::cout << l[i] << std::endl;     // O(n)
}
```

- O(n^2) traversal
- No direct access to "next" pointers, how can we do efficient iteration?

- **Design Patterns**
    - Well known solutions to well-studied problems
    - Adapted to suit needs
- **Iterator Pattern**
    - Efficient iteration over a collection, without exposing the underlying structure

**Idea:** Create a class that "remembers" where you are in the list (abstraction of a pointer)  
**Inspiration:** C

```C
for (int *p = arr; p != arr + size; ++p) {
    printf("%d\n", *p);
}
```

```C++
class list {
    struct Node {...};
    Node *theList;

    public:
        class iterator {
            Node *p;

            public:
                iterator(Node *p): p{p} {}
                bool operator!=(const iterator &other) const {return p != other.p}
                int &operator*() {return p->data;}
                iterator &operator++() {    // Prefix version
                    p = p->next;
                    return *this;
                }
        }

        iterator begin() {return iterator{theList};}
        iterator end() {return iterator{nullptr};}
};
```
```C++
list l;

for (list::iterator it = l.begin(); it != l.end(); ++it) {
    std::cout << *it << '\n';
}
```

**Q:** Should `list::begin` and `list::end` be `const` methods?  
**Consider:**

```C++
ostream &operator<<(ostream &out, const list &l) {
    for (list::iterator it = l.begin(); it != l.end(); ++it) {
        ...
    }
    ...
}

``` 

Won't compile if `begin`/`end` are not `const`

```C++
ostream &operator<<(ostream &out, const list&l) {
    for (...) {
        out << *it << '\n';
        ++*it;  // increment items in the list
    }
}
```

Will compile but shouldn't, the list is supposed to be `const`, but `*` returns as non-`const`

**Conclusion:** iteration over `const` is different from iteration over non-`const`
- Make a second iterator class

```C++
class list {
    struct Node {...};
    Node *theList;

    public:
        class iterator {
            Node *p;

            public:
                iterator(Node *p): p{p} {}
                bool operator!=(const iterator &other) const { return p != other.p; }
                int &operator*() const { return p->data; }
                iterator &operator++() {    // Prefix version
                    p = p->next;
                    return *this;
                }
        };

        class const_iterator {
            Node *p;

            public:
                const_iterator(Node *p): p{p} {}
                bool operator!=(const const_iterator &other) { return p != other.p; }
                const int &operator*() const { return p->data; }
                const_iterator &operator++() {
                    p = p->next;
                    return *this;
                }
        };

        iterator begin() { return iterator{the_list}; }
        iterator end() { return iterator{nullptr}; }
        const_iterator begin() const { return const_iterator{theList}; }
        const_iterator end() const { return const_iterator {nullptr}; }
};
```

Works now:

```C++
list::const_iterator it = l.begin();    // Mouthful
```

Shorter:
```C++
ostream &operator<<(...) {
    for (auto it = l.begin(); it != l.end(); ++it) {
        out << *it << '\n';
    }

    return out;
}
```

Even shorter:

```C++
ostream &operator<<(___) {
    for (auto n : l) { 
        out << n << '\n';   
    }

    return out;
}
```

This is a range-based `for` loop
- Available for any class with:
    - Methods (or functions) `begin()` and `end()` that return an iterator object
    - The iterator class must support unary`*`, prefix`++`, and `!=`

**Note:**
- `for (auto n: l) ++n;`
    - `n` is declared by value
    - `++n` increments n, not the list items
- `for (auto &n : l) ++n;`
    -  `n` is a reference, will update list elements
- `for (const auto &n : l) ____;`
    - `const` reference, cannot be mutated

One small encapsulation problem:

**Client:** `list::iterator it {nullptr}`
- Forgery, create an end iterator without calling `end();`


**To fix:** make iterator constructor private  
**BUT:** List can't create iterators either  
**Solution:** _friendship_ <3

```C++
class list {
    ...
    public:
        class iterator {
            ...
            iterator(Node *p) {}
           
            public:
            ...
            friend class list;  // list has access to all iterator's/const_iterator's implementation
        };

        class const_iterator {  // Same (friend class list)
            ...
        }
};
```

Recall: Encapsulation + Iterators for linked list

Can do the same for vectors:

```C++
class vector {
    size_t size, cap;
    int *theVector;
    
    public:
        class iterator {
            int *p;
            ...
        };

        class const_iterator {
            ...
        };

        iterator begin() {
            return iterator{theVector};
        }

        iterator end() {
            return iterator{theVector + n};
        }

        const_iterator begin() const {
            return const_iterator{theVector};
        }

        const_iterator end() const {
            return const_iterator{theVector + n};
        }
};
```

Limit friendships, they weaken encapsulation

Could do this, OR:

```C++
typedef int *iterator;
typedef const int *const_iterator;

---

using iterator = int*;
using const_iterator = const int*;

iterator begin() {return theVector;}
iterator end() {return theVector + n;}
```

---
[Tampering << ](./problem_7.md) | [**Home**](../README.md) | [>> Staying in bounds](./problem_9.md) 
