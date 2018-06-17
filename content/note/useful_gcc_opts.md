+++
title = "Useful GCC options"
description = "Know thy compiler!"
date = "2017-02-08T16:45:02+05:30"
tags = ["tech", "gcc", "tools"]

+++

Here are some of the useful, but not widely known, options of GCC that I want to document.

    gcc -dumpspecs

Gives the specifications with which GCC was built.

    gcc -fstack-usage

Gives the stack usage function-wise for the compiled translation unit; this is helpful in measuring runtime memory usage.  [See the manual][stack_usage] for details on deciphering its output.

    # get macros defined when the language is C++
    cpp -xc++ -dM /dev/null

    # get macros defined when the language is C
    cpp -dM /dev/null

[stack_usage]: https://gcc.gnu.org/onlinedocs/gcc/Developer-Options.html#index-fstack-usage

Prints the macros defined when the preprocessor was called.  The last command [is popular][macro_SO_question] as `gcc -dM -E - << /dev/null` but is not as good for two reasons:

1. If you want to talk to the preprocessor, talk directly to it, why go through the compiler?
2. Looking at this, one might get a false hope that `g++ -dM -E - << /dev/null` will spit C++ macros; it doesn't.  Instead one has to do `g++ -xc++ -dM -E - << /dev/null` for [some reasons][macro_g++].

[macro_SO_question]: http://stackoverflow.com/a/2224357/183120
[macro_g++]: http://stackoverflow.com/a/27980787/183120

For setting up auto completion in Emacs using Irony, I needed to know the include directories GCC searches.  How do we find them?

    cpp -xc++ -Wp,-v /dev/null

This prints the list of standard include directories.  `g++ -v some.cpp` would give it too, but this is fast & easy; no input file or fake compiler calls.

    g++ -std=c++14 -O3 -c -masm=intel -fverbose-asm -Wa,-adhln=prgm.s prgm.cpp

will show the disassembly of a compilation unit in Intel syntax *with inter-weaved source listing*.  This one is very popular among optimisation enthusiasts 😃  Here's one more for them:

    g++ -fdump-class-hierarchy my_classes.cpp

This would show object memory layout of classes in the source; includes classes with complex inheritance hierarchies.

What interesting GCC options do you know?
