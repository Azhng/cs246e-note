# Problem 31 - I want total control over `vector`s & `list`s


Incorporating custom allocation into our containers 

- Issue:
    - we may want different allocators for different kinds of `vector`s 
- Solution:
    - Make the allocator an argument to the template 
    - since most users won't write allocators, we will need a default value 

Template Signature: 

``` C++
template <typename T, typename Alloc = allocator<T>> 
class vector {/*...*/};
```

Now we write the interface for allocators and the `allocator` template 

``` C++
template <typename T>
struct allocator { 
    using value_type = T;
    using pointer = T*;
    using reference = T&;
    using const_pointer = const T*;
    using const_reference = const T&;
    using size_type = size_t;
    using difference_type = ptrdiff_t;

    allocator() noexcept {}  // <- trivial implementation since we don't have data memebers 

    allocator(const allocator&) noexcept {}

    template <typename U>
    allocator(const allocator<U>&) noexcept {}

    ~allocator() {}

    pointer address(reference x) const noexcept {
        return &x;
    }

    pointer allocate(size_type x) {
        return ::operator new(n * sizeof(T));
    }

    void deallocate(pointer p, size_type n) {
        ::operator delete (p);
    }

    size_type max_size() const noexcept {
        return std::numeric_traits<size_type>::max(sizeof(value_type));
    }

    template <typename U, typename... Args>
    void construct(U *p, Args&&... args) {
        ::new (static_cast<void *>(p))U(forward<Args>(args)...);
    }

    template <typename U>
    void destroy(U* p) {
        p->~U();
    }
};
```

If we want to write an allocator for an STL containers, this is its interface 

- Note: 
    - `operator new` takes a number of *bytes*, but `allocate` takes number of *objects*
    - What happens if a `vector` is copied ? copy the `allocator` ? 
    - What happens if we copy an `allocator`? Can 2 copies of `allocate` / `deallocate` each other's memory 

- C++03 - `allocator`s must be stateless 
- C++11 - `allocator`s can have state, can specify copying behaviour via `allocator` traits, 

To Adapt `vector`:
    - `vector` has a field `Alloc alloc` 
    - everywhere `vector` calls:
        - `operator new` replace it with `alloc.allocate` 
        - `place new` replace it with `alloc.construct` 
        - `dtor` replace with `alloc.destroy`
        - `operator delete` replace with `alloc.deallocate`
        - `taking address` replace with `alloc.address` 


Can we do the same with `list` ? 
- not quite 

``` C++
template <typename T, typename Alloc = allocator<T>>
class list {/*...*/} 
```


this is correct so far, but curiously, `Alloc` will never be used to allocate memory in list
- Why not? 
    - `list` are node-based
        - means we don't want to allocate `T` objects 

How do we get an `allocator` for nodes ? 
- Every conforming `allocator` has a member template called `rebind` that gives the `allocator` type for another type 

``` C++
template <typename T>
struct allocator { 
    template <typename U> 
    struct rebind {
        using other = allocator<U>;
    };
};
```


Witin `list`
- to create an allocator for for nodes as a field of `list`:
    - `Allocator::rebind<Node>::other alloc;`

Then use as in `vector`, details left as exercise 






