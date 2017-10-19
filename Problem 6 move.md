# Problem 6: Moves
Consider 

``` C++
Node plusOne(Node n) {
    for(Node *p=&n; p; p=p->next) {
        ++p->data;
    }
    return n;
} 
```


``` C++
Node n(1, new Node(2, nullptr));
Node m = plusOne(n); // copy ctor 
         ^~~~~~~~~~
         but what is "other" here ? - reference to what?         
```

- **temporary object**: created to hold the result of `plusOne`
- **other** is a reference to this temporary object 
- copy ctor deep-copies the data from this temporary object 

## BUT
- the temporary is just going to be thrown out anyway, as soon as the statement `Node m = plusOne(n);` is done. 
- Why would we waste time to copy from something that is going to be thrown out later. 
- Why not just steal the data instead? saving the cost of copying 
- Need to be able to tell whether other is a ref to a temporary obj or a standalone object 

## **Rvalue** references / Moving ctor
- `Node &&` is a reference to a temporary object (`rvalue`) of type Node. 
- We want a version of the constructor that takes a `Node &&` 

``` C++ 
struct Node {
    // ...

    // this is called a `move` constructor 
    // to "steal" `other`'s data 
    Node(Node &&other) : data{other.data}, next{other.next} {
        ohter.next = nullptr; // we literally "steal" everything from `other` 
                              // so when other is destructed, the `next` is not  
                              // also being destroyed 
    }
}; 
```


Now consider this:
``` C++
Node m;
m = plusOne(n); // assignment from temporary, same issue 
```

Now we have the same issue with assignment operator (`=`)

## Move assignment operator 
``` C++
struct Node {
    // ...
    Node& operator=(Node &&other) { 
        // we need to steal other's data 
        // and our own data need to be destroyed 
        // best way? give it to other, since other is going to be destroyed anyway
        using std::swap;
        swap(data, other.data);
        swap(next, other.next);
        return *this;
    }
};
```

We can actually combine copy/move assignment: 
``` C++
struct Node {
    // ...

    // unified assignment operator
    // we are using pass-by-value 
    // it triggers a copy 
    // it invokes copy ctor if other is an lvalue 
    // it invokes move ctor if other is an rvalue 
    Node& operator=(Node other) {
        swap(other);
        return *this;
    }
};
```

Note: 
- copy/swap can be more expensive
- hand-coded operators may do less copying 


But now consdier: 
``` C++
struct Student {
    std::string name; 
    
    Student(const std::string &name) : name{name} {}
    //                            ^~~~~~~~~
    //                           this copies the nae into the field (copy ctor)
    // but what happens if the string passed in is a `rvalue` ? (e.g. string literal)
};
```

Then consider: 

``` C++
struct Student {
    std::string name; 

    Student(std::string name) : name{name} {}
    // copies if name is an lvalue 
    // moves if name is an rvalue 
};
```

``` C++
struct Student {
    std::string name;
    Student(std::string name) : name{std::move(name)} {}
//             ^~~~~~~~~~~~~~~
//             this may come from an rvalue, but it is an lvalue 
//             std::move forces name to be treated as an rvalue 
//             hence make it effectively a string's move ctor rather than copy ctor 
}
```

``` C++
struct Student {
    Student(Student &&other) : name{std::move(other.name)} { }
    //              ^~~~~~~
    //              it is pointing at an rvalue 
    //              but itself at the moment is a lvalue 
};

```

``` C++
// unified assignment operator 
Student& operator=(Student other) {
                     //    ^~~~~
                     //    lvalue 
    name = std::move(other.name);
}
```

If you don't define `move` operations, `copy` ones will be used. 

If you do define them, they replace `copy` operations whenever the arg is a temporary (rvalue)


Recall: copies, moves, std::move

``` C++
// basis constructor 
vector makeAVector() {
    return vector{};
}

int main() {
    // copy/move elision 
    vector v = makeAVector(); 
    //         ^~~~~~~~~~~~~
    //         which constructor gets called? basic ctor? copy ctor? 
    //         or move ctor?
    //         Try in g++ - just the basic ctor - no copy/move 
}
```

In some circumstances, the compiler is allowed to skip calling copy/move ctors, (but doesn't have to)

`makeAVector` is writing its result directly into the space occupied by __`v`__, rather than copy/move it later 

It also explains this: 
``` C++
vector v = vector{};
       ^   ^~~~~~~~
       |   basic ctor 
       copy/move ctor
```
Here, though, compilers is __REQUIRED__ to elide/skip the copy/move ctor, therefore only basic ctor is called 

In an extreme case, 

``` C++
vector v = vector{vector{vector{}}};  // still only called basic ctor once 
```

E.g. 
``` C++
void doSomething(vector v) {
    //           ^~~~~~~
    //           this is passed-by-value
    //            - therefore copy/move ctor 
    //... 
}

// however
doSomething(makeAVector());
//          ^~~~~~~~~~~~~~~
//          - result of `makeAVector` is written directly into the parameter 
//            so no copy/move ctor calls 
//          - this is allowed EVEN IF DROPPING CTOR CALLS CHANGES THE BEHAVIOUR 
//            e.g. something is being printed during copy/move ctors 
```

If you need all of the ctors to run: 
``` bash
$ g++14 -fno_elide_constructors ~~~~~~~~
```
- this can slow down your program considerably 

In summary: __Rule of 5__ (Big 5)
- if you need to customize any one of 
    - copy ctor
    - copy assignment 
    - dtor
    - move ctor
    - move assignment 

    then you usually need to customize all 5 of them 