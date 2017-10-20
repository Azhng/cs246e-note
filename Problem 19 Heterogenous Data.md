# Problem 19: I want a mixture of types in my `vector`

This is a problem that we cannot solve using template 
we could try this, but it won't work 
``` C++
vector<template <typename T> T> v; 
```

This is not allowed - tempaltes are compile-time 

E.g. fields of a struct 
``` C++
class MediaPlayer {
    template <typename T> T nowPlaying; // not allowed either 
};
```