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




