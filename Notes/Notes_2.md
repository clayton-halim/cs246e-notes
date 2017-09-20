# References (cont.)
**2017-09-14**

- Cannot create a pointer to a reference: `int &*x;`
- Read right to left:
    - > x is a pointer to a reference of an int
- However you can create a reference to a pointer: `int *&x = __;`
- Create a reference to a reference: `int &&r = z;` (compiles but means something else) 
- Create an array of references: `int &r[3] = {...};`
- You can use as function parameters:

```C++
void inc(int &n) {
    ++n;
}

int x = 5;
inc(x);

cout << x; // 6
```

**Note:** cannot use `inc(5)` because references require objects with addresses (ie. literals do not)

`cin >> x` works because `x` is passed by reference.

`istream& operator >> (istream &in, int &n);`

Now consider `struct ReallyBig {...};`

```C++
int f(ReallyBig rb) {
    ...
}
```

**Note:** `struct ReallyBig` is not necessary in C++ unlike in C

The case above uses passes by value which copies the huge struct -> this is really slow!

Instead let's pass by reference which doesn't copy it -> this is fast!
But this can be dangerous as the function may propogate changes to the caller.

```C++
int g (ReallyBig &rb) {
    ...
}
```

So finally we can use constant references, this is fast and safe!

```C++
int h(const ReallyBig &rb) {
    ...
}
```

Prefer pass-by-const-ref over pass-by-value for anything larger than a pointer, unless the function needs to make a copy anyways.

Also:

```C++
int f(int &n) {
    ...
}

int g(const int &n) {
    ...
}
```

`f(5)` - (BAD) can't initialize an lvalue reference `n` to a literal value `5`.
`g(5)` - (GOOD) since n can never be changed, the compiler allows this (stored in some temp location so n has something to point to)

Back to `cat`:

```C++
void echo(istream f) {
    ...
}
```

- `f` is passed by value, `istream` is copied
- Copying streams is _not allowed_ (C++ has mechanisms to prevent copying)

Works if you pass the stream by reference:

```C++
void echo (istream &f) {
    f >> noskipws;
    char c;

    while (f >> c) {
        cout << c;
    }
}
```

To compile:
- `g++ -std=c++14 -Wall mycat.cc -o mycat`
- OR `g++14 mycat.cc -o mycat`
- `./mycat`

## Separate compilation
Put echo in its own module

### echo.h
```
void echo (istream &f);
```

### echo.cc
```C++
#include "echo.h"

void echo(istream &f) {
    ...
}
```

### main.cc
```C++
#include <iostream>
#include <fstream>
#include "echo.h"

int main(...) {
    echo(cin);
    ...
    echo(f);
}

```

**Compiling separately:** 
- `g++14 echo.cc` (fails)
    - linking error: no main
- `g++14 main.cc` (fails)
    - linking error: no echo

Correct:

`g++14 -c echo.cc` -> creates `echo.o`
`g++14 -c main.cc` -> creates `main.o` (these are object files)

`-c` indicates **only compile, don't link**

`g++14 echo.o main.o -o mycat` (linker)

Advantage:

- Only have to recompile the parts you change, then relink (no so expensive)
    - Ex. Change echo.cc -> recomplie echo.cc -> relink
- However if you change echo.h, you must recompile `echo.cc` and `main.cc` and relink
    - This is because both files include `echo.h`
- What if we don't remember what we changed or what depends on what?
    - Linux tool: `make`
    - Create a **Makefile**

```C++
my cat: main.o echo.o
    g++ main.o echo.o mycat

main.o: main.cc echo.h
    g++ -std=c++14 -Wall -c main.cc

echo.o: echo.cc echo.h
    g++ -std=c++14 -Wall -c echo.cc

.PHONY: clean

clean:
    rm mycat main.o echo.o
```

**Note:** cannot use aliases, requires _TABS_ not _SPACES_

- **targets:** mycat, main.o
- **dependencies:** everything right of colon
- **recipes:** tabbed information

How make works: list a dir in long form `ls -l`

-rw-r----- 1 j2smith j2smith 25 Sep 9 15:27 echo.cc

- Based on last modified time
- Starting at the leaves of the dependency graph
    - if the dependency is newer than the target, rebuild the target ... and so on, up to the root target

ex. echo.cc nwewer than echo.o -rebuild echo.o
    echo.o newer than mycat - rebuild mycat

Shortcuts - use variables:

```C++
CXX = g++ (name of compiler)
CXXFLALS = -std=c++13 -Wall
EXEC = myprogram
OBJECTS = main.o echo.o

${EXEC}: ${OBJECTS}
    ${CXX} ${OBJECTS} -o ${EXEC}

main.o: main.cc echo.h
echo.o: echo.cc echo.h

.PHONY: clean

clean: 
    rm ${OBJECTS} ${EXEC}
```

\*Omit recipes - make "guesses" the right one

Writing the dependencies still hard (but compiler can help).

- `g++14 -c -MMD echo.cc`: generates `echo.o, echo.d`
- `cat echo.d` -> `echo.o echo.cc echo.h` 

```C
CXX = g++
CXX FLAGS = -std=c++14 -Wall -MMD
EXEC = mycat
OBJECTS = main.o echo.o

DEPENDS = ${OBJECTS:.o=.d}

${EXEC}: ${OBJECTS}
    ${CXX} ${OBJECTS} -o ${EXEC}

-include ${DEPENDS}

.PHONY: clean

clean:
    rm ${EXEC} ${OBJECTS} ${DEPENDS}

```



