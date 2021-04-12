---
title: What's this extern "C" thing?
date: 2021-04-12 13:14:16
tags: c++
---

> [What's this extern "C" thing?](https://web.archive.org/web/20181115023158/http://www.comeaucomputing.com/techtalk/#externc)

Most C++ compilers do something called _name decoration_, or _name mangling_. In short, in order to support C++ function overloading, and C++ type-safe linkage, most C++ compilers will tack on some type encoding information onto the end of some of your identifiers. For instance, when Comeau C++ see this prototype

```cpp
void foo(int, double);
```

it internally changes the name to `foo__Fid`, where the `F` indicates that it is a function, the `i` indicates the first argument is an `int` and the `d` indicates that the second argument is a `double`. Other compilers do this in a similar manner. Since the definition of this function will be similarly decorated, the linker will resolve this symbol fine.

Now, consider something such as `printf` which is normally prototyped as follows:

```cpp
int printf(const char *, ...);
```

which hence might map into `printf__FPCce`. However, if `printf` was compiled with a C compiler, well, no C compiler that we know of does name decorating (although they could, they don't), therefore, the name will not be resolved by the linker. To get around this, in most cases, you should use a linkage specification. Here's the above example again:

```cpp
extern "C" void foo(int, double);
```

Now most implementations will not decorate `foo` any longer. Of course, you would `extern "C"` the function definition too (since the definition file should also be `#include`ing the header file which contains the prototype, that will be satisfactory as far as the compiler is concerned, though you should consider self-documenting code). As well, if you have many such functions, you can create linkage blocks, consider:

```cpp
// stdio.h
extern "C" {
    // ...
    int printf(const char *, ...);
    // ...
}
```

Here, most C++ compilers will not decorate `printf`'s name, therefore, it will probably now link with the binary version that was compiled by the C compiler. It's often useful to create a "co-ed" header file, so in such cases, this coding technique might be helpful:

```cpp
// stdio.h
#ifdef __cplusplus
extern "C" {
#endif

// stuff from before

#ifdef __cplusplus
}
#endif
```

Since Standard C is not required to define `__cplusplus`, then, when compiling with a C compiler, the extern block won't be established (but of course all the prototypes and such will be, which is as the C compiler would expect it).

Please note that name decoration is not required by C++, it is strictly an implementation detail. However, all C++ compilers do it. Similarly, a linkage specification should be considered a fighting chance at cross language linkage and not a guarantee. Again though, for most platforms, the reality is that it will work fine.

Too, do note that a linkage specification does not bring you into the other language! That is, you are still writing C++ code. As such, note that C++ keywords are still in existance even within a linkage specification -- so for instance, using `new` or `bool` or `private` as identifier names will end up being kicked out as errors; obviously then you'll need to rename those identifiers (you could do some preprocessor gymanstics but in the long run doing so usually does not pan out). Also note that doing something like passing a class based object, a reference, etc., to a C function means you are on your own. Note that other things effect name decoration too, such as a `class` name, `namespace`, etc. As well, you won't be overloading a function which you've `extern "C"`d because then you would have two functions with the same name (since most implementation wouldn't mangle them).

Also, doing this is usually a mistake:

```cpp
// ...
extern "C" { // wrapped around #include, but prefer within header
#include "SomeHeaderFile.h"
}
// ...
```

because if the header is not prepared for being `extern "C"`d then doing the above is most likely just going to result in a bunch of error messages, and probably cryptically so too. Chances are good that the header will drag in other headers and bad fun will just ensue from there with nested headers, typedefs and who knows what else. Your `extern "C"`ing should follow the Las Vegas tenet _What happens in extern "C" stays in extern "C"_ and **keep your `extern "C"` blocks within files not across them**.

The above has so far considered only the scenario of calling a C function from C++. The contrary, calling a C++ function from a C function, has the same solution. In other words, if you `extern "C"` a C++ function, then most implementations won't mangle it, therefore, most C compilers will be able to link with it. However, as just mentioned, if the C++ function expects something such as a reference argument, you are usually of course on your own. There's other issues too in this direction, for instance, consideration of C++'s overloading, the function expecting a C++ class based object as a parameter, etc. If it's not clear, C does not have all the features available in C++, and trying to mimic them (the calling routine(s) from C will have to do this) can be challenging to say the least in some cases. This all tends to complete with calling a C++ function from C, even with `extern "C"`. In some cases, it may be worth considering stubs routines to try and ease the pain, but this should be decided carefully. Of course as well, a C++ application can use C++ functions which have been `extern "C"`d.

There are other aspect to linkage specifications that you might want to be aware of. For instance, it can be applied to objects. With the above under your belt, you may now want to pursue a quality C++ text book on this matter.
