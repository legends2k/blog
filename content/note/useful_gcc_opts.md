+++
title = "Useful GCC options"
description = "Know thy compiler!"
date = "2017-02-08T16:45:02+05:30"
tags = ["tech", "gcc", "tools"]

+++

Here are some of the useful, but not widely known, options of GCC that I want to document.

# GCC Build Specs

{{< highlight basic >}}
gcc -dumpspecs
{{< /highlight >}}

Gives the specifications with which GCC was built.

# Stack Usage

{{< highlight basic >}}
gcc -fstack-usage
{{< /highlight >}}

Gives the stack usage function-wise for the compiled translation unit; this is helpful in measuring runtime memory usage.  [See the manual][stack_usage] for details on deciphering its output.

# Macros and Include Directories

{{< highlight basic >}}
# get macros defined when the language is C++
cpp -xc++ -dM /dev/null

# get macros defined when the language is C
cpp -dM /dev/null
{{< /highlight >}}

[stack_usage]: https://gcc.gnu.org/onlinedocs/gcc/Developer-Options.html#index-fstack-usage

Prints the macros defined when the preprocessor was called.  The last command [is popular][macro_SO_question] as `gcc -dM -E - << /dev/null` but is not as good for two reasons:

1. If you want to talk to the preprocessor, talk directly to it, why go through the compiler?
2. Looking at this, one might get a false hope that `g++ -dM -E - << /dev/null` will spit C++ macros; it doesn't.  Instead one has to do `g++ -xc++ -dM -E - << /dev/null` for [some reasons][macro_g++].

[macro_SO_question]: http://stackoverflow.com/a/2224357/183120
[macro_g++]: http://stackoverflow.com/a/27980787/183120

For setting up auto completion in Emacs using Irony, I needed to know the include directories GCC searches.  How do we find them?

{{< highlight basic >}}
cpp -xc++ -Wp,-v /dev/null
{{< /highlight >}}

This prints the list of standard include directories.  `g++ -v some.cpp` would give it too, but this is fast & easy; no input file or fake compiler calls.

# Disassembly

{{< highlight basic >}}
g++ -std=c++14 -O3 -c -masm=intel -fverbose-asm -Wa,-adhln=prgm.s prgm.cpp
{{< /highlight >}}

will show the disassembly of a compilation unit in Intel syntax *with inter-weaved source listing*.  This one is very popular among optimisation enthusiasts 😃  Here's one more for them.

# Class Memory Layout

{{< highlight basic >}}
g++ -fdump-class-hierarchy my_classes.cpp
{{< /highlight >}}

This would show object memory layout of classes in the source; includes classes with complex inheritance hierarchies.

# Optimization Result

With GCC 9 you can check if an optimization was performed or missed ([`-fopt-info`][opt-info]). Example

{{< highlight basic >}}
$ g++ -c inline.cc -O2 -fopt-info-inline-all
inline.cc:24:11: note: Considering inline candidate void foreach(T, T, void (*)(E)) [with T = char**; E = char*]/2.
inline.cc:24:11: optimized:  Inlining void foreach(T, T, void (*)(E)) [with T = char**; E = char*]/2 into int main(int, char**)/1.
inline.cc:19:12: missed:   not inlinable: void inline_me(char*)/0 -> int std::puts(const char*)/3, function body not available
inline.cc:13:8: optimized:  Inlined void inline_me(char*)/4 into int main(int, char**)/1 which now has time 127.363637 and size 11, net change of +0.
Unit growth for small function inlining: 16->16 (0%)

Inlined 2 calls, eliminated 1 functions
{{< /highlight >}}

[opt-info]: https://gcc.gnu.org/onlinedocs/gcc-9.1.0/gcc/Developer-Options.html#index-fopt-info

# Vectorization

Use `-ftree-vectorize` to enable vectorization of loops; this is implicitly enabled at `-O3`.

# Sanitizers

If you’re a C or C++ programmer, you should definitely try the sanitizers GCC has ([`-fsanitize`][instrumentation]).  Some interesting ones

* Address sanitizer (`-fsanitize=address`)
* Leak sanitizer (`-fsanitize=leak`)
* Thread sanitizer (`-fsanitize=thread`)
* Undefined Behaviour sanitizer (`-fsanitize=undefined`)

What interesting GCC options do you know?

[instrumentation]: https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html
