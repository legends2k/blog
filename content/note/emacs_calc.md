+++
title = "Calculator Cheat Sheet"
description = "Emacs Calc Hacks"
date = 2024-03-30T06:17:52+05:30
tags = [ "tech", "emacs", "tools" ]
mathjax = true
+++

I use the GNU [Emacs Calc][], an [RPN][] calculator, exclusively unless I’m away from a computer; as a programming, scientific and general calculator.  The manual documents all features extensively, with examples too; this is my cheat sheet for my oft-used functions.

[rpn]: https://en.wikipedia.org/wiki/Reverse_Polish_notation
[emacs calc]: https://www.gnu.org/software/emacs/manual/html_mono/calc.html

# Programming

## Binary

* Set binary radix: `d 2`
  - Prefix with `O` to display two’s complement as `2##1100` instead of `-2#0100`
    + Notice `#`-count for disambiguation
  - Other useful radices: `16`, `0` and `8`
* Set word size (32 bits by default) before doing anything interesting: `b w`
  - Set negative word size if you want to work on signed numbers
    + Arithmetic `-1320 >> 4` = `65530` (base = 16) instead of `-83` (base = -16)
  - Manually clip values to $2^w$: `b c`
  - Set word size per-operation (except shifts/rotates) with prefix: `C-u 4`
    + Example: `C-u 4 b n` flips bits and clips to $2^4$
* Enable leading zero display: `d z`
  - Useful to see the entire word
* Enable digit grouping: `d g`
* Input numbers in a particular radix: `R#DIGITS` e.g. `16#f00ba4`
* Most useful binary operations start with prefix `b`
  - Binary NOT: `b n`
  - Binary AND, OR, XOR, DIFF: `b a`, `b o`, `b x`, `b d`
    + Takes two operands from stack
    + DIFF is actually bits set in `X` but not in `Y` i.e. `X & ~Y`
  - Shift: `b l`, `b r`, `b R`, `b L`
    + Takes 1 or prefix as second operand; use top-of-stack with Hyperbolic prefix `H` 
    + Uppercase does arithmetic shift; arithmetic left shift makes sense with negative arguments
  - Rotate: `b t`
    + Takes 1 or prefix as second operand; use top-of-stack with Hyperbolic prefix `H` 
    + Use negative second operand for rotate-right

## Misc

* Integer divide quotient / reminder: `\` / `%`
* Floor, Ceil, Round: `F`, `I F`, `R`
* Vector
  - (Euclidean) Length / l-2 norm: `A`
  - Dot / Cross products: `*` / `V C`
  - Determinant / Inverse / Transpose: `V D` / `&` / `v t`
  - Element-wise add (map): `V M +`
  - Sum elements (reduce): `V R +`
  - Pack / Unpack / Reverse: `v p` / `v u` / `v v`
* Convert to new unit / base unit: `u c` / `u b`
  - Input with unit using algebraic input e.g. `' 60 km/hr`
    + Temperature units: `degC`, `degF`, `K`
* Prime numbers
  - Test prime: `k p`
  - Prime factorize: `k f`
  - Next/Previous prime: `k n`, `I k n`
* Random number: `k r`
  - Between [0, M) or (M, 0] depending on sign(M); M is top-of-stack or prefix
  - Use `1.` (mind the decimal point) to generate numbers between [0, 1)
  - Random again (same M): `k a`
* Shuffle $\binom{n}{k}$: `k h`
  - Takes `n` and `k` from stack and returns a list
  - Shuffle top-of-stack list with prefix: `C-u -1`
* Permutation (${}_n\mathrm{ P }_k$) and Combination (${}_n\mathrm{ C }_k$): `H k c`, `k c`
  - Factorial: `!`

# Scientific

* Get constants

| Key     | Constant | Name                 | Value          |
|:--------|:--------:|:---------------------|:---------------|
| `P`     | $\pi$    | [Pi][]               | 3.14159265359  |
| `H P`   | $e$      | [Euler’s number][]   | 2.71828182846  |
| `I P`   | $\gamma$ | [Euler’s constant][] | 0.577215664902 |
| `I H P` | $\phi$   | [Golden ratio][]     | 1.61803398875  |

* Set precision: `p`
  - Floating-point / Fixed-point / Scientific notation: `d n` / `d f` / `d s`
* Log to top-of-stack as base: `B`
* Square root: `Q`
  - n-th root: `I ^`
* Reciprocal: `&`
* Convert to fraction/float: `c F` / `c f`
* Change to radians/degrees: `m r` / `m d`
* Calculate sin and cos: `calc-sincos`

# General

* Edit top-of-stack: backtick
* Flip top-of-stack and previous
* Rotate stack: `C-u 0 ESC-TAB`

[euler’s number]: https://en.wikipedia.org/wiki/E_(mathematical_constant)
[golden ratio]: https://en.wikipedia.org/wiki/Golden_ratio
[euler’s constant]: https://en.wikipedia.org/wiki/Euler%27s_constant
[pi]: https://en.wikipedia.org/wiki/Pi
