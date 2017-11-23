# Problem 29: Polymorphic Cloning 

``` C++
Book *pb = /*...*/;
Book *pb2 = // I want an exact-copy of *pb 
```


We can't tell a ctor directly since we don't know what `pd1` is 
- don't know which ctor to call 

Standard Solution: 
- virtual clone method 

``` C++
class Book {

// ...

public:
    
    virtual Book* clone() {
        return new Book {*this};
    }

};

class Text : public Book {

// ...

public:
    
    Text* clone() override {
        return new Text{*this};
    }

};


class Comic : public Book {

// ...

public:
    
    Comic* clone() override {
        return new Comic{*this};
    }

};
```

- we have a lot of boilerplate code 
- Can we reuse the code ? 

``` C++
class AbstractBook { 

public:
    
    virtual AbstractBook* clone() = 0;
    virtual ~AbstractBook();

};


template <typename T>
class BookClonable : public AbstractBook {

public:
    
    T* clone() override {
        return new T{ static_cast<T&>(*this); }
    }

};


class Book : public BookClonable<Book> {}
class Text : public BookClonable<Text> {}
class Comic : public BookClonable<Comic> {}

```


This is an instance of CRTP 





