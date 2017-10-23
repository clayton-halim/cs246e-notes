[Abstraction over containers << ](./problem_17.md) | [**Home**](../README.md)

# Problem 18 - Heterogenous Data
**2017-10-19**

I want a mixture of types in my vector.

Can't do this with a template.

```C++
vector<template<typename T> T> v;
```

- Not allowed - templates are compile time entities, don't exist at runtime

Ex. Fields of a struct

```C++
class MediaPlayer {
    template<typename T> T nowPlaying;   // X - wrong
};
```

...