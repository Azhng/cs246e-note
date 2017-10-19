# Problem 10 - Staying in bounds 

``` C++
vector v; 
v.push_back(4);
v.push_back(5);
v[2]; // out of bounds!, undefined behaviour, may or may not crash 
```

Can we make this safer? 
- we can add bound check to `operator[]` 
    - needlessly expensive
- we could have a second safer fetch operator

``` C++ 
class vector {
    // ...
public: 
    int& at(size_t i) {
        if(i < n) return theVector[i];
        // if we return a non-int : it is not type-correct 
        // if we crash the program - but can we do better ?
        // e.g. don't crash and don't return 
        // solution:
        //     raise an exeption 
        else throw range_error();
        //         ^~~~~~~~~~~~~
        //         construct an object of type range_error and throw it 
    }
};

class range_error {};
```

Client's option: 
1) do nothing:
``` C++
vector v;
v.push_back(1);
v.at(1); // exception is raised and the exception will crash the program 
```
2) catch the exception:
``` C++
try {
    vector v;
    v.push_back(1);
    v.at(1); 
} catch(range_error &r) {
    //              ^~~
    //             r is the thrown object 
    //             we are catching it by reference - saves a copy 
    // do something that users deemed to be appropriate 
}
```
3) let the caller catch (throw it to the caller / a.k.a. ducking). 
   - the exception will __propagate__ back through the call chain until a handler is found 
   - called __`unwinding`__ the stack 
   - if no handler is found, program aborts 
        - `std::terminate` gets called 
   - control resumes after the catch block (problem code is not returned)
``` C++ 
void f() {
    vector v;
    v.push_back(1);
    v.at(1);
}

int g() {
    try{
        f();
    } catch(range_error &r) {
        // do something 
    }
}

```

## Question: What happens if a ctor throws an exception ? 
- object is considered partially constructed 
- dtor will not run on partially constructed object 

``` C++
class C {/*...*/};

class D {
    C a;
    C b;
    int *c;
public:
    D() {
        c = new int[100];
        if(/*...*/) {
            throw something(); // *
        }
    }

    ~D() {
        delete c;
    }
};

int main() {
    D d;
    // at *, D object is not fully constructed, 
    // so ~D() wil not run on d
    // but a and b are fully constructed ?
    // so their dtors will run 
    // so if a ctor wants to throw, it must clean up after itself 
    
}
```

So this is the proper way: 
``` C++

class C {/*...*/};

class D {
    C a;
    C b;
    int *c;
public:
    D() {
        c = new int[100];
        if(/*...*/) {
            delete[] c;
            throw something(); // *
        }
    }

    ~D() {
        delete c;
    }
};
```

## Question: What happens if a dtor throws ?
- __Trouble__
- `std::terminate` called by default, program immediately aborts 
- if you really want a "throwing dtor", tag it with noexcept(false)
    - but watch out: 

``` C++
class myexn {};

class C {
// ...
public:
    ~C() noexcept(false) {
        throw myexn();
    }
};

void h() {
    C c1; // dtor for c1 throws at the end of h 
}

void g() {
    C c2; // unwind through g
    h();  // dtor for c2 throws as we leave g
          // no handler yet, but there are 2 unhandled exceptions 
          // if you have more than 1 unhandled exception immediate termination 
          // std::terminate is called 
}

void f() {
    try {
        g();
    } catch(myexn &ex) {
        // ...
    }
}
```

### Take away: __NEVER__ let dtors throws, swallow the exception if necessary 
Also note: 
- in C++, exception doesn't necessary have to be objects 
    - it can be any value 


