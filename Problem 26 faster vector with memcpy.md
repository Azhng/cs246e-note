# I wan an even faster `vector` 

In the good old days of C, we could copy an array or even struct very quickly by calling `memcpy` (similar to `strcpy` but for arbitrary memory not just string)

`memcpy` was probably written in assembly, and was as fast as the machine could possibily be. 

But nowdays in C++, copies invoke copy ctors, which are costly function calls 

In C++, a type is considered *POD* (plain old data) if it:
- has a trivial default ctor (equivlent to `=default`)
- is trivially copyable
    - copy/move operations, dtor have default implementation 
- is standard layout 
    - no virtual methods or base class 
    - no reference members 
    - no fields in both base class and subclass 

For POD types, semantics is compatible with C, and `memcpy` is safe to use 

How can we use it - only safe if `T` is a POD type 

One option

``` C++
template <typename T> 
class vector {
    T *theVector;

public:

    // ... 

    vector(cont vector &other): /*...*/ {
        if (std::is_pod<T>::value) {
            memcpy(theVector, other.theVector, n*sizeof(T));
        } else {
            // as before 
        }
    }

};
```


Above implementation works, but the decision is being made at runtime, however the type is known at runtime (compiler may or may not optimize it)

2nd Option: (no runtime cost but won't work)

``` C++
template <typename T>
class vector { 

// ... 

public: 
    
    template <typename X=T> 
    vector(enable_if<std::is_pod<X>::value, const vector<T>&>::type other): /*...*/ {
        memcpy(/*...*/);
    }

    template <typename X=T>
    vector(enable_if<!std::is_pod<X>::value, const vector<T>&>::type other): /*...*/ {
        // original implementation 
    }

};
```

How does this compile (i mean, it's like it's actually gonna work):

``` C++
tempalte <bool b, typename> struct enable_if; // no implementation
tempalte <typename T> struct enable_if<true, T> { 
    using type = T;
}
```


With metaprogramming, what you don't say is as important as what you do say. 

This means that when `b` is true, `enable_if` defines a struct whose `type` member typdef is `T`. So if `std::is_pod<T>::value == true` then `enable_if<is_pod<T>::value, const vector<T>&>:type` reduces to `const vector<T>&`.
when `b` is false, the `enable_if` doesn't have an implementation, hence the code won't even compile. 

So one of the two versions of the copy ctor won't compile (it will be the one with false condition)

Then how is this a valid program ? 
- because it is, C++ said so. 

C++ rule: `SFINAE`
- Substitution-Failure-Is-Not-An-Error
- in other words, 
    - if `t` is a type, then `template <typename T> ____ f(____) {____}` is a template function, and substituting `T=t` results in an invalid function, the compiler then does not signal an error - it just removes that instantiation from consideraton during overload resolution 
    - on the other hand, if _NO_ version of the function is in scope and substitutes validly, that's an error 

Question: Why is wrong ?
- why do we need the extra templates out front ? 
    - because SFINAE applies to template functions, and these methods are ordinary functions (ctors) not templates,
        - they depend on `T`, but `T`' value is determined when we decided what to put in vector 
        - if substituting `T=t` fails, it invalidates the entire `vector` class, not just the method 
        - So make the method a separate template, with a new argument `X` which can be defaulted to T, and does `is_pos<X>`/`!is_pod<T>`

``` C++
template <typename T> 
class vector {

// ...

public:

    vector(typename enable_if<std::is_pod<T>::value, const vector<T>&)::type other) :: /*...*/ {
        /*...*/
    }

};
```



- For the template implementation we have above, it does compile, but hwen we run it, it crashes 
- Why ?
    - Hint: if you put debug statement statement into both ctors, they don't print. 
- Answer
    - we are getting the compiler - supplied copy ctor, which is doing shallow copies
    - because these templates are not enough to suppress the auto-generated copy ctor. 
    - and a non-templated match is always prefered to a templated one 
- How do we fix it? 
    - maybe this : 

``` C++
template <typename T>
class vector {
// ... 
public:
    vector(const vector &other) = delete; // explicitly disable the default copy ctor 
};
```

- well this is not allowed 
    - we cannot disable the copy dtor then create antoher one 
- Solution: overloading 

``` C++
template <typename T>
class vector {
// ...
    struct dummy{};

public:
    
    vector(const vector& other): vector{other, dummy{}} {} 
    //                           ^~~~~~~~~~~~~~~~~~~~~
    //                           ctor delegation 
    //                           causing this ctor using other ctor 

    tempalte <typename X=T>
    vector(typename enable_if<std::is_pod<X>::value, const vector<T>&)::type other, dummy) {/*...*/}

    template <typename X=T>
    vector(typename enable_if<!std::is_pod<X>::value, const vector<T>&)::type other, dummy) {/*...*/}
};
```


- overload the ctor with an unused "dummy" arg, 
- have the copy ctor delegate to the overloaded ctor 
- copy ctor is inline, so there is no function call overhead 
- and it WORKS 


With C++14, we can write soem "helper" functions to make `is_pod` and `enable_if` easier to use 
- E.g.
``` C++
template <typename T> constexpr bool is_pod_v = std::is_pod<T>::value;

template <bool b, typename T> using enable_if_t = typename enable_if<b, T>::type;

// with these short hand, we can do following:

template <typename T>
class vector {
// ...
public:

    // ...
    template <typename X=T>
    vector(enable_if_t<is_pod_v<X>, const vector<T>&> other, dummy) {/*...*/}

    template <typename X=T>
    vector(enable_if_t<!is_pod_v<X>, const vector<T>&> other, dummy) {/*...*/}

};
```


# `Move` / `Forward` implementation

We now have enough machinery to implement `std::move` and `std::forward` 

## `std::move` - first attempt 

``` C++
template <typename T>
T&& move(T&& x) {
    return static_cast<T&&>(x);
}
```

- doesn't quite work 
    - `T&&` is a universal reference, not an rvalue reference 
        - thus, if `x` was lvalue reference `T&&` is an lvalue reference 
    - We need to make sure `T` is *NOT* an lvalue reference
        - If `T` is an lvalue reference, get rid of the reference

## `std::move` - second attempt 
``` C++
template <typename T>
inline typename std::remove_reference<T>::type&& move(T&& x) {
    return static_cast<typename std::remove_reference<T>::type &&> (x);
    //                 ^~~~~~~~~~~~~~~~~~~~~~~
    //                 turns `T&` and `T&&` into `T` 
}
```

## `std::remove_reference` - left as exercise 


- Question: Can we save typing and using auto ? 
    - E.g.
``` C++
template <typename T>
auto move(T&& x) {/*...*/}
```
- Answer: NO ! 
    - By-value `auto` throws away refs and otuer consts 
    - By-ref `auto&&`  is universal reference

``` C++
int z; 
int &y = z;
auto x = y; // `x` is an int 

const int &w = z; 
auto v = w; // `v` is int 
```


- we need a type definition rule that doesn't descard `ref` 
    - *`decltype(/*...*/)`*
        - returns the type that `/*...*/` was *declared* to have 
    - E.g.
        - `decltype(var)` returns the declared type of the variable 
        - `decltype(expr)` returns lvalue reference or rvalue refernece depending on the epxression 

``` C++
int z;
int &y = z;
decltype(y) x = z; // x is now int& 

x = 4; // now z = 4 

auto b = z; 
b = 4;  // `z` is unaffected 

decltype(z) s = z; // `s` is int 
s = 5; // `z` is unaffected 

// interesting fact 
decltype((z)) r = z; // `r` is int& is `(z)` is an expression with return value of `int&` 
r = 5;  // does affect `z` 
```

Now we can write `std::move` using `decltype`:
    - `decltype(auto)` 
        - perform type deduction like `auto` but using the `decltype` rules 

## `std::move` - third attempt 
``` C++
template <typename T>
decltype(auto) move(T&& x) {
    return static_cast<std::remove_reference_t<T>&&>(x);
}
```



## `std::forward` - first attempt 
``` C++
template <typename T>
inline T&& forward(T&& x) {
    return static_cast<T&&>(x);
}
```

- if `x` is a lvalue, `T&&` is an lvalue ref, 
- if `x` is a rvalue, `T&&` is an rvalue ref 
- it seems like we are casting it to the right thing 
    - but of course it doesn't work 
    - `std::forward` is called on expression that are lvalues that may point at rvalues 
    - E.g.
```
template <typename T>
void f(T&& y) {
    ... std::forward(y) ... 
//      ^~~~~~~~~~~~~~~
//      regardless whether `y` points at lvalue or rvalue 
//      `y` itself is always lvalue 
//      then our implementation will always return lvalue ref 
}
```

- the information of lvalue / rvalue is actually stored in `T` instead of `y` 
- `std::forward` must know what type (including l/rvalue) was deduced for `y`
    - i.e. it needs to know `T` 
    - in principle, `forward<T>(y)` would work (but doesn't in real life)
        - we have 2 problems 
            - supplying `T` means `T&&` is no longer universal 
            - we need to prevent user from omit `T` argument 
    - instead:
        - separate rvalue lvalue cases 

## `std::forward` - second attempt 
``` C++
template <typename T>
inline constexpr T&& forward(std::remove_reference_t<T>& x) noexcept {
    return static_cast<T&&>(x);
}

tempalte <typename T>
inline constexpr T&& forward(std::remove_reference_t<T>&& x) noexcept {
    return static_cast<T&&>(x)
}
```







