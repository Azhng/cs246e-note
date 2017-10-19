# Problem 14 - Less copying ! 

Before: 
``` C++
void push_back(int n) 
```

Now: 
``` C++
void push_back(T x) {
    //         ^~~~ (1)
    //         since now T is an object, how many times is T being copied 
    increaseCap();
    new (theVector + (n++)) T(x);
    //   ^~~~~~~~~~~~~~~~~~~~~~~~ (2)
    //   here we are copying T again 
}
```

Q: How many times is T being copie ?
- if the argument is an lvalue (2 copies)
    - (1) and (2) are using move ctor  
    - we are making two copies
    - but we only want one 
- if the argument is an rvalue (1 copy)
    - (1) is a move ctor 
    - (2) is a copy ctor because `x` is an lvalue 

Well, we want __0__ copy 

Fix: 
``` C++
void push_back(T x) {
    increaseCap();
    new (theVector + (n++)) T(std::move(x));
    //                        ^~~~~~~~~~~~
    //                        forcing this to be a move ctor rather than copy ctor
}
```

Now:
- lvalue: copy + move
- rvalue: move + move 

## Question: What if `T` doesn't have a move ctor: 
- well it is going to 2 copies anyway 

Then why don't just take `T` by reference 
- 1 copy 
- no move 
``` C++
void push_back(const T &x) {
    increaseCap();
    new (theVector + (n++)) T(x);
    //                       ^~~
    //                       copy ctor 
}
```

But what if `T` has a move ctor ? we need to take advantage of that 
``` C++
void push_back(T &&x) {
    //          ^~~~~
    //          no move no copy 
    increaseCap();
    new (theVector + (n++)) T (std::move(x));
}
```

If there is no move ctor
- lvalue: 1 copy 
- rvalue: 1 move 

If no move ctor:
- 1 copy regardless 

Now consider: 
``` C++
vector<posn> v;
v.push_back(posn{3, 4});
```
For this code:
1. `ctor` call to create the `posn` obj
2. `copy` of `move` construction into the `vector` (depending on whether posn has a move ctor)
3. `dtor` call on the temporary object 

Well we could eliminate step 1 and 3 if we could get vector to create the object instead of the client 
- pass `ctor` args to `vector` instead of the object itself 

## HOW ? 
### Soon but first ......

## A note on template functions
Consider `std::swap` - seems to work on all types 

Implementation of `std::swap`
``` C++
template <typename T>
void swap(T &a, T &b) {
    T tmp (std::move(a));
    a = std::move(b);
    b = std::move(tmp);
}
```


``` C++
int x=1, y=2;
swap(x, y);
swap<int>(x, y); // this is equivlent to last one, but not required 
                 // type are inferred from the parameters 
```

In general: only have to say `f<T>(~~~~)` if `T` cannot be deduced from the arguments

### Type deduction for template arguments 
- follows the same rules as type deduction for `auto` 

## Problems: 
Back to `vector` - passing `ctor` args 
1.  we don't know what types `ctor` argument should have 
    - `T` could be any class, and that class could have several `ctor`s 
2. we don't know how many `ctor` args there are?

## Solutions
Idea/Solution - member __template__ function 
1. similar to `std::swap`, it would take anything 
2. __variadic template__ - similar to Racket macros

``` C++
template <typename T>
class vector {
// ...
public:
    // ...
    template <typename... Args> 
    void emplace_back(Args... args) {
        increaseCap();
        new (theVector + (n++)) T(args...);
    }
};
```

Here:
- `Args` is a __sequence__ of type of variables denoting the __types__ of the actual arguments of `emplace_back` 
- `args` is a __sequence__ of program variables denoting the __actual arguments__ of `emplace_back`


Now, we can do this:
``` C++
vector<posn> v;
v.emplace_back(3, 4);
```

## Problem: args is being taken by value
- can we take args by reference ? 
- lvalue or rvalue ref ?
    - given we have mulitiple arguments we could have a __mix__ of both 


``` C++
template <typename... Args> 
void emplace_back(Args&&... args) {
    //            ^~~~~~~
    //            C++ special rule here 
    //            universal reference 
    //            - can point to an lvalue or an rvalue 
    increaseCap();
    new (theVector+(n++)) T(args...);
}
```

## Question: when is a reference universla? 
- must have the form `T&&` where `T` is the type argument being deduced for the current template function call 

Example: 
``` C++
template <typenmae T>
class C {
public: 
    template <typename U>
    void g(U&& x) {
        // ^~~~
        // this is universal reference 
    }

    template <typename U>
    void h(const U&& x);
    //     ^~~~~~~~~~~
    //     this is NOT universal reference 
    //     it simply just means constant rvalue reference 

    void j(T&& x);
    //    ^~~~
    //    this is NOT universal reference 
    //    because T is not being deduced here, T is already known 
};
```


## Now recall:
``` C++
class C{/*...*/};

void f(C&& x) {
//     ^~~
//     rvalue reference 
//     x points to an rvalue 
//     x is an lvalue 
    g(x);
//  ^~~~
//  then x is a lvalue when it is passed into g
}
```

If we want to pass `x` as a rvalue when passed to `g`, so the "moving" version of `g` is called

``` C++
void f(C&& x) {
    g(std::move(x));
}
```

In the case of `Args&&... args`, we don't know if the `args` are lvalue, rvalue or mix, 

I want to call `move` on `args` iff `args`are `rvalue`s 
``` C++
template <typename... Args> 
void emplace_back(Args&&... args) {
    increaseCap();
    new (theVector+(n++)) T(std::forward<Args>(args)...);
    //                      ^~~~~~~~~~~~
    //                      calls std::move if its argument is rvalue 
    //                      else does nothing 
}
```

### Perfect forwarding 
Now `args` is passed to `T`'s ctor with lvalue / rvalue information preserved 
