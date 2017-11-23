# Problem 25 - Abstraction over Iterator 

Question: what if I want to jump `n` spots in my container 

``` C++
template <typename Iter>
Iter advance(Iter it, ptrdiff_t n) {
    for(ptrdiff_t i = 0; i < n; ++i) ++it;
    return it;
}
```

It works, but it is O(n), but can we do O(1) ?
- E.g. `it += n`
- Well it depends 
    - for `vector`s, yes O(1) time 
    - for `list`s, no, `+=` is not supported 
        - even if it is, it is still going to be a loop 

Related 
- can we go backwards ? 
    - `vector`s - Yes 
    - `list`s - No 
- So all iterators support `!=`, `*`, `++` 
- but some iterators support other operations 


`list::iterator` 
- called a *forward iterator* 
- can only go one step forward 

`vector::iterator` 
- called a *random access iterator* 
- we can go anywhere (arbitrary pointer arithmetic)


Question: How can we write `advance` to use `+=` for random-access iterators and loop for forward iterators ? 

Since we have different kinds of iterators, let's create a type hierarchy :

``` C++
struct input_iterator_tag { }; // can only fetch value using `*` 

struct output_interator_tag { }; // can only send things into it but not fetching it 

struct forward_iterator_tag : public input_iterator_tag { };

struct bidirectional_iterator_tag : public forward_iterator_tag { };

struct random_access_iterator_tag : public bidirectional_iterator_tag { };
```

To associate each `iterator` class with a tag:
- we could use inheritance : 
``` C++ 
class list {
// ...

public:
    
    class iterator : public forward_iterator_tag {
        // ...
    };

};
```

But it makes it hard to ask what kind of iterator we have 
- we cannot `dynmaic_cast` since we don't have Vtable (since we don't have virtual dtor)
- doesn't work for iterators that aren't classes (eg. pointers)

Instead: make the tag a member: 
``` C++
class list {
// ...

public:
    
    class iterator {
        using iterator_category = forward_iterator_tag;
    };
};
```

Here we adopted a convention: 
- every iterator class will define a type member called iterator_category
- but it also doesn't work for iterators that aren't classes 
- But we aren't done yet 
    - we can make a template that associates every iterator with its category 

``` C++
template <typename It>
struct iterator_traits { 
    using iterator_category = typename It::iterator_category; 
//                            ^~~~~~~~
//                            it is needed because:
//                            - C++ can tell that `It::iterator_category` is a *type*
//                              (remember, compiler doesn't know anything about `It` )
//                              more on this later 
};
    
```

Then:
- `iterator_traits<List<T>::iterator>::iterator_category`  => `forward_iterator_tag` 
- to support pointers, we provides a specialized template for pointers 

``` C++
template <typename>
struct iterator_traits<T*> {
    using iterator_category = random_access_iterator_tag;
};
```



More on `typename`, consider: 
``` C++
template <typename T> 
void f() {
    T::something x; // this only makes sense if `T::somethign` is a type 
}
```

But:

``` C++
template <typename T> 
void f() {
    T::something * x; 
//               ^~~~
//               pointer declaration 
//               or multiplcation ? 
//               we need to know whether if `T::something` is a type 
//               compiler also assume NO 
}

// That means we have to say: 
template <typename T>
void f() {
    typename T::something x;
    typename T::something *y;
}
```

We need to say `typename` whenever we refer to a member type of a template parameter 

Back to Iterator traits: 
- for any iterator type `T`, `iterator_traits<T>::iterator_category` resolves to the tag struct for `T` (include if `T` is a pointer )
- What do we do with this ? 
    - note: naive implementation below won't even compiler 
``` C++ 
template <typename Iter>
Iter advance(Iter it, ptrdiff_t n) {
    if(typeid(typename iterator_traits<Iter>::iterator_category) == typeid(random_access_iterator_tag)) {
        return it += n;
    } else {
        // ... 
    }
} 
```

- The reason the above implementation won't compile is:
    - suppose iterator is not random-access, and it doesn't have a `+=` operator, `it+=n` will cause a compilation error, even though it will never be used 
    - Moreover, the choice of which implementation to be used is being made at run-time, when the right choice is known at compile-time 


- To make a compile-time decision - overloading 
``` C++
template <typename Iter>
Iter doAdvance(Iter it, ptrdiff_t n, random_access_iterator_tag) {
    return it += n;
}

template <typename Iter>
Iter doAdvance(Iter it, ptrdiff_t n, bidirectional_iterator_tag) {
    if (n > 0) {
        for(ptrdiff_t i = 0; i < n; ++i) ++it;
    } else if (n < 0) {
        for(ptrdiff_t i = 0; i > n; --i) --it;
    }
    return it;
}

template <typename Iter>
Iter doAdvance(Iter it, ptrdiff_t n, forward_iterator_tag) {
    if (n >= 0) {
        for(ptrdiff_t i = 0; i < n; ++i) ++it;
        return it;
    }
    throw SomeError{};
}
```

Finally, we create a wrapper function to select the right overload
```
template <typename Iter>
Iter advance(Iter it, ptrdiff_t n) {
    return doAdvance(it, n, typename iterator_traits<it>::iterator_category{});
}
```


Now the compiler will select the fast `doAdvance` for random access iterator, the slow `doAdvance` for bidirectional iterators, and the throwing `doAdvance` for forward iterators. 
These choices made at *compile-time* with no runtime cost 

This technique to perform compile-time computation is called _template metaprogramming_

Then, C++ templates forms a _functional language_ that operates at the level of types
- we can express condition by overloading
- repetition via recursive template instantiation 

``` C++
template <int N> 
struct Fact {
    static const int value = N * Fact<N-1>::value;
};

template <>
struct Fact<0> {
    static const int value = 1;
};

int main() {
    int x = Fact<5>::value; // 120 - evaluated at compile-time 
}
```


But for compile-time computation of values, C++11/14 offers a more straight forward facility: `constexpr` 

``` C++
constexpr fact(int n) {
    if(n == 0) return 1;
    else return n*fact(n-1);
}
```

`constexpr`:
- evaluates this at compile time if the argument is known at compile-time 
- if not, then do computation at run-time 
- Note: 
    - `constexpr` does not turn compiler into interpreter 
    - a `constexpr` must be something that actually can be evaluated at compile-time 
    - that means function cannot be virtual 
    - cannot mutate non-local variables 

















