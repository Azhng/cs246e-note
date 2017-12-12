# Problem 32 - A fixed-size object allocator 

A custom allocator can be fast 

Fixed-size allocator:
- all allocated "chunks" are the same size (i.e. customized for one class)
- no need to keep track of size 
- (aside: many traditional allocators store the size of the block before pointer, so that the allocator knows how much space is allocated to that pointer)

Fixed-size:
- saves space (no hidden size field)
- saves time - no hunting for a block of the right size 


When the client has a slot : T object 

When we have it
- node in a linkedlist, each slot stores the index of the next slot in the list 

```

Allocation: from the front

        allocated 
        |
        v
|1|    |/|2|3|4|...|-1|
        0 1 2 3     n-1

|2|    |/|/|3|4|...|-1|
        0 1 2 3     n-1

Deallocation: 

free item 0: 

|0|    |2|/|3|4|...|-1|
        0 1 2 3     n-1

free item 1:

|1|    |2|0|3|4|...|-1|
        0 1 2 3     n-1

```


``` C++
template <typename T, int n> 
class fsAlloc {

    union slot {     // slot - large enough for a `T`, but still useable as an int 
        int next; 
        T data; 
        Slot() : next{0} {}
    };

    Slot theSlot[n];
    int first = 0;

public: 
    
    fsAlloc() {
        for (int i = 0; i < n - 1; ++i) {
            theSlot[i].next = i + 1;
        }
        theSlot[n-1].next = -1;
    }

    T *allocate() noexcept { 
        if (first == -1) return nullptr;
        T *result = &(theSlot[first].data);
        first = theSlot[first].next;
        return result;
    }

    void deallocate(void *item) noexcept {
        int index = (static_cast<char*>(item) - reinterpret_cast<char*>(theSlot)) / sizeof(slot);
        theSlot[index].next = first;
        first = index;
    }
};


// TO use it in class 

class Student final {  // subclass would likely to be larger, hence fixed-size allocator won't work 
    int assns, mt, final;
    static fsAlloc<Student, SIZE> pool;

public: 

    // ... 
    static void* operator new(size_t size) {
        if (size != sizeof(Student)) throw std::bad_alloc;
        while (true) {
            void *p = pool.allocate(); 
            // ... 
        }
    }

    static void operator delete(void *p) noexcept {
        if (p == nullptr) return;
        pool.deallocate(p);
    }

};



int main() {
    Student *s1 = new Student; // custom allocator 
    Student *s2 = new Student; // custom allocator 
    delete s1; // custom deallocator
    delete s2; // custom deallocator
}
```


Question: Where do `s1` and `s2` reside ? 
- static memory (not the heap) 
    - could arrange for stack / heap memory 


More Notes:
- we used a `union` to hold both `int` and `T`, it waste less space 
- we could have used a struct 
- disadvantage of fixed-size allocator 
    - if we access a dangling `T` ptr, we can corrput the linked list,


``` C++
Student *s = new Student;
delete s;
s->setAssns(5);
```


Lesson: following a dangling pointer can be VERY dangerous 

With a struct, `next` is before the `T` object, so we have to work hard to corrput it, E.g.:

``` C++
reinterpret_cast<int *>(s)[-1] = /* ... */; 
```


On the other hand, 
- with a `struct`, problem if `T` doesn't have a default ctor 


E.g.

``` C++
struct Slot {
    int next;
    T   data; 
}

void f() {
    Slot theSlots[n]; // if `T` has no default ctor 
                      // Can't do operator new / placement new 
}

union SlotChar {
    char dummy;
    Slot s;
    SlotChar() : dummy{0} {}
}
```


Also 
- Why store indices instead of pointers, 
    - smaller than pointers on this machine 
    - so waste no memory as long as `T` >= size of an int 
    - would waste if `T` is smaller than an int
    - could use a smaller index type, e.g. short / char 
        - (as long as we don't want more items than the type can hodl )
    - could amek the index type a param of the template 
    - a C 
- STUDENT final - fixed-size allocator, subclass might be larger 
    - option: 
        - check size, throw if it isn't the same size 




















