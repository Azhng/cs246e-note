# Object-Oriented Design 

## System Modelling - UML (Unified Modelling Language)
- make ideas easy to communicate 
    - aid design discussion 
```
                                         ___________________
                                        |Book               | <- class name           - italics = abstract 
                                        |___________________|
                                        |-title:  String    | <- fields (optional)
                                        |-author: String    |
                                        |#length: Integer   |
                                        |___________________|
                                        |+getTitle:  String | <- methods (optional)   - italics = virtual 
                                        |+getAuthor: String |
                                        |+getLength: String |
                                        |+_isHeavy_: Boolean|
                                        |___________________|
                                                 ▵ <- is-a relationship (inheritance)
                                                 |
                                                 | 
                      -----------------------------------------------------------------------
                      ▵                                                                     ▵
                      |                                                                     |
                      |                                                                     |
             ___________________                                                       ___________________
            |Text               |                                                     |Comic              | 
            |___________________|                                                     |___________________|
            |-topic:  String    |                                                     |-hero: String      | 
            |___________________|                                                     |___________________|
            |+getTopic:  String |                                                     |+getAuthor: String |
            |+isHeavy:   Boolean| <- not italic, means                                |+isHeavy:   Boolean|
            |___________________|                                                     |___________________|

```

`-` => private 
`+` => public 
`#` => protected 
_italics_ => virtual 


- `owns-a` relationship (composition)
    - means the motor is part of the car 
        - does not have an independent existence 
        - if I copy or destroy the car, then I am copying/destroying the motor (deep copy)
            - typical implementation: class composition, i.e. object field 
                - e.g. `class Car { Motor m; };`

```             
     _____________         m  <- the name is called `m` 
    |car          | ♦------> |motor|
    |_____________|
    |-VIN: Integer|

```


- `has-a` relationship (aggregation)
    - Duck has its own independent existence 
    - copy and destroy pond does not imply copy or destroy duck 
    - typical implementation, pointer field 
        - e.g. `class Pond { vectorDuck*> ducks; };`

```
                 a1⋆
    |pond| ♢------> |duck|

```


## Every resource (memory, file, window, etc.) should be **owned** by an object that will release it -- RAII
- a `unique_ptr` **owns** the memory it points to 
- a raw pointer means not owning the memory it points to 
    - this means calling `delete` is probably wrong 
- if we need to point at the same object with several pointers, it would be idea that one pointer owns it, the rest are raw pointers 
- moving a `unique_ptr` means transfering ownership
- if you need TRUE shared ownership - later 



