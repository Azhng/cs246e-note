# Problem 7: I want a constant vector 

Say we want to print a vector: 


Observe that following code snippet won't even compile ! 
``` C++ 
ostream& operator<<(ostream &out, const vector &v) {
    for(size_t i=0; i<v.size(); ++i) {
        out << v.itemAt(i) << " ";
    }
    return out; 
}
```

We can't call __size()__ and __itemAt()__ on a const object, because those methods might change fields.

but since they don't perform any mutations, we should declare them const 

``` C++
struct vector {
    // ...
    size_t size() const; 
    //            ^~~~~~~~~~~~
    // this means the these methods will not modify fields, that is they can be called on const objects 
    int& itemAt(size_t i) const; 
};
```

Implementation: 
``` C++
size_t vector::size() const {
    return n;
}

int& vector::itemAt(size_t i) const { 
    return theVector[i];
}
```

Now the loop will work 

__BUT__:  
``` C++
void f(const vector &v) { 
    v.itemAt(0) = 4; // this WORKS ! 
                     // WTF, Ikr 
}
```
This is because `v` is a const object, we cannot change `n`, `cap`, `theVector` (ptr), but we can change items pointed ot by `theVector` 

Can we fix this? 

``` C++
struct vector {
    // ...
    const int& itemAt(size_t i) const; 
};

const int& vector::itemAt(size_t i) const {
    return theVector[i];
}
```

Now, `v.itemAt(0) = 4;` won't compile if `v` is `const`, but the issue is it won't compile if `v` is NOT `const` 

To fix: `const` overloading 

``` C++
struct vector {
    // ...
    const int& itemAt(size_t i) const; // called if object is const 
    int& itemAt(size_t i);  // called if object is not const 
}

inline const int& vector::itemAt(size_t i) const {
//^~~~~
// this is only a suggestion to compiler to replace function call with function body
// compiler can reject this suggestion 
// when declare inline function, we usually write the definition in the header 
    return theVector[i];
}

inline int& vector::itemAt(size_t i) {
    return theVector[i];
}
```

`inline` function is a good idea for small functions 

now `v.itemAt(0) = 4;` will compile if and only if `v` is non-const 


Now let's make it prettier:
``` C++
struct vector {
    // ...
    size_t size() const { return n;}
    const int& operator[](size_t i) const { return theVector[i]; }
    // define function inside class implicitly suggest compiler to make it inline 
    int& operator[](size_t i) { return theVector[i]; }

}
```


Let's go back to our original discussion 
``` C++ 
ostream& operator<<(ostream &out, const vector &v) {
    for(size_t i=0; i<v.size(); ++i)) {
        out << v[i] << " ";
    }
    return out;
}
```