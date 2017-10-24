# Problem 15 - Memory management is HARD !

## No it isn't 

`vector`s - do everything array can
- grow as needed in O(1) amortized time or better 
- clean up automatically when go out of scope 
- are tuned to minimize copy 

Just use vectors and you will never have to use array again 

C++ has enough abstraction facilities to make programming __easier__ than C 

But what about single objects ?

``` C++
void f() {
    Posn *p = new Posn{1, 2};
    // ...
    delete p; // must explicitly delete posn 
}
```

First of all: "do we really need to use heap?" (why not just use the stack instead)

A.k.a.
``` C++
void f() {
    Posn p{1, 2};
    // ...
}
```

And stack allocation is way faster than heap allocation 


## However, sometimes we do need heap. Calling `delete` isn't so bad 
But consider: 
``` C++
void f() {
    Posn *p = new Posn{1, 2};
    // ... 

    if(some_condition) {
        throw BadNews {};
    }

    // ...
    delete p;
}
```

What happens to `p` ? __LEAKED__, UNACCEPTABLE 

## Raising and handling an exception should not corrupt the program
- we desire exception safety 
- Leaks are corruption of the program's memory 
- will eventually degrade performance and crash programs 

### If a program cannot recover from an exception without corrupting its memory, what is the point of recovering ? 

## What constitutes exception safety ? 3-levels 
1. Basic guarantee 
    - once an exception has been handled, the program is in some valid state 
    - Some valid state includes:
        - no leaked memory 
        - no corrupted data structures
        - all invariants are maintained 
        - (a.k.a. program is runnable)
2. Strong guarantee 
    - if an exception propagates out of a function `f`, then the state of the program will be as if `f` had not been called 
    - `f` either succeeds completely or not at all 
3. Nothrow guarantee 
    - a function `f` offers nothrow guarantee __never__ emits an exception 
    - and always accomplish its purpose 

// We will revisit this next week 

## Coming back to `f`...
To fix memory leak issue 
``` C++
void f() {
    Posn *p = new Posn(1, 2);
    if(some_condition) {
        delete p;
        throw BadNews {};
    }

    delete p;
}
```

Now we have duplicated effort to deallocate `p`, memory management is even HARDER 

## We want garantee that `delete p;` happens no matter what 
## What guarantees does C++ offers ? 
- Only one: `dtor` for stack-allocated objects will be called when objects go out of scope

So - we can create a class with a `dtor` that deletes the pointer 

``` C++
template <typename T>
class unique_ptr {
    T *p;

public: 
    unique_ptr(T *p) : p{p} {}

    ~unique_ptr() { delete p; }

    T* get() const { return p; }

    T* release() {
        T *q = p;
        p = nullptr;
        return q;
    }
};

void f() {
    unique_ptr<Posn> p {new Posn{1, 2}};
    if(some_condition) throw BadNews{};
}
```

That's it - now we have less memory management effort than we started 


## Using `unique_ptr` 
- can use `get` to fetch the raw pointer
- Better 
    - make `unique-Ptr` act like a pointer 

Version 2.0
``` C++
template <typename T> 
class unique_ptr {
    T *p;

public: 

    unique_ptr(T *p) : p{p} {}

    ~unique_ptr() { delete p; }
    // ...
    T& operator*() const { return *p; }
    T* operator->() const { return p; }
    // operator-> let you decide what's LHS pointer going to be 
    // whatever pointer -> returns, C++ dereference that pointer 

    explicit operator bool() const {
//  ^~~~~~~
// prevent bool b = p;
        return p;
    }   
    // conversion operator don't have types 

    void reset(T *p1) {
        delete p;
        p = p1;
    }

    void swap(unique_ptr<T> &x) {
        std::swap(p, x.p);
    }
};

void f() {
    unique_ptr<Posn> p { new Posn{1, 2} };
    cout << p->x << " " << p->y << endl; 
}
```

## But consider:
``` C++
unique_ptr<Posn> p{new Posn{1, 2}};
unique_ptr<Posn> q = p; // now we are in trouble
                        // Undefined behaviour 
                        // destructor will be ran twice 
```

## Solution: *copying `unique_ptrs` are __NOT__ allowed*
- but it's perfectly ok to move them 
``` C++
template <typename T>
class unique_ptr {
// ...
public:

    // delete copy ctor 
    unique_ptr(const unique_ptr &other) = delete; 

    // delete copy assignment 
    unique_ptr& operator=(const unique_ptr &other) = delete;

    // define move ctor
    unique_ptr(unique_ptr &&other) : p{other.p} {
        other.p = nullptr;
    }

    // define move assignment 
    unique_ptr& operator=(unique_ptr &&other) {
        swap(other);
        return *this;
    }
}
```

It is safe to return `unique_ptr` since before the `dtor` runs the value is being moved to the return value first 

## Note: 
- this is how copying of `stream`s is prevented 

## Passing `unique_ptr` to function:
``` C++
void f(unique_ptr<Posn> p);
unique_ptr<Posn> q = {/*...*/};
f(q); // this crash 
f(std::move(q)); // this forces ownership of q and give it to f 
```
### Note: passing a `unique_ptr` by value means: __transfer of ownership__ 

## Small exception safety issue: (Consider the following)
``` C++
class C{/*...*/};
void f(unique_ptr<C> x, int y) {/*...*/}
int g() {/*...*/}

f(unique_ptr<C>{new C;}, g());
```

Note here that C++ does not enforce order of argument evaluation,
The order could be:
1. new C
2. g()
3. unique_ptr<C> {`new C`}

So if this is the case, then what happen if `g` throws an exception 
- the step 1 is going to cause a leak 
- so we need a way to combine step 1 and step 3 into an atomic operation 
    - using a helper function 


``` C++
template<typename T, typename... Args> 
unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T> { new T(std::forward<Args>(args)...) };
}
```

So the application code would become 
``` C++
f(make_unique<C>(), g());
```

There is not leak in this case 

## `unique_ptr` is an example of the C++ idiom __Resource Acquisition Is Initialization__ (RAII)
- any resource that must be properly released (memory, file handle, etc.) should be wrapped in a stack-allocated object, whose `dtor` frees it 

## We now have seen 2 classes that follows RAII idiom 
- `unique_ptr`
- `ifstream` / `ofstream`
    - acquire the resource when the object is initialized 
    - release it when the object's `dtor` runs 
