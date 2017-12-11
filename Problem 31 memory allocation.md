# Problem 30 - Total Control 

So far, we can control 
- param passing 
- initialization
- method call resolution 
- etc. 

Now let's control the memory allocation

## Memory Allocation 
- memory allocation are tricky to write 
- 2 questions:
    - Why write an allocator ? 
        - built-in one is too slow 
            - general-purpose, not optimized for specific use 
            - e.g. if we know we will always allocate objects with the same size 
            - a custom allocator may perform better 
        - we might want to optimize locality 
            - maybe we want to separate heap just for objects of some class `C`, keeps the object close to each other 
            - may improve performance 
        - we might want to use "special memory"
            - put object in shared memory 
        - we might want to profile our program 
            - collecting stats 
    - How do we customize allocation ? 
        - by overload `operator new` 
        - If we define a global non-member `operator new`, all allocation in our program will use our allocator, 
        - also:
            - if we write a `operator new` we need to also write `operator delete` 
            - else: *UNDEFINED BEHAVIOUR* 


``` C++
void *operator new(size_t size) {
    cout << "Request for "" << size << "bytes\n";
    return malloc(size);
}

void operator delete(void *p) {
    cout << "Freeing " << p << endl;
    free(p);
}

int main() {
    int *x = new int;
    delete x;
}
```


- However, above works but it's not actually correct.
    - Doesn't address to the convention
    - if `operator new` fails, it is supposed to throw `bad_alloc` 
    - well, more accurately
        - if `operator new` fails, it is supposed to call the `new_handler` file 
        - the `new_handler` can 
            - free up space (somehow)
            - install a different `new_handler` / uninstall the current 
            - throw `bad_alloc`
            - abort / exit 
    - `operator new` should be calling `new_handler` on an infinite loop 
    - also: `operator new` must return a valid pointer if `size` == 0, and `operator delete` of a `nullptr` must be safe


``` C++
#include <new>

void *operator new(size_t size) {
    cout << "Request for "" << size << "bytes\n";
    if (size == 0) size = 1;
    while (true) {
        void *p = malloc(size);
        if (p) return p;

        // else 
        std::new_handler h = std::set_new_handler(0); // fetch a new handler 
        std::set_new_handler(h);                      // set new handler 
        if (h) h();                                   // call new handler if it is valid 
        else throw std::bad_alloc();                  // else throw 
    }

}

void operator delete(void *p) {
    if (p == nullptr) return;
    cout << "Deleting " << p << endl;
    free(p);
}

```

- Replacing global operator `new` / `delete` affects our entire program 
- more likely 
    - replace on a class-by-class basis 
    - especially if you are writing allocators specifically based to the size of our own objects 
- To do this: 
    - make operator `new` / `delete` *STATIC* members 

``` C++
class C {
public: /*...*/
    
    static void *operator new(size_t size) {
        cout << "Running C's allocator" << endl;
        return ::operator new(size);
    }

    static void operator delete(void *p) noexcept { 
        cout << "Freeing " << p << endl;
        ::operator delete(p);
    }
};
```

- to Generalize - we can log to arbitrary stream 

``` C++
class C {

public:
    static void *operator new(size_t size, std::ostream& out) {
        out << "/*...*/" << endl;
        return ::operator new(size);
    }

    static void *operator delete(void *p, std::ostream& out) noexcept {
        out << "Placement delete: " << p << endl;
        ::operator delete(p);
    }   

};

void () {
    C * x = new(cout) C; // log to cout 
    ofstream f{/*...*/};
    C* y = new(f) c; 
}
```

- However, above implementation will not compile 
    - we also need to write non-placement ordinary delete 

``` C++
class C {

public:

    /* ... * /

    static void operator delete(void *p) noexcept {
        cout << "Ordinary delete" << endl;
        ::operator delete(p);
    }

};
```


- for above implementation, if the client calls `delete p`, there need to be a non-specialized operator `delete`, else compile error 
- How can you call specialized `delete` - You can't 
- Then why do we need it ?  
    - if the `ctor` that called specialized `opeartor new` throws 
    - it will call the specialized `operator delete` that matches `operator new` 
    - if there isn't one, no `delete` gets called => leak 


E.g.

``` C++
class C {

public:
    C(bool b) {
        if (b) throw 0;
    }


};

void f() {
    try {
        C *p = new (cout) C{true}; // throws 
        delete p; // not reached 
    } catch (...) {
        C *q = new(cout) C{false};
        delete p;
    }
}
```


- Customizing array allocation 
    - overload `operator new[]` and `operator delete[]` 









