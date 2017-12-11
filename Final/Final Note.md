
## SFINAE: 

SFINAE: Substitution Failure Is Not An Error 

### Q1 
``` C++
template <typename T>
T create_T() {
    if (std::is_default_constructible<T>::value) {
	return T{};        
    } else {
	return T{1};
    }
}
```

Solution: 

``` C++
template <typename T>
T create_T_helper(typename std::enable_if<std::is_default_constructible<T>::value, int>::type arg) {
    return T{};
}

template <typename T>
T create_T_helper(typename std::enable_if<!std::is_default_constructible<T>::value, int>::type arg) {
    return T{1};
}

// Question: What if we define a useless local var with `enable_if` instead

template <typename T>
T create_T() {
    return create_T_helper<T>(0);
}
```

### Q2

``` C++
#include <iostream>
#include <iomanip>

template <typename...> using ignore_args = void;

template <typename, typename T>
struct is_default_constructible_helper {
    static const bool value = false;
}; 

// Begin of extra code 


// we want it to fail to compile if `T` has a defualt constructor
// since we are in the SFINAE section
// - if `T()` fails, then this template is ignored 
// - if `T()` succeed then this template will successfully instantiated

template <typename T>
struct is_default_constructible_helper<ignore_args<decltype(T())>, T> {
    static const bool value = true;
};

// End of extra code


template <typename T>
struct is_default_constructible
    : public is_default_constructible_helper<ignore_args<>, T> {};

class N {
public: 
    N(int i) {}
};

int main() {
    std::cout << std::boolalpha;
    std::cout << is_default_constructible<int>::value << std::endl;
    std::cout << is_default_constructible<N>::value << std::endl;
    return 0;
}

```

## Question 1-6
Suppose: 

``` C++
class C {};
class B {};
class D : public B, public C {};
```

`static_cast` adjust pointer position in `D` whereas `reinterpret_cast` does not


## Question 5 

``` 
C f()       A virtual g(C *) g(D *)
^           ^ 
|           |
D f()       B virtual g(C *) g(D *)
```


``` C++
C *p = new D;
p->f(); // C's method gets called 
```

However if `f()` is defined virtual in `C` then `D`'s method will be called 

In this question, 
```
        |Host (Traversal)|
                ^        // virtual void accept(OneVisitor *p)
                |        // virtual void accept(AnotherVisitor *p)
    -----------------------------
   |                             |
|OneHost|                    |AndHost|

        |Visitor (Expression)| 
                ^         // virtual void visit(Host *)
                |     
    _____________________________
   |                             |
|OneVisitor|                     |AnotherVisitor|
```

In code: 

``` 
// Inside Visitor 
virtual void OneVisitor::visit(Host *p) {
    p->accept(this); // the type of `this` depends on the function call 
}

Visitor *myVisitor = /*...*/;
Host *myHost = /*...*/;
myVisitor->visit(myHost);
```


``` C++
class Expression {
public:
	virtual void accept(Traversal* t) = 0;
};

class Plus : public Expression {

public:
	
	virtual void accept(Traversal* t) override {
		t->traverse(this);
	}

};

class Variable : public Expression {

public:
	
	virtual void accept(Traversal *t) override {
		t->traverse(this);
	}

};

class Traversal {

public:
	
	virtual int traverse(Plus* e) = 0;

	virtual int traverse(Variable* e) = 0;

};


class EvaluateTraversal : public Traverse {

	int val = 0;

public:
	
	virtual int traverse(Plus* e) override {
		int tval = 0;
		e->getLeft->accept(this);
		tval = val;
		val = 0;
		e->getRight->accept(this);
		val = val + tval;
	}

};

// Variable left as exercise
```




## Question 7

### Part 2

Destructor is wrong. it does not delete arrays

### Part 3

``` C++
template <typename T>
class unique_ptr<T[]> {
/* ... */
};
```


### Part 4

``` C++
template <typename T>
unique_ptr<T[]> make_unique(size_t num) {
	return unique_ptr<T[]>(new T[num]);
}
```





