# SOLID Principle of OO Design 

## __S__ ingle Responsibility Principle  (SRP)
- a class should have only one reason to change 
    - a class should only do one thing, not severa
    - e.g. `vector` 
- any change to the problem spec requires change to the program, 
- if changes >= 2 different parts of the spec causes changes to the same class, SRP is violated 
- E.g.
    - don't let your (main) classes print things 
    - Consider:
``` C++
class ChessBoard {
// ...
    void some_function() {
        // ...

        cout << "Your move" << endl; // 

        // ...
    }
// ... 
}
```
        
        - bad design
            - inhibits code reuse 
            - what if you want a version of your program that:
                - communicates over different streams (file or network)
                - works in another human language (French ?)
                - being graphically displayed rather than just being printed 
            - all of the above requires major changes to the chess board, instead of reuse 
            - violates SRP - must change the class if there is any change to the specification for 
                - game rules
                - strategy 
                - interface 
                - etc. 
            - low cohesion situation 
            - split up different components 
                - one module (not main, very hard to reuse main) responsible for communication 
                    - if a class wants to say something, do it via parameters / result / exceptions
                    - pass the info to the communication object and let it do the talking 
- One the other hand - specifications that are unlikely to change may not need their own class 
    - to avoid needless complexity 
    - there is always room for judgement 


## __O__ pen/closed Prinple 
- classes / modules / functions etc. should be _open for extension_ and _closed for modification_
- changes in a program's behaviour, should happen by writing new code instead of changing old code 
- E.g.
```
    |carpenter| ♦------> |handsaw|

    What if the carpenter buys a table saw ? 
```
    - this design is not open for extension
        - we have to change the carpenter code 
- Better example - abstraction 
```
    |carpenter| ♦-------> |saw|
                            ^
                            |
                        ------ - - -
                   |handsaw|    |tablesaw|
```


// TODO: FIX THIS 

## __L__ SP Liskov Substitution Principle

### Liskov Substitution 

<tag> to be filled later </tag>

* violates Liskov Substitution 
- A circle __is__ a shape 
- A shape can be compared with __any__ other shape 
- Therefore, a circle can be compared with __any__ other shape 
- (we saw this with virtual `operator=`)
- C++ will flag this problem with a compiler error


Fix: 
``` C++
#include <typeinfo>
bool Circle::operator==(const Shape &other) const {
    if(typeid(other) != typeid(circle)) return false; 
    Circle &other = static_cast<Circle &>(other);
    // compare fields of other with fields of (*this)
}
```

### Choice of using `typeid` vs `dynamic_cast` 
- `dynamic_cast<Circle&>(other)` 
    - asking is `other` a `Circle` or a subclass of `Circle`? 
- `typeid(other) == typeid(Circle)`
    - is `other` PRECISELY a `Circle` ? 
    - it returns an object of `typeinfo` 


2. Is a `Sqaure` a `Rectangle` ? 
- a `Sqaure` has all the properties of a `Rectangle` 
``` C++
class Rectangle {
    int length, width; 

public: 
    Rectangle(/*...*/);

    int getLength() const;

    virtual void setLength(int); 

    int getWidth() const; 

    virtual void setWidth(int);

    int area() const {
        return length * width;
    }
};


class Square: public Rectangle { 

public:
    Square(int side): Rectangle(side, side) {} 

    void setLength(int l) override { 
        Rectangle::setLength(l);
        Rectangle::setWidth(l);
    }

    void setWidth(int l) override {
        Rectangle::setLength(l);
        Rectangle::setWidth(l);   
    }

};


int f(Rectangle &r) {
    r.setLength(10);
    r.setWidth(20);
    return r.area(); // 200
}

int main() {
    Square s{1};
    f(s); // 400 ??? WTF 
}
```


### In this case, `Square` != `Rectangle` 
- `Rectangle`s have the property that their length and width can vary independently, square do not 
- hence this violates `LSP` 
- On the other hand, and immutable `Square` could substitute for an immutable `Rectangle` 


```
                       |shape|
                           ▵
                           |
                  ---------------------
                 |
      |RightAngleQuaduolateral|
                 ▵
                 |
        ------------------------
       |                        |
  |Rectangle|               |Square| 
    ^                           ^
can vary length / width      lenght == width 

```

### Question: Can we constrain what subclasses can do 
- Consider 
``` C++
class Turtle {

public: 
    virtual void draw() = 0; 

};


class RedTurtle : public Turtle {

public:

    void draw() override {
        drawHead();             // same as GreenTurtle
        drawRedShell();
        drawFeet();             // same as GreenTurtle
    }

};

class GreenTurtle : public Turtle {

public:

    void draw() override {
        drawHead();              // same as RedTurtle
        drawGreenShell();
        drawFeet();              // same as RedTurtle
    }
};
```


- this is code duplication 
- how can we ensure that `drawHead()` and `drawFeet()` are always being called 

``` C++
class Turtle {

public:
    
    void draw() {
        drawHead();
        drawSell();
        drawFeet();
    }

private:
    
    void drawHead() {}

    virtual void drawShell() = 0;

    void drawFeet() {}

};


class RedTurtle : public Turtle {

    void drawShell() override { }

};

class GreenTurtle : public Turtle {

    void drawSell() override { }
};
```

- note here even though `drawShell()` are not visible to subclasses, subclasses can still override it 
- now all subclasses cannot control the steps of drawing a turtle, nor the drawing of head and feet 
- they can only control the drawing of the shell 
- this is called the __Template Method Pattern__ (PS: more precisely, boiler-plate method pattern)



## Extension - NVI (Non-Virtual Interface) Idiom 
- public virtual method are simultaneously 
    - part of a class' interface  (making a promise)
        - pre/post conditions 
        - respect invariants 
    - "hooks" for customization by subclasses (offer ability to change the promise) 
        - overriding code could be anything 
- NVI:
    - all virtual methods should be private 
        - or, all public methods should be non-virtual 
    - essentially, NVI is a generalization of Template Method Pattern 
        - put __every__ virtual function inside a template method 

``` C++
// Non NVI class 
class DigitalMedia {

public:
    
    virtual void play() = 0;
    
};

// NVI version 
class DigitalMedia {

public:

    void play() {
        // now we can add before / after code that is non optional 
        // e.g. checkCopyRight(), updatePlayCount() 
        doPlay();
    }

private:
    
    virtual void doPlay() = 0;

};
```
    



## __I__ terface Segregation Principle
- many small interfaces is better than one larger interface 
- if a class has many functionalities, each client of the class should see only the functionality it needs 
- E.g.
    - Video Game 

``` C++
class Enemy {

public:
    
    virtual void strike();  // needed by game logic 
    virtual void draw();    // needed by GUI logic 

};


class UI {

    vector<Enemy *> v;

};

class Battlefield {

    vector<Enemy *> v;

};

```

- Note:
    - if we need to change the drawing interface, `Battlefield` needs to recompile for no reason 
    - similarily, `UI` need to recompile for no reason if we need to change combat interface 
    - this creates a coupling that we don't really need 

- Solution (One of): 
    - __Multiple Inheritance__
        - Example of __Adapter Pattern__ 
        - side note:
            - we don't use `unique_ptr` here since `Battlefield` and `UI` do not have ownership over `Enemy`

``` C++
class Drawable {

public:
    
    virtual void draw() = 0;

};


class Combatable {

public:
        
    virtual void strike() = 0;

};

class Enemy : public Draw, public Combat { };

class UI {
    vector<Drawable *> v;
};

class Battlefield {
    vector<Combatable *> v;
};
```



- General use of __Adapter__
    - when a class provides an interface differs from the one you need 

E.g.
```
       |NeededInterface|             |Provided Class|
       |+g()           |             |+f()          |
              ▵                              ▵
              |                              |
              --------------------------------   <-- this inheritance could be `private` 
                             |
                         |Adapter|
                         |+g() --|--------|{f();}|


```


- `private` inheritance
    - the `is-a` relationship between subclass and super class is private 
    - all public interface inherited from super class becomes private 
    - this depends on whether or not if we want the adapter to support the old interface 



## Side Note: Issue with multiple inheritance
```
    |A1  |       |A2  | 
    |+a()|       |+a()|
      ▵            ▵
      |            |
      --------------
            |
           |B|  - has 2 a() methods 

Or the famous Deadly Diamond Inheritance

          |A   |
          |+a()|
            ⧊
            |
      --------------
      |            |
     |A1|        |A2| 
      ▵            ▵
      |            |
      --------------
            |
           |D|  - has 2 a() methods 
                - and they are different 

```

### Question: How do we deal with this situation:
``` C++

class D: public B, public C {

public: 
    
    void f() {
        // ... 
        a();
//      ^~
//      this is ambigious 
//      to fix this, we can do:
        B::a();
        C::a();
        // ...
    }

};


// or 

void k() {
    D d;
    d.a(); 
//  ^~~~
// this is also ambigious
// ugly patch:
    d.B::a(); 
    d.C::a();
}

```


- The patch we used was very ugly, 
- or maybe there should be only one `A` base, and therefore only one `a()`
    - __Virtual Inheritance__
        - means the subclass is willing to share their super class methods and properties 

``` C++
class B: virtual public { /*...*/ };
class C: virtual public { /*...*/ };

void f() {
    d.a();   
//  ^~~~
//  this is no longer ambigious 
}
```

- in fact, we have been using virtual inheritance all along 
- E.g. `iostream` hierachy 
``` 
                        ios_base 
                            |
                           ios 
                          /   \ 
                    istream   ostream 
                   /   |   \ /       | \ 
            ifstream   |    iostream |  ofstream
             istringstream     |    ostringstream
                               |
                              / \
                       fstream   stringstream

```



### Problem: 
- How will a class like `D` be laid out in memory 
    - (implementation-specific)
    - Consider: 
```

D:   |vptr    | <- recall`is-a` relationship, it should look like an A*, B*, C*, D*
     |A fields|    but here this pointer does not look like a C*
     |B fields|    since `C` does not even know `B` exists 
     |C fields|
     |D fields|

```
    - what does g++ do? 

```

    |vptr    |
    |B fields|
    |vptr    |
    |C fields|
    |D fields|
    |vptr    |
    |A fields|

```









