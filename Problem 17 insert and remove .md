# Problem 17 - Insert/remove in the middle 

A method like `vector<T>::insert(size_t i, const T &x)` is easy to write 

But the same for `list<T>` requires an up-front traversal

Using `iterator`s can give both `list` and `vector` some advantage 

``` C++
template <typename T>
class vector {
// ...
public:
    iterator insert(iterator posn, const T &x) {
        ptrdiff_t offset = posn - begin();
        increaseCap(); 
        // since increaseCap might potentially change the pointers 
        // we can't direclty work with the original posn 
        // therefore we compute the offset 

        iterator newPosn = begin() + offset;

        new (static_cast<void*>(end())) T(std::move(*(end() - 1)));
        ++vb.n;
        for(iterator it = end() - 1; it != posn; --it) {
            *it = std::move(*(it - 1));
        }
        *newPosn = x;
        return newPosn;
    }
};
```


## Question: is `insert` exception safe? 
- Yes, assuming `T`'s copy/move operations are exception safe (at least basic guarantee)
- `insert` offers the basic guarantee 
- we may get a partially shuffled `vector`, but it will be a valid `vector` in the end 

## Note:
- if you have other iterators pointing at the vector:
```
       it2
       v
 _ _ _ _ _ _
|1|2|3|4|...
^    ^
it1  here

and if we insert at "here"
it2 will now point to a different itm 

       it2
       v
 _ _ _ _ _ _ _
|1|2|5|3|4|...
^    ^
it1  here
```

hence we have a convention:

## Convention: after a call to `insert` or `erase`, all iterators pointing after the point of insertion / erasure are considred invalid and should not be used 

## Also: if a reallocation happens, _ALL_ iterators pointing into the vector became invalid
- to check if reallocation happens, we can check the value of `cap` in a vector 

Exercise: 
- `erase`: remove the item pointed to by an iterator, return an iterator to the point of erasure 
- `emplace`: like `insert` but takes `ctor` args 

## But
- that means there is a problem with `push_back` 
    - if `increaseCap` successfully reallocates and placement new (ctor) throws, the `vector` is the same, but all `iterator`s are invalid, 
- exercise: fix it 
    - either fall back to say `push_back` only give basic guarantee
    - or we need to keep the old array until we know that the new array is valid 