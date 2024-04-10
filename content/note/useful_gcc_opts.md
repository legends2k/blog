+++
title = "Useful GCC options"
description = "Know thy compiler!"
date = "2017-02-08T16:45:02+05:30"
tags = ["tech", "gcc", "tools"]
toc = true
+++

Here are some of the useful, but not widely known, options of GCC that I want to document.

# Output preprocessor output, assembly and object code

{{< highlight basic >}}
gcc -save-temp=obj -o hello main.cpp
{{< /highlight >}}

outputs the following in addition to `hello`

1. `hello-main.ii`: annotated output of preprocessor
2. `hello-main.s`: assembly
3. `hello-main.o`: object code

# GCC Build Specs

{{< highlight basic >}}
gcc -dumpspecs
{{< /highlight >}}

Gives the specifications with which GCC was built.

# List supported targets

{{< highlight basic >}}
gcc --target-help
gcc --help=target
{{< /highlight >}}

clang has `clang -print-supported-cpus` and `clang -print-targets` similarly.  Though clang operates on [target triples][]/quadruples.

On older GCC [alternate command][old-gcc-march] is `gcc -E -march=help -xc /dev/null`.

[target triples]: https://wiki.osdev.org/Target_Triplet
[old-gcc-march]: https://stackoverflow.com/q/47299458/183120

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

# print GCC search directories for binaries and libraries
gcc -print-search-dirs
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

This prints the list of standard include directories.  `g++ -v some.cpp` would give it too, but this is fast & easy; no input file or fake compiler calls.  Another similar option is `g++ -### some.cpp`.

# Disassembly

{{< highlight basic >}}
g++ -std=c++14 -O3 -c -masm=intel -fverbose-asm -Wa,-adhln=prgm.s prgm.cpp
{{< /highlight >}}

will show the disassembly of a compilation unit in Intel syntax *with inter-weaved source listing*.  This one is very popular among optimisation enthusiasts 😃  Here's one more for them.

# Class Memory Layout

{{< highlight basic >}}
g++ -fdump-class-hierarchy my_classes.cpp
{{< /highlight >}}

This would show object memory layout of classes in the source; includes classes with complex inheritance hierarchies.  Padding can needlessly increase the size of a structure; use `-Wpadded` to reveal when padding bits are added.

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

A language as low-level and powerful C++ is comes with its own set of challenges.  Many tools have been built over the years to help programmers; sanitizers are an important set of tools GCC has ([`-fsanitize`][instrumentation]).  Some interesting ones

* Address sanitizer (`-fsanitize=address -static-libasan`)
* Leak sanitizer (`-fsanitize=leak -static-liblsan`)
* Thread sanitizer (`-fsanitize=thread -static-libtsan`)
* Undefined Behaviour sanitizer (`-fsanitize=undefined -static-libubsan`)

Related: [Gavin’s blog post][san-blog].

# Header Dependency Tree

There’re times when multiple headers with interlinked dependencies are a problem.  It’d be easier to understand why a definition is deemed missing by the compiler, despite including a header you _think_ should’ve made it visible.  These preprocessor options are your friends:

* `-M` show dependencies for all headers
* `-MM` show dependencies for non-system headers
* `-H` show dependencies as a hierarchy

{{< highlight basic >}}
> g++ -Wall -stc=c++17 -pedantic -H test.cpp

. /…/XcodeDefault.xctoolchain/usr/include/c++/v1/cmath
.. /…/XcodeDefault.xctoolchain/usr/include/c++/v1/__config
.. /…/XcodeDefault.xctoolchain/usr/include/c++/v1/math.h
... /…/XcodeDefault.xctoolchain/usr/include/c++/v1/__config
... /…/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/math.h
{{< /highlight >}}

# Old compilers + new flags

GCC doesn’t seem to have this feature _yet_.

When an unknown flag is passed to Clang, it’ll raise a warning by default; projects often use `-Werror` to promote warnings to errors.  If using a new Clang-16-introduced flag (e.g. `-Wno-enum-constexpr-conversion`), unfamiliar to older versions, but still want the project be build-able using older compiler versions, use `-Wno-unknown-warning-option`.  Older versions will silently ignore the unknown (but new) flag.  Newer versions will recognize and honour both.

A small caveat: using this flag means even really incorrect flags passed to Clang will go unwarned.  Perhaps using this for the short run is a better middle ground.


What interesting GCC options do you know?

[instrumentation]: https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html
[san-blog]: https://gavinchou.github.io/experience/summary/syntax/gcc-address-sanitizer/

