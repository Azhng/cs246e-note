# Problem 17 - Abstraction over Container 

Recall - `map` from Racket
``` Lisp
(map f (list a b c)) ;; (list (f a) (f b) (f c))
```

Maybe we want to do the samething with `vector` 
```
 _ _ _ _      ___ ___ ___ ___
|a|b|c|d| ~> |f a|f b|f c|f d|
```

Assume: target has enough space to hold as much of source as we want to send 

``` C++
template <typename T1, typename T2>
void transform(const vector<T1> &source, 
               vector<T2> target, 
               T2(*f)(T1)) {
    auto it = target.begin();
    for(auto &x : source) {
        *it = f(x);
        ++it;
    }
}
```

This is fine, but not very abstract
- what if we only want part of the source? 
- what if we want to send source to other part of target? rather than the beginning 

New version

``` C++
template <typename T1, typename T2>
void transform(vector<T1>::iterator start, 
               vector<T1>::iterator finish, 
               vector<T>::iterator target, 
               T2(*f)(T1)) {
    while(start != finish) {
        *target = f(*start);
        ++start;
        ++target;
    }
}
```

Ok, but:
- what if I want to transform a `list`, I'll write the same code again 
- what if I want to transform a `list` to a `vector`, or vice-versa 
- we can make the types stand for the iterators, not the container elements 
- but then how do we indicate the type for `f` ? 

``` C++
template<typename InIter, typename OutIter, typename Fn> 
void transform(InIter start, InIter finish, OutIter target, Fn f) {
    while(start != finish) {
        *target = f(*start);
        ++start;
        ++target;
    }
}
```

works over `vector` iterators, `list` iterators or any other kind of iterators, `InIter` / `OutIter`, including basic pointers 

C++ will instantiate a template function with any type that has the operation being used by the function 

`Fn` can be any type that supports function applications, 

``` C++
class Plus {
    int n;
public:
    Plus(int n) : n{n} {}
    int operator()(int m) {return n + m;}
};

Plus p{5};
cout << p(7); // 12 
//      ^~~~
//      `p` here is essentially a function object 

transform(v.begin(), v.end(), w.begin(), Plus{1});
// OR
transform(v.begin(), v.end(), w.begin(), [](int n) {return n+1;});
//                                        ^~~~~~~~~~~~~~~~~~~~~~~~
//                                        lambda 
```

# _`Lambda`_: 

## Syntax
```
[~~capture list~~](~~param list~~) mutable? noexcept? {~~body~~}
```


## Semantics: 
``` C++
void f(T1 a, T2 b) {
    [a, &b](int x) {/* body */}
}
```
translate to :
``` C++
void f(T1 a, T2 b) {
    class ~some_jibberish~ {
        T1 a;
        T2 &b;
    public:
        ~some_jibberish~(T1 a, T2 &b) : a{a}, b{b} {}

        auto operator()(int x) const {
            /* body */
        }
    }
}
```

- It is basically an anonymous class, but cannot access the name 
- note that `operator()` are marked as `const` method, that means lambda function are not allowed to change the fields they capture 
- If the lambda is declared `mutable`, then `operator()` is not marked `const`
- `capture list` provides access to selected variables in the enclosing scope 


