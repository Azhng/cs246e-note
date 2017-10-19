# Problem 16: Is `vector` exception-safe ?
## Consider: 
``` C++
template <typename T>
class vector {
    size_t *theVector;

public:
    vector(size_t n, const T &x) : 
        n{n}, 
        cap{n}, 
        theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
            //                    ^~~~~~~~~~~~~~~~~~~~
            //                    this could throw 
            for(size_t i = 0; i<n; ++i) {
                new (theVector + i) T(x);
                //                  ^~~
                //                this could throw
            }
        }
};
```


## If an exception is being thrown during the `ctor`
- partially constructoed vector 
- `dtor` will not run 
    - broken invariant (we don't have `n` valid objects)
- Note:
    - if `operator new` throws 
    - nothing has been allocated 
    - strong guarantee 

## Fix 
``` C++
template<typename T>
vector<T>::vector(size_t n, const T&x) :
    n{n}, cap{n}, 
    theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
        size_t progress = 0;
        try {
            for(size_t i = 0; i<n; i++) {
                new (theVector+i) T(x);
                ++progress;
            }
        } catch (...) { // catch anything 
            for(size_t i=0; i<progress; ++i) {
                theVector[i].~T();
            }
            operator delete(theVector);
            throw; // rethrow the exception 
        }
    }
```

## We can abstract the filling part into its own function 
``` C++
template <typename T>
void uninitialized_fill(T *start, T *finish, const T &x) {
    T *p;
    try {
        for(p=start, p != finish; ++p) {
            new (static_cast<void *>(p)) T(x);
            //   ^~~~~~~~~~~~~~~~~~
            //   prevent user customize override the operator new 
        }
    } catch (...) {
        for(T *q = start; q != p; ++q) q->~T();
        throw; 
    }
}
```

## This is our `All or nothing`
- strong guarantee 

## New version of `ctor`
``` C++
template <typename T> 
vector<T>::vector(size_t n, const T &x) :
    n{n},
    cap{n},
    theVector{ static_cast<T*>(operator new (n*sizeof(T))) } {
        try {
            uninitialized_fill(theVector, theVector + n; x);
        } catch (...) {
            operator delete (theVector);
            throw;
        }
    }
```

## We can clean this up by using RAII
``` C++
template <typename T>
struct vector_base {
    size_t n, cap;
    T *v;

    // acquire resource uppon initialization 
    vector_base(size_t n): n{n}, cap{n == 0 ? 1 : n}, 
        v{ static_cast<T*> (operator new(n*sizeof(T))) } { }

    // release resoruce upon destruction 
    ~vector_base() {
        operator delete (v);
    }
};

template <typename T>
class vector {
    vector_base<T> vb; // cleaned up implicitly when vector is destroyed 

public:

    vector(size_t n, const T &x) : vb{n} {
        uninitialized_fill(vb.v, vb.v + vb.n, x);
    }

    ~vector() {
        destroy_items();
        // this is defined in placement_new problem 
    }
};
```

Right now the `vector(size_t n, T &x)` is completely exception safe 

## Copy ctor:
``` C++
template <typename T>
vector<T>::vector(const vector &other) : vb{other.size()} {
    uninitialized_copy(other.begin(), other.end(), vb.v);
//  ^~~~~~~~~~~~~~~~~~~~
//  left as exercise // WTF
//  similar to uninitialized_fill
}
```

## Assignment operation
using `copy` and `swap` is exception safe, because `swap` is a nothrow


## Push_back
``` C++
void push_back(const T &x) {
    increaseCap();
    new (vb.v + (vb.n++)) T{x};
    //                   ^~~~~
    //                   this could potentially throw 
}
```

therefore we should write: 
``` C++
void push_back(const T &x) {
    increaseCap();
    new (vb.v + vb.n) T{x};
    ++vb.n;
}
```
In this case, if the contructor throws, we have the same vector 

But what about `increaseCap` ? now we have more memory allocated than what we originally have

Better `increaseCap`
``` C++
void increaseCap() {
    if(vb.n == vb.cap) {
        vector_base vb2 {2 * vb.cap}; // RAII
        uninitialized_copy(vb.v, vb.v + vb.n, vb2.v); // strong guarantee 
        destroy_item(); // presumed to be no-throw 
        std::swap(vb, vb2); // no-throw 
    }
}
```

Combining `increaseCap` and `push_back` we now have strong guarantee using a single `try` block in `uninitialized_copy` and `uninitialized_fill`

## Efficiency issue 
Now we have an efficiency issue 
- copying from the old array to the new one 
- old array should be destroyed 
- then maybe we should be __MOVING__ then __COPYING__
    - but moving destroys the old array, 
    - so if an exception is thrown during moving, our `vector` is destroyed 
    - we can only move if the we are sure that `move` operation is nothrow

`increaseCap` V3.0
``` C++
void increaseCap() {
    if(vb.n == vb.cap) {
        vector_base vb2{2 * vb.cap};
        uninitialized_copy_or_move (vb.v, vb.v + vb.n, vb2.v);
        destroy_elements();
        std::swap(vb, vb2);
    }
}

template <typename T>
void uninitialized_copy_or_move(T *start, T *finish, T *target) {
    T *p;
    try {
        for(p=start; p != finish; ++p, ++target) {
            new (static_cast<void *>(target)) T(std::move_if_noexcept(*p));
        }
    } catch(...) { // will never happen if T has a non-thorwing move ctor 
        for(T *q=start; q != p; ++q) (target + (q-start))->~T();
        throw;
    }
}
```

`std::move_if_noexcep(x)` 
- produces `std::move(x)` if `x` has a non-throwing move ctor 
- produces `x` otherwise 

But how should the compiler know if `T`'s move ctor is non-throwing ?
- You tell the compiler 

``` C++
class C {
public:
    C(C &&other) noexcept; // guarantee that this won't throw 
    // ...
};
```

In general, `move` and `swap` should be non-throwing, declare them so will allow more optimized code to run.

Any function you are sure will never throw or propagate an exception, we should declare `noexcpt` 

## Question: Is `std::swap` noexcept? 
``` C++
template <typename T> 
void swap(T &a, T &b) {
    T c(std::move(a)); 
    a = std::move(b);
    b = std::move(c);
}
```

## Answer: Only if `T` has 
- `noexcept` move `ctor`
- `noexcept` move assignment 

## Question: How do we specify this? 
``` C++
template <typename T> 
void swap(T &a, T &b) 
    noexcept(std::is_nothrow_move_constructible<T>::value &&
             std::is_nothrow_move_assignable<T>::value) {

}
```

## Note: `noexcept` is equivlent to `noexcept(true)`