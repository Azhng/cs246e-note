# Problem 27: Collecting stats 

I want to know how many `Student`s I have create 

``` C++
// Student.h
class Student { 
    int assn, mt, final;
    static int count;  // static - associated with a class, not with object 

public:
    
    Student(/*...*/) {
        ++count;
    }

    // static method 
    // - have no `this` 
    // - in some sense, they are not really methods, they are more like scoped functions 
    static getCount() const {
        return count;
    }

};

// Student.cc
int Student::count = 0; // must define the variable 

// other.cc
Student s1{/*...*/}, s2{/*...*/}, s3{/*...*/};
cout << Student::getCount() << endl;
//      ^~~~~~~~~~~~~~~~~~~
//      we don't have receiver object 
```

Now I want to count objects in ohter classes, How do we abstract this solution into reusable code ? 

``` C++
template <typename T> 
struct Count {
    static int count;
    Count() { ++count; }
    Count(const Count&) { ++count; }
    Count(Count&&) { ++count; }
    ~Count() { --count; }
    static int getCount() { 
        return count;
    }
};


// in implementation 
tempalte <typename T> int Count<T>::count = 0;

// Student.h
class Student : Count<Student> { // private inheritance
                                 // - inherits Count's implementation without creating a `is_a` relatonship 
                                 // - member of Count becomes private 
    int assns, mt, final;

public:
    Student(/*...*/) {/*...*/}
    
    // accessors 

    using Count<Student>::getCount; // make Count::getCount visible 
};

// Book.h
class Book : Count<Book> {
// ...

public:

// ...    
    using Count<Book>::getCount;

};
```

Question: Why is `Count` a template ? 
- So that for each class `C`, there will be a separate, new and unique instantiation of `Count`
- this gives each `C` its own counter, instead of sharing all one single `Count` class 


This technique:
- inheriting from a template specialized by ourselves
- looks weird, but happens enough to have its own name
- *The Curiously Recurring Template Pattern (CRTP)*


