# Problem 34 - I want a (tiny bit) smaller revec class 

Currently, vector / vector_base have an alloccator field,

What is size ?  0 ?
- No C++ does not allow size of 0 type 
- mess things up (e.g. pointer arithmetic)


Every type has size >= 1

Practically 
- compiler adds a dummy char to the allocator 

So having an allocator field makes the vector larger by a byte(probably more due to alignment) 


Having an allocator field in vector_base may add another byte (or more)

To save this space, C++ provides the empty space optimization 



Unser EBO 
- an empty base cass does not have to occupy space in an object 


So we can eliminate the space cost of allocator by making it base class, at the same time, make vector_base a base class of vector

``` C++
template <typename T, typename Alloc = allocator<T>
struct vector_base : Alloc {
    size_t n, cap; 
    T* v;
    using Alloc::allocate;
    using Alloc::deallocate;

    // etc.
    vector_base (size_t n) : n{0}, cap{n}, v{allocate(n)} {}
    ~vector_base() { deallocate(v); }
};

template <typename T, typename Alloc=allocator<T>>
class vector : vector_base<T, Alloc> {
    using vector_base<T, Alloc>::n;
    using vector_base<T, Alloc>::cap;
    using vector_base<T, Alloc>::v;

    using Alloc::allocate:
    using Alloc::deallocate:

public:
    
    // use n, cap, v instead of vb.n, vb.cap, vb.v 

}
```


`uninitialized_copy` etc. 
- need to call construct / destroy 
- simplest - let them take an allocator as param 

``` C++
template <typename T, typename Alloc>
uninitialized_fill(T* start, T* finish, const T& x, Alloc) {
    // ... 
    a.construct(/*...*/);
    // ... 
    a.destroy(/*...*/);
}
```


Question: How can `vector` pass an allocator to these functions? 
- `uninitialize_fill(v, v + n, x, static_cast<Alloc&>(*this))`;
    - cast ourselves to base class reference 
- remaining details - exercise 







