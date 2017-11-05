# Measure of Design Quality

## **Coupling** and **Cohesion** 
- Coupling
    - how strongly different module depends on each other 
    - Coupling Level
        - Lowest: function calls with parameters / result of basic type 
        - Mid   : function calls with array / struct parameters
        - High  : modules affect each other's control flow 
        - VHigh : modules share global data 
        - EHigh : modules access each other implementation (friends)

- Cohesion 
    - how closely are elements of a module to each other 
    - Cohesion level 
        - Low   : arbitrary grouping (e.g. `<utility>`, `std::move`, `std::swap`)
        - Mid   : common theme, otherwise unrelated, maybe some common base code (e.g. `<algorithm>`, `std::transform`)
        - High  : elements that manipulate state over the lifetime of an object (e.g. `open`/`read`/`close` file)
        - VHigh : elements pass data to each other 
        - EHigh : elements cooporate to perform exactly one task (do one thing and do it well)



- high coupling
    - changes to a module affect other modules 
    - harder to reuse individual modules 
    - E.g. 
        - `whatIsIs()` from problem 23, that checks the type of `Book` 
            - tightly coupled to the Book hierachy 
            - every new book type requires as to change this function


- Low cohesion 
    - poorly organized code 
    - hard to understand, maintain and reuse 


## We are aiming for __High cohesion, Low coupling__ 







