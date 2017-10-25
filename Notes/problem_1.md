[**Home**](../README.md) | [>> Linear Collections and Modularity](./problem_2.md) 

# Problem 1: Program Input/Output
**2017-09-07**

- Read 2.2, 4.3 
- running a program from the command line

`./program-name` or `path/to/programe-name`

**Note:** `.` means current directory

Providing input: 2 ways

1. `./program-name arg1 arg2 ... argn`
    - args are written into the program's memory
    - [code][n + 1 (argc)][arg1][arg2][...][argn] (argv) 

2. `./program-name`
    - (Then type something)
    - input comes through standard input stream (stdin)
    - keyboard -> program -> stderr (never buffered) OR -> stdout (maybe buffered) -> screen

**Redirection**
`./my-program < infile > outfile 2>errfile`

**note:** 0 and 1 exists respectively, but only 2 is needed since it shares the same operator

Consider (`C`):

```C
#include <stdio.h>

void echo(FILE *f) {
    for (int c = fgetc(f); c != EOF; c = fgetc(f))
        putchar(c);
    }
}
```

**c is an `int` and not `char` because other information cannot be expressed as a `char`, for example `EOF` (end of file)**

```C
int main(int argc, char *argv[]) {
    if (argc == 1) {
        echo(stdin);
    } else {
        for (int i = 1; i < argc; ++i) {
            FILE *f = fopen(argv[i], "r");
            echo(f);
            fclose(f);   
        }     
    }

    return 0;  // Status code given to shell (echo $?)
}
```

**note:**
```C
argv[0] = program-name
...
argv[argc] = NULL
```

Observe: cmd line args / input from stdin - 2 _different_ programming techniques

To compile: `gcc -std=c99 -Wall my program.c -o myprogram`
`gcc` translates C source to executable binary
`-Wall` means warn all, gives warning that only appear if asked for
`-o` means what to rename program

This program is a simplification of the linux `cat` command
`cat file1 file2 ... filen` opens them and prints them one after another

`cat` - echos stdin
`cat < file`

- File used as a source for stdin
- The shell (not cat) opens the file
- Displays

_Can we write the `cat` program in C++?_

- Already valid C++
- The "C++" way
    - Command-line args are the same as in C
    - stdin/stdout: #include <iostream>

```C++
#include <iostream>

int main() {
    int x, y;
    std::cin >> x >> y;
    std::cout << x + y << std::endl;
}
```

`std::cin`, `std::cout`, `std::cerr`: streams, types
`std::istream`: (cin)
`std::ostream`: (cout, cerr)
`>>` input operator: (`cin >> x`), populates `x` as a side-effect, returns cin
`<<` output operator: (`cout << x + y`), prints `x + y` as a side-effect, returns cout

These operators return cin/cout so they can be chained:
From above: `std::cin >> x >> y;`

## File Access
```C++
std::ifstream f{"name-of-file"};  // ofstream for output
char c;

while (f >> c) {
    std::cout << c;
}
```

\*`f >> c`

- implicity converts to a bool
- true = read succeded
- false = read failed

**Input:** the quick brown fox
**Output:** thequickbrownfox

> Stream input slips whitespace (just like scanf)

To include whitespace:

```C++
std::ifstream f {"name-of-file"};
f >> std::noskipws;
char c;
...
```

**Note:** 

- No explicit calls to fopen/fclose
- Initializing `f {"name-of-file"}` opens the file
- When `f`'s scope ends, the file is closed

Try `cat` in C++

```C++
#include <iostream>

using namespace std;  // Avoids having to say std::

void echo (istream f) {
    char c;
    f >> noskipws;;
    while (f >> c) {
        cout << c;
    }
}

int main(int argc, char *argv[]) {
    if (argc == 1) {
        echo(cin);
    } else {
        for (int i = 1; i < argc; ++i) {
            ifstream f{argv[i]};
            echo(f);
        }
    }
}
```

Doesn't work, won't even compile!

`cin` has type `istream` - echo takes an istream
`f` has type ifstream - is `echo(f)` as type mismatch?

_No!_ - this is actually fine, ifstream is a **subtype** of istream

Any ifstream can be treated as an istream
- Foundational concept in OOP
- details later

The error is: you can't write a function that takes an istream the way that echo does

Why not?

Compare:

```C
int x;
scanf("%d", &x);
```
vs
```C++
int x;
cin >> x;
```

C and C++ are pass-by-value languages, so scanf needs the address of `x` in order to change `x` via pointers

So why is it not `cin >> &x`?
C++ has another small pointer-like type

## References

```C++
int y = 10
int &z = y; 
```

- `z` is an **lvalue reference** to `y`
- Similar to `int *const z = y;` but with **auto-dereferencing**

`z = 12;`

NOT `*z = 12`;
`y` is now 12

`int *p = &z;  // Gives the address of y`

In all cases, `z` acts as if it were `y`
`z` is an **alias** ("another name for `y`")

lvalue references must be initialized to something that has an address

`int &z = 4;` _**WRONG**_
`int &z = a + b;` _**WRONG**_

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
mycat: main.o echo.o
    g++ main.o echo.o -o mycat

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

ex. echo.cc newer than echo.o -rebuild echo.o
    echo.o newer than mycat - rebuild mycat

Shortcuts - use variables:

```C++
CXX = g++ (name of compiler)
CXXFLAGS = -std=c++14 -Wall
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
CXXFLAGS = -std=c++14 -Wall -MMD
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
---
[**Home**](../README.md) | [>> Linear Collections and Modularity](./problem_2.md) 
