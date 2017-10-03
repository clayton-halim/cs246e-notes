# Tampering (cont.)
**2017-09-28**

A similar problem with linked lists:

```C++
Node n {3, nullptr};    // Stack allocated
Node m {4, &n}; // m's dtor will try to delete &n (undefined)
```

There was an invariant that - `next` is `nullptr` or was allocated by `new`

How can we enforce this? 
- Encapsulate Node inside a "wrapper" class

```C++
class List {
    struct Node {    // Private nested class - not available outside
        int data;
        Node *next; // ... methods
    };

    Node *theList;
    
    public:
        List(): theList{nullptr} {}
        ~list() {delete theList;}
        size_t size() const;

        void push_front(int n) {
            theList = new Node{n, theList};
        }

        void pop_font() {
            if (theList) {
                Node *tmp = theList;
                theList = theList->next;
                tmp->next = nullptr;
                delete tmp;
            }
        }

        const int &operator[](size_t i) const {
            Node *cur = theList;
            for (size_t j = 0; j < i && cur; ++j, cur=cur->next);
            return curr->daata;
        }

        int &operator[](size_t i) {
            Node *cur = theList;
            for (size_t j = 0; j < i && cur; ++j, cur=cur->next);
            return curr->data;
        }   
};
```
Client cannot manipulate the list directly
- No access to next pointers
- Invariant is maintained

## Problem 8: Efficient Iteration

Consider the two implementations Vector and List

```C++
Vector v;
v.push_back(___);
...

for (size_t i = 0; i < v.size(); ++i) {
    std::cout << v[i] << std::endl;     // O(1)
}
```

- Array access - efficient
- O(n) traversal

```C++
List l;
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
class List {
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

List l;

for (list::iterator it = l.begin(); it != l.end(); ++it) {
    std::cout << *it << '\n';
}
```

**Q:** Should List::being and List::end be `const` methods?  
**Consider:**

```C++
ostream &operator<<(ostream &out, const List &l) {
    for (List::iterator it = l.begin(); it != l.end(); ++it) {
        ...
    }
    ...
}

``` 

Won't compile if `begin`/`end` are not `const`

```C++
ostream &operator<<(ostream &out, const List&l) {
    for (...) {
        out << *it << '\n';
        ++*it;  // increment items in the list
    }
}
```

Will compile but shouldn't, the list is supposed to be `const`, but it does because `*` returns as non-`const`

**Conclusion:** iteration over `const` is different from iteratoin over non-`const`
- Make a second iterator class

```C++
class List {
    struct Node {...};
    Node *theList;

    public:
        class iterator {
            Node *p;

            public:
                iterator(Node *p): p{p} {}
                bool operator!=(const iterator &other) const {return p != other.p}
                int &operator*() const {return p->data;}
                iterator &operator++() {    // Prefix version
                    p = p->next;
                    return *this;
                }
        };

        class const_iterator {
            Node *p;

            public:
                const_iterator(Node *p): p{p} {}
                bool operator!=(const const_iterator &other) {
                    return p != other.p;
                }

                const_iterator &operator++() {
                    p = p->next;
                    return *this;
                }

                const int &operator*() const {
                    return p->data;
                }
        };

        iterator begin() {return iterator{the_list};}
        iterator end() {return iterator{nullptr};}
        const_iterator begin() const {return const_iterator{theList};}
        const_iterator end() const {return const_iterator {nullptr};}
};
```

Works now:

```C++
List::const_iterator it = l.begin();    // Mouthful
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

This is a range based for loop
- Available for any class with
    - Methods (or functions) begin and end that return an iterator object
    - The iterator class must support unary`*`, prefix`++`, `!=`

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
**Solution:** _friendship_

```C++
class List {
    ...
    public:
        class iterator {
            ...
            iterator(Node *p) {}
           
            public:
            ...
            friend class List;  // List has access to all iterator's/const_iterator's implementation
        };

        class const_iterator {  // Same (friend class List)
            ...
        }
};
```

Limit friendships, they weaken encapsulation