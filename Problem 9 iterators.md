# Problem 9 - Efficient Iteration

``` C++
vector v;
v.push_back(/*...*/);
// ...
for(size_t i=0; i<v.size(); ++i) {
    cout << v[i] << endl; // O(1) array access 
                          // efficient 
                          // but if we use list instead of vector 
                          // then it is going to be O(n) access 
                          // then it is O(n^2) traversal 
}
```

If we don't have direct access to "next" pointers - how can we do efficient iteration ?


## Design Pattern
- well-known solutions to well-studied problems 
- adapted to suit needs 

### Iterator Pattern 
- for specific iteration over a collction without exposing the underlying structure 

Idea: 
- create a class that "remembers" where you are in the list 
- abstraction of a pointer 


Inspiration: pointer arithmetic in C
``` C
for(int *p=arr; p != a+size; ++p) {
    printf("%d\n", *p);
}
```

Implementing List
``` C++
class list {
    struct Node {/*...*/};
    Node *theList;
public:
    /*...*/
    class iterator {
        Node *p;
    public: 
        iterator(Node *p) : p(p) {}

        bool operator!=(const iterator &other) const {
            return p != other.p;
        } 

        int& operator*() {
            return p->data; 
        }

        iterator& operator++() { // this is prefix operator 
            p = p->next;         // if we through in a dummy int param
            return *this;        // to overload post-fix operator 
        }
    };

    // observe that these functions are inlined 
    iterator begin() {
        return iterator(theList);
    }

    iterator end() {
        return iterator(nullptr);
    }
}

int main() {
    list l;
    for(list::iterator it=l.begin(); it != l.end(); ++it) {
        cout << *it << endl; // this is constant time read 
    }
}
```



## Question: Should `list::begin`, `list::end` be const method ?
Consider: 

if `begin` and `end` are non-const 
``` C++
ostream& operator<<(ostream &out, const list &l) {
    for(list::iterator it=l.begin(); it!=l.end(); ++it) {
        //             ^~~~~~~~~~~~~~~~~~~~~~~~~
        //            now observe that `begin()` and `end()` are not const 
        //            thus the function won't even compile
        // ...
    }
    // ...
}
```

If `begin` / `end` are const: 
``` C++
ostream& operator<<(ostream &out, const list &l) {
    for(/*...*/) {
        out << *it << endl;
        ++*it; // increment items in the list 
        //^~~
        //this will compile 
        //but shouldn't, because list is supposed to be const 
        //this compiles because operator * returns a non-const ref 
        //should compile if l is non-const
    }
}
```

Conclusion: 
- iteration over `const` is different from iteration over `non-const` 
- we need to make a second iteratior class 

``` C++
class list {
    //...
public:
    class iterator {
        // ...
    public: 
        int& operator*() const {
            // ...
        }
    };

    class const_iterator {
        // ...
    public: 
        const_iterator(Node *p) : p(p) {};
        
        bool operator!=(const const_operator &other) {
            return p != other.p; 
        }

        const_iterator& operator++() {
            p = p->next;
            return *this;
        }

        const int& operator *() const {
      //^~~~~
      // constant 
            return p->data;;
        }
    };

    iterator begin() {
        return iterator(theList); 
    }

    iterator end() {
        return iterator(mullptr);
    }

    const_iterator begin() const {
        return const_iterator(theList);
    }

    const_iterator end() const {
        return const_iterator(nullptr);
    }

    // above 4 fnuctions are essentially const overloading


    ostream& operator<<(/*...*/, const list &l) {
        for(list::const_iterator it=l.begin(), it != l.end(); ++it) {
            out << *it << endl;
        }
        return out;
    }
};

```


Shorter version:
``` C++
ostream& operator<<(/*...*/) {
    for(auto it=l.begin(), it != l.end(); ++it) {
        out << *it << endl;
    }
    return out; 
}
```


Observe that `auto x = expr;` 
- saves writing down x's type 
- x will be given expr's type 


Even shorter version:
``` C++
ostream& operator<<(/*...*/) {
    for(auto n : l) { // range based for-loop
        out << n << endl;
    }

    return out;
}
```

### Range-based for-loop
- available for any class with methods (or functions) `begin` and `end` that returns an iterator object (it doesn't have to be called object)
- the iterator class must support unary `*`, prefix `++` and `!=` operators 

## Note: 
``` C++
for(auto n : l) ++n; // n is declared by value 
                     // `++n` increments n, not the list item 

for(auto &n : l) ++n; // n is a ref
                      // will update list elements 


for(const auto &n : l) /*...*/; 
                         // const ref
                         // can't be mutated
```


### One small encapsulation problem:
client: 
``` C++
list::iterator it(nullptr);
// forging 
// - create an end iterator without calling end 
```

One way to fix this, is to make `iterator` ctors private 

But `list` class won't be able to create `iterator` either 

Solution: to grant `list` priviledge to `iterator` ctors 

## Friendship

``` C++
class list {
    // ...
public:
    class iterator {
        // ...
        iterator(Node *p) {}   
    public: 
        // ...
        friend class list; 
        // list has access to all of iterator's/const_iterator's implementations 
    };

    class const_iterator {
        // ...
        const_iterator(Node *p) {}
    public:
        // ...
        friend class list;
    };
};
```

NOTE: in the scope of C++, do limit friendships
- because they weaken encapsulation


We can do similar operation for linked list in vector 
``` C++
class vector {
    size_t size, cap;
    int *theVector;
public:
    // ...
    class iterator {
        int *p;
        // ...
    };

    class const_iterator {
        // ...
    };
    
    iterator begin() { return iterator(theVector); }
    iterator end() { return iterator(theVector + n); }

    const_iterator begin() const {return const_iterator(theVector); }
    const_iterator end() const {return const_iterator(theVector + n); }
};
```

or we can directly abstract the pointer into iterator using `typedef` 

``` C++
// the ancient way
class vector {
    size_t size, cap;
    int *theVector;
public:
    // ...
    typedef int * iterator;
    typedef const int * const_iterator; 

    iterator begin() { return theVector; }
    iterator end() { return theVector + n; }

    const_iterator begin() const { return theVector; }
    const_iterator end() const { return theVector + n; }
};
```

``` C++
// the modern way
class vector {
    size_t size, cap;
    int *theVector;
public:
    // ...
    using iterator = int*;
    using const_iterator = const int *;
    
    iterator begin() { return theVector; }
    iterator end() { return theVector + n; }

    const_iterator begin() const { return theVector; }
    const_iterator end() const { return theVector + n; }
};
```