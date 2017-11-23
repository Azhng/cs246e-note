# Problem 28: Resolving method overrides at compile-time 

Recall: Template Method Pattern 

``` C++
class Turtle {

public:
    
    void draw() {
        drawHead();
        drawShell();
        drawFeet();
    }

private:
    
    void drawHead();
    virtual void drawShell() = 0;
    void drawFeet(); 

};

class RedTurtle : public Turtle {

    void drawShell() override; 

};

```

- ability to call virtual function is implemented using Vtable, 
- how can we save that cost ? 
    - using template 

``` C++
template <typename T>
class Turtle {

public:
    
    void draw() {
        drawHead();
        static_cast<T*>(this)->drawShell(); // we just assume our subclass will define drawShell 
                                            // so when we cast itself down to its subclass `drawShell` will be available 
        drawFeet();
    }

private:
    
    void drawHead(); 
    void drawFeet();

};

class RedTurtle : public Turtle<RedTurtle> {

    friend class Turtle;
    void drawShell(); 
};

class GreenTurtle: public Turtle<GreenTurtle> {
    friend class Turtle;
    void drawShell(); 
};

```

- no virtual method 
- no Vtable lookup 

Drawback:
- no relationship between `RedTurtle` and `GreenTurtle` since now they are inheriting from different base class 
- we cannot stored mixture of them in a container 
    - in this case we just give the `Turtle<T>` class a non-template parent 

``` C++
template <typename T> 
class Turtle : public Enemy {
    // ... 
}
```

- then we can store `RedTurtle`s and `GreenTurtle`s in single contianer 
- but we no longer have access to `draw` method 
    - well, we could give `Enemy` a virtual `draw()` method 

``` C++
class Enemy {

public:
    
    virtual void draw() = 0; 

};
```

- But now there *WILL* be a Vtable lookup 
- on the other hand, if `Turtle::draw` calls several would-be virtual helpers, could trade away several Vtable lookups for one 
