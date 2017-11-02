# Problem 23 - I want to know what kind of `Book` I have ?

For simplicity, assume we use the original object hierachy

```
       |Book|
         ^
   ------|-------
  |              |
|Text|       |Comic|
```


## The C++ casting operators - 4 operators 

Recall C-style casting: `(type)expr)` 
- forces `expr` to be treated as type `type`
    - e.g. 
        - `int *p;`
        - `int q = (int)p;`
    - this is potentially a huge source of bug in C 


in C++
1. `static_cast` 
    - for conversion with a well-defined semantics 
    - E.g. explicit primitive type conversion 
``` C++
void f(int a); 
void f(double d);
int x;
f(static_cast<double> x);
```
    - E.g. super class pointer to subclass pointer 
        - in this case programmers need to have absolute certainty that `b` is pointing to a `Text` 
        - otherwise it is undefined behaviour 
``` C++
Book *b = new Text{/*...*/}; // allowed 
Text *t = static_cast<Text*>(b);
```

2. `reinterpret_cast` 
    - for casts without a well-defined semantics 
    - unsafe, implementation-dependent 
    - we only know if we cast it to something else then cast it back, we get the same pointer 
    - see `reinterpret.cc` in lecture note for more example 
    - simple example:
``` C++
Book *b = new Book{/*...*/};
int *p = reinterpret_cast<int*>(b); // reinterpret Book as an int array 
                                    // usually to get memory dump 
```

3. `const_cast`
    - for adding / removing `const`
    - the only C++ `const` that can "cast away `const`"
    - E.g.
``` C++
void g(Book &b);

// if we happen to know that g doesn't actaully modify the Book reference 
// and g does exactly what f does 
void f(const Book &b) {
    g(const_cast<Book &>(b));
}
```

4. `dynamic_cast`
    - what if we __don't__ know whether `pb` points at a `Text`? 
    - `static_cast` is not safe 
    - it also works with references 
``` C++
Book *pb = /*...something...*/
Text *pt = dynamic_cast<Text*>(pb);
```
    - if `*pb` is a `Text` or a class of `Text`
        - cast succeeds, `pt` points at the object 
    - else `pt = nullptr;`
        - E.g.
``` C++
if(pt) { cout << pt->getTopic() << endl; }
else /*... something else ... */
```
    - E.g.
``` C++
void whatIsIt(Book *pb) {
    if(dynamic_cast<Text*>(pb)) {
        cout << "Text";
    } else if(dynamic_cast<Comic*>(pb)) {
        cout << "Comic";
    } else {
        // we assume it is not null 
        cout << "Book";
    }
}
```
    - note that this is not a good style 
        - what happens when you create a new book type ? 
    - dynmaic reference casting:
``` C++
Book *pb = ??????;
Text &t = dynamic_cast<Text&>(*pb); 
// if *pb is Text - OK
// else           - raises std::bad_cast 
```
    - Note: dynmaic casting works by accessing an objects Run-Time Type Information (RTTI)
        - usually RTTI information is stored in the Vtable for the class 
        - this means you can only cast object with at least one virtual method 

## Dynamic reference casting offsers a possible solution to the polymorphic assignmnet problem (back in 22): 
- in Problem 22 we solved it by replacing concrete super class with abstract super class 
- now with dynamic_cast
``` C++
Text& Text::operator=(Book other) { // virtual 
    Text &textOther = dynamic_cast<Text&>(other); // throws if other is not a Text
    Book::operator=(std::move(textOther));
    topic = std::move(textOther);
    return *this;
}
```











