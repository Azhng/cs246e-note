# More Design Patterns 

## Factor Method Patterns 
- when you don't know exactly what kind of object you want, and your preference may vary 
- also called the "Virtual Constructor Pattern"
    - basically strategy pattern applied to constructors 

## Example: 
```
                |Enemy|
                   ⧊
                   |
           -------------------
          |                   |
      |Turtle|             |Bullet|


                |Level|
                   ⧊
                   |
            ------------------
           |                  |
        |Easy|              |Hard|
```

- `Enemy` are randomly generated
- more `Turtle`'s in easy levels 
- more `Buullet` in hard levels 
- The `Level` class here is basically acting like our strategy 

``` C++
class Level {

public:
    
    virtual Enemy* getEnemy() = 0; // unique_ptr would make more sense here since it means ownership 

};

class Easy : public Level() {

public:
    Enemy *getEnemy() override {
        // mostly turtle 
    }

};


class Hard : public Level() {

public:
    Enemy *getEnemy() override {
        // mostly bullets 
    }

};

void f() {

    Level *l = new Easy;
    Enemy *e = l->getEnemy();

}


```



## Decorator pattern 
- add / remove functionailities to / from objects at runtime 
    - E.g.
        - add menus / scrollbars to windows 
            - either or both 
- avoid combinatorial explosion of subclasses 
```
    
                    |Component |   <-------------------
                    |+operation|                      |
                         ⧊                            |
                         |                            |
        ------------------------------------          |
       |                                   |          |
    |ConcreteComponent|                |Decorator| ♢--
    |+operation       |                     ⧊
// plain window                             |
                            -----------------------------------
                           |                                  |
                    |ConcreteDecorator-A|             |ConcreteDecorator-B|
                    |+operation()       |             |+operation()       |
                //   window with scrollbar             window with menu 

```

- Every Decorator IS a `component` AND HAS a `Component`
    - window with scrollbar is a kind of window AND HAS a pointer to the underlying plain window 
    - window with scrollbar and menu is a window AND HAS a pointer to a window with scrollbar, which HAS a pointer pointing to a plain window 
    - E.g.
``` C++
WindowInterface *w = new WindowWithMenu {
    new WindowWithScrollBar {
        new Window{}
    }
};
```
// see pizza example in repository 



## Visitor Pattern 
- for implementing *double dispatch* 
  - method chosen based on runtime types of 2 objects, not just one 

E.g.
```
      |Enemy|
         ^
         |
   ---------------
  |               |
|Turtle|       |Bullet|

  
      |Weapon| 
          ^
          |
   ---------------
  |               |
|Stick|         |Rock|


```

- effect of striking an enemy with a weapon depends on *both* the enemy *and* the weapon 
- C++
  - virtual methods are dispatched on the type of the reciever object, and *not* method params 
  - there is no way to specify two receiver objects 

_Visitor_:
- two stage dispatch, combining overriding with overloading 
``` C++

class Enemy {

public:
  
  virtual void beStruckBy(Weapon &w) = 0;

};


class Turtle : public Enemy {

public: 
  void beStruckBy(Weapon &w) override { w.strike(*this); } // this strike is different from strike below 

};

class Bullet : public Enemy {

public: 
  void beStruckBy(Weapon &w) override { w.strike(*this); } // this strike is an overloaded version of previous strike 

};

class Weapon {

public:
  
  virtual void strike(Turtle &t) = 0;
  virtual void strike(Bullet &b) = 0;

}

class Stick : public Weapon { 

public:

  void strike(Turtle &t) override {
    // strike Turtle with stick 
  }

  void strike(Bullet &t) override {
    // bulet with stick 
  }

};

void f() {
  Enemy *e = new Bullet {/*...*/};
  Weapon *w = new Rock{/*...*/};
  e->beStruckBy(*w); // what happens 
                     // - Bullet::beStruckBy runs, virtual method dispatch 
                     // - Weapon::strike(Bullet &) runs, since *this is a Bullet& 
                     //   - known at compile time 
                     // - methods resolves to 
                     //   - Rock::strike(Bullet &)
}

```

Another Example for Visitor Pattern: 
- add functionality to a class hierarchy without adding new virtual method 
- we add a visitor to the `Book` hierarchy

``` C++
class Book {

// ...

public: 
  // ...

  virtual void accept(BookVisitor &v) {
    v.visit(*this);
  }

};

class Comic : public Book {

// ...

public:

  void accept(BookVisitor &v) override {
    v.visit(*this);
  }

};


class Text : public Book {

// ...

public:

  void accept(BookVisitor &v) override {
    v.visit(*this);
  }

};

class BookVisitor {

public: 
  
  virtual void visit(Book &) = 0;
  virtual void visit(Text &) = 0;
  virtual void visit(Comic &) = 0;

}

```

A more concrete example on Book
- we categorize and count 
  - Books <- by author 
  - Texts <- by topic
  - Comics <- by hero 
- we could implement this with virtual method 
- Or with a visitor 

``` C++
class Catalogue :  public BookVisitor {

public: 
// for simplicity, we are going to have a public field here 
  
  map<string, int> theCat; 

  void visit(Book &b) override { ++theCat[b.getAuthor()]; }

  void visit(Text &t) override { ++theCat[t.getTopic()]; }

  void visit(Comic &c) override { ++theCat[c.getHero()]; }

  
};
```


## Detour: Circular include dependency (from visitor example from the course)
- `book.h`, `BookVisitor.h` includes each other 
  - include guard prevents multiple inclusion 
  - whichever ends up occurring first will refer to things that are not yet defined 
- it is important for C++ prgrammers to know when an `#include` is actually needed 
- needless `#include`s creates artificial compilation dependency and slow down compilation 
  - or even prevent compilation altogether 
- Sometimes a forward class declaration is good enough 

Consier: 
- `D` needs to `#include` for compiler to know how large `D` is 
- `B` needs `#include` with similar reason 
- `F` needs `#include`
- `C` forward declaration is ok since all pointers have same size 
- `E` forward declaration is ok since it only contains a function declaration 
- `G` depends 
  - depending on how the template `t` uses `A` 
  - once we know how `t` uses `A` it then just becomes one of the two other cases 

Note: class `F` only needs `#include` because method implementation is present and uses a method of `A` 
- it is one of the good reason to keep the implementation in the `.cc` file 


``` C++
class A {/*...*/}; // A.h
class B {
  A a;
};

class C {
  A *a;
}

class D : public A {

};

class E {
  A f(A);
};

class F {
  A f(A a) {a.someMethod(); }
}

class G {
  t<A> x;
};


```


Also Note:
- `B` needs an include but `C` does not
  - if we want to break the compilation dependency of `B` on `A` 
    - we can make `B` more like `C` 
    - more generally: *pImpl idiom*
      - it breaks compilation dependency

``` C++
class A1{}; class A2{}; class A3{}; 

// in b.h
class B {
  A1 a1; // <- now we have all these dependency and we are forced to include all the A's headers 
  A2 a2; 
  A3 a3; 
};

// solution: b.h

class BImpl; // forward declare only 
class B {

  unique_ptr<BImpl> pImpl;
};

// bimpl.h
#include "a1.h"
#include "a2.h"
#include "a3.h"

struct BImpl {
  A1 a1; 
  A2 a2; 
  A3 a3;
};

// b.cc 
#include "b.h"
#include "bimpl.h"

// methods reference pImpl->a1 / pimpl->a2 / pimpl->a3 
// now b.h is no longer compilation dependent on a1.h a2.h a3.h   etc. 

```    



### Another advantage of `pImpl idiom`
- since now `pImpl` is a pointer 
  - pointer have a non-thorwing swap 
  - can provides the strong guarentee on a `B` method by 
    - copying the `Impl` into a new `BImpl` structure (heap-allocate)
    - method modifies the copy, 
    - if anything throws, discard the new structure 
      - (easy and automatic with `unique_ptr`)
    - if all succeeds 
      - swap `Impl` structure (pointer swap - nothrow)
      - previous `Impl` automatically destroyed by the smart pointer 

``` C++
class B {
  unique_ptr<BImpl> pImpl;
// ... 

public: 

  void f() { 
    auto temp = make_unique<BImpl>(*pImpl);
    temp->doSomething();
    temp->doSomethingElse(); 
    std::swap(pImpl, temp); // no throw 
  } // string guarentee
};
```






