# Problem 30: Logging 

- want to encapsulate logging functionality and "add" it to any class 

``` C++
template <typename T, typename Data> 
class Logger {

public:
    
    void loggedSet(Data x) {
        cout << "setting data to " << x << endl;
        static_cast<T*> (this)->set(x);
//      ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
//      reuseable 
//      no virtual call overhead  
    }

};

class Box : public Logger<Box, int> {
    friend class Logger<Box, int>;

    int x;

    void set(int y) { x = y; }

public:

    Box(): x{0} {
        loggedSet(0);
    }

};


void f() {
    Box b;
    b.loggedSet(1);
    b.loggedSet(4); 
    // ... 
}
```


we can do this using another approach:

``` C++
class Box {
    int x;

public:
    Box() : x{0} {}

    void set(int y) { x = y; }
};

template <typename T, typename Data>
class Logger : public T {

public: 
    void loggedSet(Data x) {
        cout << "Setting data to " << x << endl;
        set(x);
    }

};


```

## `Mixin`s - Can mix and match subclass functionality without writing new classes 

- Note
    - if `SpecialBox` is a subclass of `Box`, then `SpecialBox` has no relation to `Logger<Box, int>` Nor is there any relationship between `Logger<SpecialBox, int>` 
    - but with CRTP, `SpecialBox` is a subtype of `Logger<Box, int>` 
        - we can specialize behaviour of virtual functions 


