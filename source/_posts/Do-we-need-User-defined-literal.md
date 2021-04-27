---
title: Do we need User-defined literal?
date: 2021-04-27 13:22:52
tags: c++
---

> https://stackoverflow.com/a/238057/6799372

At first sight, it seems to be simple syntactic sugar.

But when looking deeper, we see it's more than syntactic sugar, as **it extends the C++ user's options to create user-defined types that behave exactly like distinct built-in types**. In this, this little "bonus" is a very interesting C++11 addition to C++.

# Do we really need it in C++?

I see few uses in the code I wrote in the past years, but just because I didn't use it in C++ doesn't mean it's not interesting for _another C++ developer_.

We had used in C++ (and in C, I guess), compiler-defined literals, to type integer numbers as `short` or `long` integers, real numbers as `float` or `double` (or even `long double`), and character strings as normal or wide chars.

**In C++, we had the possibility to create our own types (i.e. classes), with potentially no overhead (inlining, etc.).** We had the possibility to add operators to their types, to have them behave like similar built-in types, which enables C++ developers to use matrices and complex numbers as naturally as they would have if these have been added to the language itself. We can even add cast operators (which is usually a bad idea, but sometimes, it's just the right solution).

**We still missed one thing to have user-types behave as built-in types: user-defined literals.**

So, I guess it's a natural evolution for the language, but to be as complete as possible: "_If you want to create a type, and you want it to behave as much possible as a built-in types, here are the tools..._"

I'd guess it's very similar to .NET's decision to make every primitive a struct, including booleans, integers, etc., and have all structs derive from Object. This decision alone puts .NET far beyond Java's reach when working with primitives, no matter how much boxing/unboxing hacks Java will add to its specification.

# Do YOU really need it in C++?

This question is for **YOU** to answer. Not Bjarne Stroustrup. Not Herb Sutter. Not whatever member of C++ standard committee. This is why you have the choice in C++, and they won't restrict a useful notation to built-in types alone.

If you need it, then it is a welcome addition. If you don't, well... Don't use it. It will cost you nothing.

Welcome to C++, the language where features are optional.

## Bloated??? Show me your complexes!!!

There is a difference between bloated and complex (pun intended).

Like shown by Niels at [What new capabilities do user-defined literals add to C++?](https://stackoverflow.com/questions/237804/what-new-capabilities-do-user-defined-literals-add-to-c#237821), being able to write a complex number is one of the two features added "recently" to C and C++:

```cpp
// C89:
MyComplex z1 = { 1, 2 } ;

// C99: You'll note I is a macro, which can lead
// to very interesting situations...
double complex z1 = 1 + 2*I;

// C++:
std::complex<double> z1(1, 2) ;

// C++11: You'll note that "i" won't ever bother
// you elsewhere
std::complex<double> z1 = 1 + 2_i ;
```

Now, both C99 `double complex` type and C++ `std::complex` type are able to be multiplied, added, subtracted, etc., using operator overloading.

But in C99, they just added another type as a built-in type, and built-in operator overloading support. And they added another built-in literal feature.

In C++, they just used existing features of the language, saw that the literal feature was a natural evolution of the language, and thus added it.

In C, if you need the same notation enhancement for another type, you're out of luck until your lobbying to add your quantum wave functions (or 3D points, or whatever basic type you're using in your field of work) to the C standard as a built-in type succeeds.

In C++11, you just can do it yourself:

```cpp
Point p = 25_x + 13_y + 3_z ; // 3D point
```

**Is it bloated? No**, the need is there, as shown by how both C and C++ complexes need a way to represent their literal complex values.

**Is it wrongly designed? No**, it's designed as every other C++ feature, with extensibility in mind.

**Is it for notation purposes only? No**, as it can even add type safety to your code.

For example, let's imagine a CSS oriented code:

```cpp
css::Font::Size p0 = 12_pt ;       // Ok
css::Font::Size p1 = 50_percent ;  // Ok
css::Font::Size p2 = 15_px ;       // Ok
css::Font::Size p3 = 10_em ;       // Ok
css::Font::Size p4 = 15 ;          // ERROR : Won't compile !
```

It is then very easy to enforce a strong typing to the assignment of values.

## Is is dangerous?

Good question. Can these functions be namespaced? If yes, then Jackpot!

Anyway, **like everything, you can kill yourself if a tool is used improperly**. C is powerful, and you can shoot your head off if you misuse the C gun. C++ has the C gun, but also the scalpel, the taser, and whatever other tool you'll find in the toolkit. You can misuse the scalpel and bleed yourself to death. Or you can build very elegant and robust code.

So, like every C++ feature, do you really need it? It is the question you must answer before using it in C++. If you don't, it will cost you nothing. But if you do really need it, at least, the language won't let you down.

## The date example?

Your error, it seems to me, is that you are mixing operators:

```cpp
1974/01/06AD
    ^  ^  ^
```

This can't be avoided, because `/` being an operator, the compiler must interpret it. And, AFAIK, it is a good thing.

To find a solution for your problem, I would write the literal in some other way. For example:

```cpp
"1974-01-06"_AD ;   // ISO-like notation
"06/01/1974"_AD ;   // french-date-like notation
"jan 06 1974"_AD ;  // US-date-like notation
19740106_AD ;       // integer-date-like notation
```

Personally, I would choose the integer and the ISO dates, but it depends on YOUR needs. Which is the whole point of letting the user define its own literal names.

