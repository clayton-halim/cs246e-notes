# CS 246e
**2017-09-07**

_Brad Lushman_  
_DC 3110_  
_bmlushma@uwaterloo.ca_  
_[https://www.student.cs.uwaterloo.ca/~cs246e](https://www.student.cs.uwaterloo.ca/~cs246e)_  

Must use Linux:

**Windows:** 

putty.exe 

- connect to linux.student.cs.uwaterloo.ca
- enable X11 forwarding
- win scp

**Mac/Linux:** 

- terminal, ssh userid@linux.student.cs.uwaterloo.ca

Also Install xwindows server, eg. Xming, XQuartz

**TUTORIAL ATTENDANCE IS MANDATORY!**

## Goals:

- Meet the CS 246 objectives, more breadth, more depth
- A course on abstraction
- Demand-driven, problem-oriented presentation, introduce C++ concepts as needed
- Linux tools on the side/tutorials

## Problem 1: Program Input/Output

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

