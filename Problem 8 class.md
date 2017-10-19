# Problem 8: Tampering

``` C++
vector v;
v.cap = 100; // sets cap w/o allocating memory 
             // this is undefined behaviour, very likely to crash 
v.push_back(1);
v.push_back(10);
```


Interfering with ADTs 
- Forgery - creating an object w/o using a ctor function. 
    - this becames impossible once we wrote ctors. 
- Tampering - accessing the internal without using provided interface functions 
    - What's the big deal? - __Invariants__
    - __Invariants__ - statement that will always be true about an abstraction 

ADTs provide and rely on invariants 
- stacks 
    - provide the invariant that the last item you pushed is the first item pops 
- vectors 
    - reply on the invariant that elements 0 ... cap -1 denotes valid locations 

We cannot guarantee the invariants if user interferes and makes the program hard to reason about 


__Fix__: **Encapsulation** - seal objects into black box 

``` C++
struct vector {
private: // these fields are accessible only within the vector class 
    size_t n, cap;
    int *theVector; 
public:
    vector();
    size_t size() const;
    void push_back(int n);
    // ... 
}
```

If no access specified given, it is going to be `public`

``` C++
#include "vector.h"
namespace {
    // this function doesn't work anymore 
    // because this function is modifying `cap` which is now private 
    void increaseCap(vector &v) {
        //...
    }
}
```

Let's try again
``` C++
// vector.h
struct vector {
private: 
    size_t n, cap;
    int *theVector;

public:
    vector();
    // ... 

private:
    void increaseCap(); // now a private method 
}


// vector.cc
namespace CS246E {
    vector::vector() {...}
    // ...
    void vector::incraseCpa() {...}
   
}
```

We no longer need anonymous namespace anymore!

For `struct` - default public access 

For `class` - default private access 


``` C++
class vector {
    size_t n, cap;
    int *theVector; 

public: 
    vector();
    // ... 

private: 
    void increaseCap();
}

// vector.cc

// nothing changes 
```

Now observe that we are having a similar problem with linked list 
``` C++
Node n(3, nullptr); // stack-allocated 
Node m(4, &n);      // BOOM 
                    // m's dtor will try to delete &n - undefined behaviour 
```

There is a invariant that we are relying on here, `next` is either a `nullptr` or was allocated by `new` 

Question: how can we enforce this?
- we will encasulate `Node` inside a "wrapper" class 

``` C++
class list {
    // observe that class essentially provides extra layer of scope 
    
    // we can now have a private nested class 
    // - not available outside of class list 
    struct Node {
        int data;
        Node *next;
        // ... methods/ctor/dtor
    };

    Node *theList;
public:

    list() : theList{nullptr}

    ~list() {
        delete theList;
    }

    size_t size() const; 

    void push_front(int n) {
        theList = new Node(n, theList);
    }

    void pop_front() {
        if(theList) {
            Node *tmp = theList;
            theList = theList->next;
            tmp->next = nullptr; // make sure chain destruction does not happen
            delete tmp; 
        }
    }

    const int& operator[](size_t i) const {
        Node *cur = theList;
        for(size_t j=0; j<i && cur; ++j) cur=cur->next;
        return cur->data;
    }

    int& operator[](size_t i) {/*...*/}
}
```

Client cannot manipulate the list directly 
- no access to next pointers 
- invariant is maintained

