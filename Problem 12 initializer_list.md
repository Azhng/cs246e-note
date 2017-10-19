# Problem 12 - Better Initialization

Array: 
``` C++
int a[] = {1, 2, 3, 4, 5}; // :) 
```

Vectors:
``` C++
vector<int> v;
v.push_back(1); // :( 
// ... 
```

Long sequence of push_backs can be __clunky__

## Goal: __Better Initialization__ 
``` C++
template <typenmae T>
class vector {
    // ... 
public: 
    vector() : /*...*/ {}
    vector(size_t n, T i = T() ) 
        : n(n), cap(n == 0 ? 1 : n), theVector{new T[cap]} {
            for(size_t j=0; j<n; ++j) {
                theVector[j] = i;
            }
        }
};

int main() {
    vector<int> v; // empty 
    vector<int> w{5}; // {0, 0, 0, 0, 0}
    vector<int> z{3, 4}; // {4, 4, 4}
}
```

### Note: 
- `T{}` - (default ctor) means 0 if `T` is a built-int type 
- Better, but not as good as TRUE array-style initialization 

## Initializer list 
``` C++ 
#include <initializer_list>
template <typename T>
class vector {
// ...
public: 
    vector() : /*...*/ {}
    vector(size_t n T, T i = T{}) /*...*/;
    vector(std::initializer_list<T> init)
        : n{init.size()}, cap{init.size()}, theVector{new T[cap]} {
            size_t i=0;
            for(auto &t : init) theVector[i++] = t;
        }
};

int main() {
    vector<int> v{1,2,3,4,5}; // 1, 2, 3, 4, 5
    vector<int> v; // empty 
    vector<int> v{5} // 5
    vector<int> v{3, 4}; // 3, 4
}
```
Observe that other ctor not being called
- Default ctors take precedence over initializer_list ctor
- initializer_list ctor takes precedence over other ctors

Therefore, to get the other ctor to run: we use round bracket initialization 
``` C++
int main() {
    vector<int> v(5); // 0, 0, 0, 0, 0
    vector<int> v(3, 4); // 4, 4, 4
}
```

Recall for C++ initialization: 
``` C++
int x=5;                // allowed 
string s = "hello";     // allowed 
ifstream f= "file.txt"  // X not allowed 

int x(5);               // allowed 
string s("hello");      // allowed
ifstream f("file.txt"); // allowed 
struct vec v{1, 2};     // allowed

int x{5};               // new trend allowed 
string s{"hello"};      // allowed
struct vec v{1, 2};     // allowed 
```

## A note on cost: 
- item in an initializer_ist are stored in continous memory (because `begin()` method returns a pointer)
- so we are using one array to build another 
- in principle, there are two copies in memory 
- However, do not attempt to directly replace vector/array with initializer_list 
    - initializer_list are meant to be immutable 
    - do not try to modify their contents
    - do not use them as standalone data structures 

## But also note: 
- only one allocation in vector, not several 
- no doubling and reallocating as there was with a sequence of `push_back` 
- in general: if you know how big your vector will be, you can save reallocation cost by requesting space upfront 

``` C++
template<typename T> 
class vector {
// ...
public:
// ...
    void reserve(size_t newCap) {
        if(cap < newCap) {
            T *newVec = new T[newCap];
            for(size_t i=0; i<n; ++i) newVec[i] = theVector[i];
            delete[] theVector;
            theVector = newVec;
            cap = newCap;
        }
    }
};
```

Rxercise: rewrite `vector` so that `push_back` uses `reserve` instead of `increaseCap` 

``` C++
int main() {
    vector<int> v;
    v.reserve(10);
    v.push_back(1); 
    // ... 
    // ...
    // now we can do 10 push_back without reallocating
}
```