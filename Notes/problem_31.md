[<< Total Control](./problem_30.md) | [**Home**](../README.md) | [>> A fixed-size object allocator](./problem_32.md) 

# Problem 31: I want total control over vectors and lists
**2017-11-29**

Incoporating custom allocation into out containers.

**Issue:** may want different allocators for different kinds of vectors.

**Solution:** Make the allocator an argument to the template.

Since most users won't write allocators, we'll need a  default value.

**Template Signature:**
```C++
template<typename T, typename Alloc = allocator<T>> class vector { ... }
```

Now write the interface for allocators and the allocator template:

```C++
template<typename T> struct allocator {
    using value_type = T;
    using pointer = T*;
    using reference = T&;
    using const_pointer = const T*;
    using const_reference = const T&;
    using size_type = size_t;
    using difference_type = ptrdiff_t; 

    allocator() noexcept {} // Trivial - no data members
    allocator(const allocator &) noexcept {}
    template<typename U> allocator(const allocator<U> &) noexcept {} 
    ~allocator() {}

    pointer address(reference x) const noexcept { return &x; }
    pointer allocate(size_type n) { return ::operator new(n * sizeof(T)); }
    pointer deallocate(pointer p, size_type n) { ::operator delete(p)); }
    size_type max_size() const noexcept {
        return std::numeric_traits<size_type>::max(sizeof(value_type));
    }
    template<typename U, typename ...Args>
    void construct(U *p, Args &&... args) {
        ::new(static_cast<void*>(p)) U(forward<Args>(args)...);
    }
    template<typename U> void destroy(U *p) { p->~U(); }
};
```

If you want to write an allocator for an STL container, this is its interface.

**Note:** `operator new` takes a number of _bytes_, but `allocate` takes a number of _objects_.

What happens if a vector is copied? Copy the allocator? What happens if you copy an allocator? Can 2 copies allocate/deallocate each other's memory?

C++03 allocators must be stateless - copying allocators is allowed and trivial
C++11 allocators can have state, can specify copying behaviour via allocator traits

Adapting `vector`:
- `vector` has a field `Aloc alloc`;
- everywhere `vector` calls 
    - `operator new`, replace with `alloc.allocate`
    - `placement new`, replace with `alloc.construct`
    - `dtor` explicitly, replace with `alloc.destroy`
    - `operator delete`, replace with `alloc.deallocate`
    - takes an address, replace with `alloc.address`
- Details: exercise

Can we do the same with list?
```C++
template<typename T, template Alloc = allocator<T>> class list { ... }
```

Correct so far... but curiously, `Alloc` will never be used to allocate memory in a list.

Why not? lists are node-based, which means you don't want to actually allocate `T` objects; you want to allocate nodes (which contains `T` objects and pointers.)
- but `Alloc` allocates `T` objects

How do we get an allocator for nodes?
- Every conforming allocator has a member template called `rebind` that gives the allocator type for another type:

```C++
template <typename T> struct allocator {
    template <typename U> struct rebind {
        using other = allocator<U>;
    }; 
};
```

Within `list` - to create an allocator for nodes as a field of list:
```C++
Allocator::rebind<Node>::other alloc;
```

Then use as in vector. Details: exercise

---
[<< Total Control](./problem_30.md) | [**Home**](../README.md) | [>> A fixed-size object allocator](./problem_32.md) 
