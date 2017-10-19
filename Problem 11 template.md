# Problem 11 - I want a vector of chars 

Start over? __NO__

## Introducing a major abstraction mechanism - __Templates__
- generalize over types

``` C++
// vector.h

namespace CS246E {
    template <typename T>
    class vector {
        size_t n, cap, size;
        T *theVector;
    public:
        vector();
        // ... 
        void push_back(T n);
        T& operator[](size_t i);
        const T& operator[](size_t i);

        using iterator = T *;
        using const_iterator = const T *;

        // etc.
    };

    template <typename T> 
    vector<T>::vector() n(0), cap(1), size(0), theVector(new T[cap]) {}

    template <typename T>
    void vector<T>::push_back()(T n) {/*...*/}
}
```

## NOTE: 
- we must put implementation in the `.h` files 
    - why? 
        - `template` is essentially boiler plate code, 
        - it is instantiate very early in the compilation process 

``` C++
// main.cc
int main() {
    vector<int> v; // vector of ints 
    v.push_back(1);
    // ...

    vector<char> w; // vector of chars
    w.push_back('a');
    // ...

}
```

Here we are essentially asking C++ compiler to write `vector` class twice, once for `char` and once for `int`

## Semantic: 
- the first time compiler encounters `vector<int>`, int creates a version of the vector code where `int` replaces `T`, and then compile that class 
- it can't do that unless it has all the details about the class 
- so the implementation must be available in the `.h` file