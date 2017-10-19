# Prolbem 13: I want a vector of Posns
``` C++
struct Posn {
    int x, y;
    Posn(int x, int y) : x{x}, y{y} {} 
}

int main() {
    vector<Posn> v; // won't compile !! WHY NOT ?! 
}
```

## Take a look at `vector`'s ctor 
``` C++
template <typename T>
vector<T>::vector<T>() : n{0}, cap{1}, theVector{new T[cap]} {}
//                                              ^~~~~~~~~~~~
//                                              here T = Posn 
//                                              but posn doesn't have default ctor 
```

We need to separate memory allocation (step 1 in 4 step object creation) from object initialization (step 2 - 4)

## Allocation: 
=> `void* operator new(size_t)`
- allocates size_t bytes 
- no initialization 
- returns void*
- in C, `void *` implicitly converts to any pointer type 
- in C++ it is not true, the conversion requires a cast 

## Initialization: "placement new"
=> `new (addr) type`
- it constructs a `type` object at `addr` and does not allocates memory 



``` C++
template <typename T>
class vector {
// ...
public:
    vector() : n{0}, cap{1}, theVector{static_cast<T*>(operator new(sizeof(T)))} {}
    //                                                ^~~~~~~~~~~~~~~~~~~~~~~~~~
    //                                                 this returns void *
    //                                              therefore explicit cast is needed 

    vector(size_t n, T x = T{}) 
    : n{n}, cap{n}, theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
        for(size_t i=0; i<n; ++i) {
            new(theVector+i) T(x);
        //  ^~~~~~~~~~~~~~~~~~~~~
        //  we essentially use placement_new to invoke copy constructor 
        //  the point is we separate allocatation from filling (initialization)
        }
    }

    // ...
    void push_back(T x) {
        increaseCap();
        new(theVector + (n++)) T(x);
    }

    void pop_back() {
        if(n) {
            theVector[n-1].~T();
            //            ^~~~~
            //            explicit call to dtor 
            --n;
        }
    }

    ~vector() {
        destroy_items();
        operator delete(thisVector);
    }

private:
    void destroy_items() {
        for(auto &x : *this) { // assume we implemented iterator 
            x.~T();
        }
    }
};
```