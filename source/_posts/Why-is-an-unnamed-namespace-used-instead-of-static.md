---
title: Why is an unnamed namespace used instead of static?
date: 2021-04-12 10:43:45
tags: c++
---

> Copy from [Why is an unnamed namespace used instead of static?](https://web.archive.org/web/20181115023158/http://www.comeaucomputing.com/techtalk/#nostatic)

in C++, some uses of the `static` keyword have been **deprecated**. In particular, an unnamed namespace should be favored over some previous uses of "file scope static's". In fact in some cases an unnamed namespace must be used in order to obtain a similar effect. That is to say, this code:

```cpp
// x.cpp
static bool flag = false; // AAA

void foo() { if (flag)... }
void bar() { ...flag = true... }
```

should instead often be composed this way in C++:

```cpp
// x.cpp
namespace /* NOTHING HERE!! */ { // BBB
    bool flag = false; // no need for static here
}
```

The use of static in `AAA` indicates that flag has internal linkage. This means that flag is local to its translation unit (that is, effectively it is only known by its name in some source file, in this case `x.cpp`). This means that flag can't be used by another translation unit (by its name at least). The goal is to have less global/cross-file name pollution in your programs while at the same time achieving some level of encapsulation. Such a goal is usually considered admirable and so therefore is often considered desirable (note that the goal, not the code, is being discussed in this sentence).

Contrast this to `BBB`. In the case of using the unnamed namespace above, flag has external linkage, yet it is effectively local to the translation unit. It is effectively still local because although we did not give the namespace a name, the compiler generated a unique name for it. In effect, the compiler changes `BBB` into this:

```cpp
// Just get UNIQUE established
namespace UNIQUE { } // CCC
// Bring UNIQUE into this translation unit
using namespace UNIQUE;
// Now define UNIQUEs members
namespace UNIQUE {
    bool flag = false; // As Before
}
```

For each translation unit, a uniquely generated identifier name for `UNIQUE` somehow gets synthesized by the compiler, with the effect that no other translation unit can see names from an unnamed namespace, hence making it local even though the name may have external linkage.

Therefore, although flag in `CCC` has external linkage, its real name is `UNIQUE::flag`, but since `UNIQUE` is only known to `x.cpp`, it's effectively local to `x.cpp` and is therefore not known to any other translation unit.

Ok, so far, most of the discussion has been about how the two provide local names, but what are the differences? And why was static deprecated and the unnamed namespace considered superior?

First, if nothing else, static means many different things in C++ and reducing one such use is considered a step in the right direction by some people. Second to consider is that names in unnamed namespaces may have external linkage whereas with static a name must have internal linkage. In other words, although there is a syntactic transformation shown above between `AAA` and `BBB`, the two are not exactly equal (the one between `BBB` and `CCC` is equal).

Most books and usenet posts usually leave you off about right here. No problem with that per se, as the above info is not to be tossed out the window. However, you can't help but keep wondering what the **BIG** deal some people make about unnamed namespaces are. Some folks might even argue that they make your code less readable.

What's significant though is that **some template arguments cannot be names with internal linkage**, instead some require names with external linkage. Remember, the types of the arguments to templates become part of the instantiation type, but names with internal linkage aren't available to other translation units. A good rule of thumb to consider (said rather loosely) is that external names shouldn't depend upon names with less linkage (definitely not of those with no linkage, and often not even w ith names of internal linkage). And so it follows from the above that instantiating such a template with a static such as from AAA just isn't going to work. This is all similar to why these won't work:

```cpp
template <const int& T> struct xyz { };

int c = 1;
xyz<c> y; // ok
static int sc = 1; // This is the kicker-out'er above
xyz<sc> y2; // not ok
```

```cpp
template <char *p> struct abc { };

char comeau[] = "Comeau C++";
abc<comeau> co; // ok
abc<"Comeau C++"> co2; // not ok
```

```cpp
template <typename T> struct qaz { };
void foo()
{
    char buf[] = "local";
    abc<buf> lb; // not ok
    static char buf2[] = "local";
    abc<buf2> lb2; // not ok

    struct qwerty {};
    qaz<qwerty> dq; // not ok
}
```

Last but not least, `static` and unnamed namespaces are not the same because `static` is deficient as a name constrainer. Sure, a C programmer might use it for flag above, but what do you do when you want to layer or just encapsulate say a class, template, enum, or even another namespace? ...for that you need namespaces. It might even be argued that you should wrap all your files in an unnamed namespace (all the file's functions, classes, whatever) and then only pull out the parts other files should know about.

Draw your attention to that that none of the above is equal to this:

```cpp
// x.cpp
namespace { // DDD
    static bool flag = false;
}
```

(The point of showing `DDD` is that you really wouldn't want to say it. I guess one could say that it is redundant, and really just brings all the above issues right back (as in this flavor, it's not external). So, it's only shown to make sure you see that none of the previous versions look like this :).

Note that namespaces containing `extern "C"` declarations are in some ways as if they were not declared in the namespace, so since an unnamed namespace is a namespace, this holds true for an unnamed namespace as well.

Note also that the above discussion does not apply to the other uses of static: static lifetime, static members or to static locals in a function.
