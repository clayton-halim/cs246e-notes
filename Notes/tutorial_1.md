[**Home**](../README.md)

#Valgrind + GBD
**2017-09-27**

- Examples of Memory Errors (undefined behaviour)
    - Memeory leaks
    - Segmentation fault
    - Uninitialized variable access
    - Double free
    - Mixing new/delete 

**Valgrind** - tool to detect memory errors 
- Linux only
- Syntax: `valgrind <args to valgrind> <program> <args to program>`

### 4 types of memory leaks
1. **Definitely lost** 
    - Fix these (usually some memory not having anything pointing to it)
1. **Indirectly lost** 
    - Probably ignore these (usually some other thing pointing to it is lost)
    - Usually the source of the missing data is definitely lost (fixing that usually fixes this)
1. **Possibly lost**  
    - Treat the same definitely lost for our purposes (very complicated)
1. **Still reachable**
    - Still has something pointing to it, but not deallocate when program ends
    - For our purposes, always fix
    - In the real word: case-by-case, use your judgement

But valgrind doesn't give us line by line information, but luckily we can compile with the arg `-g` in `g++`

`-g` does two things:
1. Stores all line information, filename info, ...
2. Turns of all optimizations

Now we get information of where memory was allocated but wasn't freed

### Downsides
- It doesn't catch things that are not memory errors
- Bad at helping you find _why_ a problem occured


## GDB (Command Line Debugger)

**GDB commands**

`gdb <args to gdb> program`

- `run <args to prog> < infile > outfile`
    - Run the program 
- `backtrace` / `bt`
    - Stack trace
- `print <any C expression`
    - Print the value of expression
- `up`
    - Goes to the next function in the stack trace
- `down`
    - Goes the opposite direction of up
- `break <function name OR file:line>` / `b`
    - Pause everytime we reach the breakpoint
- `continue` / `c`
    - Start running again
- `next` 
    - Move forward one line
- `step`
    - Move forward one line and step into function calls