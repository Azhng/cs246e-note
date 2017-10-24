# Problem 19: I want a mixture of types in my `vector`

This is a problem that we cannot solve using template 
we could try this, but it won't work 
``` C++
vector<template <typename T> T> v; 
```

This is not allowed - tempaltes are compile-time 

E.g. fields of a struct 
``` C++
class MediaPlayer {
    template <typename T> T nowPlaying; // not allowed either 
};
```

## What is available in C: 
- `union`s 
``` C
union Media {
    song s; 
    Movie m;
};

Media nowPlaying;
```

Disadvantage of `union`: `union` does not remember type, one can save a song and retrieve it as movie 


- `void *` 
``` C 
void *nowPlaying; 
```

This is worse, `nowPlaying` can point to ANYTHINg 

### Conclusion: all the options in C are not type safe 

Note:
- items in a heterogenous collections usually have somthing in common, e.g. provide a common interface 
- can be viewed as different kinds of a more general "thing"
- So have a `vector` of "thing"s or a field of type "thing"


``` C++
class Book { // this is superclass or base class
    string title, author;
    int length; 

public:
    Book(string title, string author, int length) :
        title{title}, author{author}, length{length} {}

    bool isHeavy() const {
        return length > 100;
    }

    string getTitle() const { return title; }

    // ... 
};
```


In Memory: 
```
 ________
| title  |
| author |
| length |
 --------
```

But some of my book is special
``` C++
class Textbook : public Book { // subclass or derived class 
    string topic;

public:
    Textbook(string title, string author, int length, string topic)
        : Book(title, author, length), topic{topic} {}

    string getTopic() const { return topic; }

    bool isHeavy() const {
        return length > 500;
    }

};
```

In Memory (Textbook): 
``` 
 ________
| title  |
| author |
| length |
| topic  |
 --------
```


Now suppose we have comic books:
``` C++
class Comicbook : public Book {
    string hero;

public:
    Comicbook(string title, string author, int length, string hero)
        : Book(title, author, length), hero{hero} {}
    
    string getHero() { return hero; }

    bool isHeavy() const {
        return length > 50;
    }

};
```



In Memory (Comicbook): 
``` 
 ________
| title  |
| author |
| length |
| hero   |
 --------
```




Note:
- `subclass`es inherit all members (fields and methods) from their `superclass`
- all 3 classes have `title`, `author`, `length`, `isHeavy()`, `getTitle()`, `getTitle()`, `getLength()`
- except that this doesn't work 
    - because:

``` C++
bool Textbook::isHeavy() const { return length > 500; }
//                                      ^~~~~~~
//                                      this is a private in Book, Textbook cannot access it ?
``` 


We now have 2 options: 
1. Modify `Book`
``` C++
class Book {
    string title, author;

protected: // accessible only to this class and its subclasses 
    int length;

public:
    // ...
};
```

2. use public accessor method instead of using the field name 
``` C++
bool Textbook::isHeavy() const {
    return getLength() > 500;
}
```


Note: 
- we prefer using option 2
- `protected` weaken encapsulation
- since with option 1, we cannot enforce class invariant to be preserved by subclasses
- If we ever want subclasses to have priviledged access 
    - keep fields private 
    - provide `protected` mutator methods 
        - e.g. `get____` and `set____` 
    - we still preserve the ability to enforce class invariant



Now with inheritance, object creation / destruction protocols needs to be updated 

- Construction
    1. Space is allocated 
    2. Superclass is constructed 
    3. Fields constructed in delcaring order 
    4. `ctor` body runs 


- Destruction 
    1. `dtor` body runs 
    2. Fields destructed in reverse declaring order 
    3. Superclass is destructed 
    4. Space is deallocated 

We must revisit everything we know to see theeffect of inheritance

## Type compatibility
- `Textbook` and `Comicbook` are special kind of `Book`s 
    - should be useable in place of `Book`s 
    E.g.
``` C++
    Book b = Comicbook{"", "", 75, ""}; // this is ok 
    b.isHeavy(); 
    // Question : is the result of above expression true or false ? 
    // 75 -> heavy comic, but not heavy book 
    // Anser: False 
```



If `b` is a `Comicbook`, why is it acting like a `Book`? 
- Because `b` is a `Book`! 
- this is the consequence of having stack allocated objects: 
``` C++
Book b = Comicbook{...}; // compiler gives enough space to hold the book 
                         // RHS is a rvalue 
                         // we can't really fit Comicbook (4 fields) in Book (3 fields)
                         // we only get the book part, the comic part is being chopped off 
                         // a.k.a. "slicing"
```

- So right now `b` is really just a `Book` now instead of `Comicbook` 
- slicing happens even if superclass and subclass are the same size 


Similarly, if you want to collect your books:
``` C++
vector<Book> library;
library.push_back(Comic{/*...*/});
```

- this is going to work, but the book-part will be pushed 
- this is not really heterogenous collection, it's only a collection of books 

Also note:
``` C++
void f(Book book s[]); // raw array 
Comicbook comics[] = {/*...*/};
f(comics); // perfectly legal 
           // perfectly trouble 
```
- when loop through `s`, compiler will generate code to loop through array by the size of `Book` rather than `Comicbook`
- therefore the array will be misaligned
- will not act like array of Books 
- Undefined behaviour 
- but slicing does not happen through pointers 

Observe the following code:
``` C++
Book *p = new Comicbook{"", "", 75, ""};
p->isHeavy(); // still False 
```

Rule:
- the choice of which `isHeavy()` to run is based on the type of the pointer (static type), not the object (dynamic type)
- Why:
    - because it is cheaper 
    - the underyling design principle of C++: if you don't use it, you shouldn't have to pay for it 
        - i.e. if you want something more expensive, you have to ask for it 



To make `*p` act like a `Comicbook` when it is a Comicbook:
``` C++
class Book {
// ...
public:
    // ...
    virtual bool isHeavy() const { /*...*/}
//  ^~~~~~~
//  can be overidden 
//  once a virtual 
//  always virtual 
};

class Comicbook {
// ...
public:
    bool isHeavy() const override {/*...*/}
};

void f() {
    Book *p = new Comicbook("", "", 85, "");
    p->isHeavy(); // True 
}
```

Note:
- `override` here is not a keyword, it is a _contextual keyowrd_, only has meaning when declared after class methods 



Assume `isHeavy` is `virtual` 















