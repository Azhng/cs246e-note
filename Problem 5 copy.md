# Problem 5: Copies 

``` C++
  Vector v;
  v.push_back(100);
  // ...
  Vector w = v; // allowed - constants w as a copy of v
  w.itemAt(0); // 100
  v.itemAt(0) = 200; 
  w.itemAt(0); // 200, shallow copy - `v` and `w` shares data 
               // this invokes "copy constructor"
```


``` C++
struct Vector {
  // ... 
  Vector(const Vector &other) {/**/} // copy ctor 
  // this is the compiler-supplied copy-ctor 
  // copy all fields 
  // shallow copy
};
```


If we want to have deep copy, we need to write our own copy constructor 

``` C++ 

strcut Node {
  int data;
  Node *next; 
  // ... 
  Node(const Node &other) : data{other.data}, next{other.next? new Node(*other.next) : nullptr} {}                   // ^~~~~~~~~~
                              // recursively calls copy constructor
  // we cannot pass by value here 
  // since copy by value need to make a copy of the object 
  // which requires copy ctor   
};

```

However, consider following code:
``` C++
Vector v;
Vector w;
w = v;  // copy, but not a construction 
        // copy assignment operator 
        // compiler-supplied: copies each field (shallow)
        // and memory leak occurs since it's not a destruction but an assignment 
        // the destructor is not called 
        // hence we need to implement copy assignment operator 
```

Deep copy assignment: 

``` C++
struct Node { // vector version is left as exercise
  // ...
  Node& operator=(const Node &other) {
//^~~~~
// it is a reference here to give expressions a return value 
// e.g. a = b = c = d
    data = other.data; 
    // next = other.next; would create a shallow copy 

    // important! we need to free the next pointer here if it is already allocated 
    delete next; 
    next = other.next ? new Node(*other.next) : nullptr;
    return *this;
  }
}; 
```

However, above implementation is wrong incredibly dangerous 

Consier: 
``` C++
Node n(/*...*/);
n = n; 
^~~~~
Boom! totally undefined behaviour
``` 

In real life, this actually happens in the form of pointers or loop

E.g. 
``` C++
*p = *q;
a[i] = a[k];
```

That is, we must always ensure that operator `=` works in the case of self-assignment 

``` C++
Node& Node::operator=(const Node &other) {
  if(this == &other) return *this;
  // same as before
}
```

Alternative: copy-and-swap idiom 

``` C++
#include <utility>

struct Node {
  // ...
  void swap(Node &other) {
    using std::swap; 
    swap(data, other);
    swap(next, other.next);
  }

  Node& operator=(const Node &other) {
    Node temp = other; // creating a deep copy of other 
                       // assuming copy constructor is implemented 
    swap(temp);        // swap old data and the new data 
    return *this;      // return new data, temp goes out of scope and gets destroyed
  }
};
```

