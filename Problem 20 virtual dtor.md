# Problem 20: I'm Leaking 

``` C++
class X {
    int *a;
public:
    X(int n) : a {new int[n]} {}
    ~X() {
        delete[] a;
    }
};

class Y : public X {
    int *b;

public:
    Y(int n, int m) {X{n}}, b{new int[m]} {}
    ~Y() {
        delete[] b;
        // note Y's dtor will call X's dtor 
    }
}


void f() {
    X *px = new Y{x, 4};
    delete px; // leak, because X's dtor gets ran 
}
```

What we need here is a __virtual__ destructor 

``` C++
class X {
// ...
public:
    virtual ~X() { delete[] a; }
}
```

- No more leaks 
- __Always__ make the dtor virtual in classes that are meant to be superclasses 
    - even if the dtor does nothing, we don't know what the subclass might do 
    - so we need to make sure its dtor gets called 
- if a class is not meant to be a superclass, 
    - then not need to create virtual dtor which incurs the cost of virtual methods needlessly 
    - leave the dtor non-virtual, but declare the class final 

``` C++
class X final {  // cannot be inherited 
//      ^~~~~
//      contexual keyword, just like override 
};
```



















