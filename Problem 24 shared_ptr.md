# Problem 23 - Shared Ownership

Question: "Is C++ hard" 
- No (if you're a client programmer)
- But 
    - explicit memory management ...
    - use `vector` if you need an arry 
    - use `unique_ptr` if you need a heap object 
    - use stack-allocated objects as much as possible 
- if we follow these, you should never have to call `delete` or `delete[]` 

- `unique_ptr` also respects `is-a` 
``` C++
unique_ptr<Base> p(new Derived); // OK 
p->virtfn(); // runs dervied version, OK 
```
- But: 
``` C++
unique_ptr<Derived> q = /*...*/;
unique_ptr<Base> = std::move(q); // Not allowed (the version we implmented in class)
                                 // Type error
                                 // - conversion between unique_ptr<Derived> and unique_ptr<Base>
```

- However, we can fix this
    - solution works for any `unique_ptr` whose pointer is assignment with `this->p` 
    - E.g.
        - subtypes of `T`, but not super-types of `T` 

``` C++ 
template <typename T>
class unique_ptr {
    T *p;
    // ... 

public:
    
    // ... 
    template <typename U>
    unique_ptr(unique_ptr<U> && q) : p{q.p} { // only compiles when `p` is assignable from `q.p` 
        q.p = nullptr;
    }

    template <typename U>
    unique_ptr& operator=(unique_ptr<U> &&q) {
        std::swap(p, q.p); // this implies T and U is mutually assignable ? 
                           // what if this is not the case ?
                           // left as exercise 
        return *this;
    }
};
```



- But: what if I want two smart pointers pointing at the same object ? 
    - Why ? 
        - the pointer that owns the object should be a unique_ptr
        - all others could be raw pointer 
    - When would we ever need TRUE shared ownership ? 
        - Recall (Racket):

``` Racket 
(define l1 (cons (1 cons(2 (cons 3 empty)))))
(define l2 (cons 4 (rest l1)))
```

In memory: 
```
the tail are being shared: 
l1 |1|-> |2|->|3|/|
l2 |4->|->^
```


Shared data structure are a nightmare in C, how can we ensure each node is freed exactly once ? 
- this is rather easy in garbage collected languages 
- What can C++ do ? 

``` C++
template <typename T>
class shared_ptr {

    T *p;
    int *refcount; // count how many shared_ptrs are pointing at `*p` 
                   // updated each time a shared_ptr is initialized / assigned / destroyed 
                   // `refcount` is shared among all `shared_ptr` that point to `*p` 
                   // `*p` is only deleted if its refcount reaches 0 
                   // implementation is left as exercise
public: 

};
```

So:
``` 
l1: |1| --shared_ptr-> |2| --shared_ptr--> |3|/|
l2: |4| --shared_ptr-->-^
```


We just need to watch out for: *Cycles*

```
    |1| -> |2|*|
     ^-<------V

```

If we have a cyclic data 
- we may have to physically break the cycle 
- or we can use `weak_ptr`s 

Also what for:
``` C++
Book * p = new /*...*/; 
shared_ptr<Book> p1 {p};
shared_ptr<Book> p2 {p}; // NOOOOO 
                         // it will compile, but just .... NO pls 
```

- `p1` and `p2` will not be sharing `refcount` 
- if we want 2 shared_ptrs at an object, create one shared_ptr then copy it 



But 
- we can't `dynamic_cast` these pointers 
- how about we just write one ? (FYI, it's in STL)
``` C++
template <typename T, typename U>
shared_ptr<T> dynamic_pointer_cast(const shared_ptr<U> &spu) {
    return shared_ptr<T>(dynamic_cast<T*>(spu.get()));
}
```

- similarily, we can have `const_pointer_cast` and `static_pointer_cast` 




