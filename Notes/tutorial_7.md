# Tutorial 7: Recursive Descent
**2017-11-01**

**Language:** set of words
**Word:** string of letters/symbols
**Letter/Symbol:** atomic "thing"
**Grammar:** rules to define a language

Our language - simple arithmetic

__Six operations:__ `+`, `-`, `*`, `/`, `^`, `!`
__Precendence:__
- `+`, `-` (lowest)
- `*`, `/`
- `^`, `=`
- `!`

Is `a + b * c` = `(a + b) * c` OR `a + (b * c)`

Arguments to operations are natural numbers.

I want to:
- Read a string from standard in
- Turn it into data structure
- Evaluate
- Pretty print in Reverse Polish Notation  

Idea:
- Parse the string into a "parse tree"
- **Parse tree** - "proof" tthat a word is in a language

An arithmetic expression (AE) is any of:
1. AE `+`/`-` AE such that `+`/`-` does _not_ occur in `()`   "top level" 
1. If there are no "top level" `+`/`-`, AE `*`/`/` AE 
1. No top level `*`/`/` => AE `^` AE (`^` TL)
1. AE`!` (`!` must be top)
1. (AE)
1. A number

Ex. `(1 + 2) * 2 ^ 4`

**Parse tree:**
```
    *
   / \
  ()  \
  |    \ 
  +     ^
 / \   / \
1   2 2   4
```

**Abstract Syntax Tree:** Parse tree minus the useless stuff
- Ex: () is not very helpful in our parse tree

**Ambiguity Problem:** `1 - 2 - 3`
- Could be `(1 - 2) - 3 = -4` or `1 - (2 - 3) = 2`

So we decide that everything is left assosciative (arbitrary decision) and modify rule 1 to account for this.

1. AE `+`/`-` (arithmetic expression WITHOUT `-`/`+`) such that `+`/`-` does _not_ occur in `()` "top level" 


