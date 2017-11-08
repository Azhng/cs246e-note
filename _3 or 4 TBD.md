### Liskov Substition 

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

## Question: Can we constrain what subclasses can do 
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
    - part of a class' interface 
















