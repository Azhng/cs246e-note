# Problem 21 - I want a class with no objects 

``` C++
class Student {
public:
    virtual float fees() const;
};


class RegularStudent : public Student {
public:
    float fees() const override; // regular student fee
};

class CoopStudent : public Student {
public:
    float fees() const override; // coop student fee 
}
```

## What should `Student::fees` do ? 
- Don't know, every student should be regular or coop 
- that means `Student::fees` should explicitly have no implementation 

``` C++
class Student { // this is what we called abstract class, we don't have a keyword for it in C++ 
public:
    virtual float fees() const = 0; // pure virtual method 
}
```

- abstract classes cannot be instantiated 
    - `Student s; // now allowed`
    - `Student *s = new Studnet; // not allowed`
- can point to instances of concrete classes 
    - `Student *s = new RegularStudent;`
- subclasses of abstract classes are abstract, unless they implements all pure virtual methods
- abstract classes can be
    - used to organize concrete classes 
    - can contain common fields, methods, default implementation