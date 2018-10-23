[Is vector exception safe? << ](./problem_15.md) | [**Home**](../README.md) | [>> Abstraction over containers?](./problem_17.md) 

# Problem 16: Insert/Remove in the Middle
**2017-10-19**

A method like `vector<T>::insert(size_t i, const T &x)` is easy to write.  
But for the same `list<T>` requires an upfront traversal.

Using iterators can be good for both.

```C++
template<typename T> class vector {
    ...
    public:
        iterator insert(iterator posn, const T&x) {
            increaseCap();
            ptrdiff_t offset = posn - begin(); // ptrdiff_t incase result is negative (in general)
            iterator newPosn = begin() + offset;
            new(static_cast<void*>(end()) T(std::move(*(end() - 1)));
            ++vb.n;
            for (iterator it = end() - 1; it != Posn; --it) {
                *it = std::move(*(it - 1));
            }
            *newPosn = x;
            return newPosn;
        }
    }
};
```

Exception safe? Assuming `T`'s copy/move operations are exception safe (at least basic guarantee), insert offers the basic guarantee.
- May get a partially shuffled vector, but it will be a valid vector.

**Note:** if you have other iterators pointing at the vector

```
+---+---+---+---+---+  
| 1 | 2 | 3 | 4 |...|  
+---+---+---+---+---+  
  ^       ^   ^    
 it1      h  it2  

and you insert at h  

+---+---+---+---+---+---+  
| 1 | 2 | 5 | 3 | 4 |...|  
+---+---+---+---+---+---+  
  ^       ^   ^    
 it1      h  it2  
```

it2 will now point at a different item.

**Convention:** after a call to insert or erase all iterators pointing after the point of insertion/erasure are considered invalid and should not be used.
- Also, if a reallocation happens, _all_ iterators pointing into the vector become invalid.

Exercises: 
- **erase** - remove the item pointer to by an iterator, return an iterator to the point of erasure
- **emplace** - like insert but takes constructor args

BUT - that means there is a problem with `push_back`. If `increaseCap` successfully reallocates and placement new (constructor) throws, there vector is the same but the iterators were invalidated!

Exercise: fix this

---
[Is vector exception safe? << ](./problem_13.md) | [**Home**](../README.md) | [>> Abstraction over containers?](./problem_17.md) 
