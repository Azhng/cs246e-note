# Probelm 21 - The copier is broken 

How do copies & moves interact with inheritance 

Copy ctor: 
- `Textbook::Textbook(const Textbook &other) : Book{other}, topic{other.topic} {}`

Move ctor: 
- `Textbook::Textbook(Textbook &&other) : Book{std::move(other)}, topic{std::move{other.topic}} {}`

Copy/move assignment:
``` C++
Textbook& Textbook::operator=(Textbook other) {
    Book::operator=(std::move(other));
    topic = std::move(other.topic);
    return *this;
}
```

But consider:
``` C++
Book *b1 = new Textbook{"Some dude", "", "", "Basic"}, *b2 = new Textbook{"Stroustrap", "", "", "C++"};
(*b1) = (*b2);
// What happens ?
// - only the Book part is being copied  
//   - partial assignment 
//   - *b1 becomes Textbook("Stroustrap", "", "", "Basic")
// - topic doesn't match title and author - object is currupted 
```

Possible solution
- make `operator=` `virtual` 
    - however it is not correct 
    - following code snippet won't even compile 
    - since the method signature of the `operator=` in `Text` class does not match the one in `Book`
        - or it is not a override 
``` C++
class Book {
// ...
public:
    virtual Book& operator=(const Book &other);

};

class Textbook : public Book {
// ...
public:
    Textbook& operator=(const Textbook &other) override;
    //                    ^~~~~
    //                   it must be Book here to be a legal override
}
```

    - to fix this
``` C++
class Book {
// ...
public:
    virtual Book& operator=(const Book &other);

};

class Textbook : public Book {
// ...
public:
    Textbook& operator=(const Book &other) override;
    //                    ^~~~~
    //                    then we can potentially assign a Textbook with Comicbook 
}
```


## Question: How do we avoid corruption?
- we need to remove the problem 
    - a.k.a. get rid of Book
    - or we can make all super classes **abstract**
```
Original design:

       |Book|
         ^
   ------|-------
  |              |
|Text|       |Comic|


Better design:

      |Abstract Book|
            ^
   ---------|----------------
  |         |               |
|Text| |NormalBook|      |Comic|
```

- To implement:
``` C++
class AbstractBook {
// ...
protected:
    AbstractBook& operator=(AbstractBook other) {/*...*/} // non-virtual
                                                          // unified assignment 
};

class Textbook : public AbstractBook {

public: 
    Textbook& operator=(Textbook other) {
        AbstractBook::operator=(std::move(other));
        topic = std::move(other.topic);
    }
};

```
- new implmentation works because 
    - `operator=` in `Texbook` is non-virtual 
        - therefore no mixed assignment 
    - no partial assignment 
        - it is impossible now to invoke super class's assignment operator 
        - since it is now  `protected`
- Therefore: `*b1 = *b2` won't even compile 















