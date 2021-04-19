---
title: Constructors and destructors order with class hierarchies
date: 2021-04-19 12:49:40
tags: c++
---

> The C++ Programming Language 17.2.3 Base and Member Destructors

Constructors and destructors interact correctly with class hierarchies. A constructor builds a class object **from the bottom up**:

[1] first, the constructor invokes its base class constructors,
[2] then, it invokes the member constructors, and
[3] finally, it executes its own body.

A destructor **tears down** an object in the reverse order:

[1] first, the destructor executes its own body,
[2] then, it invokes its member destructors, and
[3] finally, it invokes its base class destructors.

In particular, a `virtual` base is constructed before any base that might use it and destroyed after all such bases. This ordering ensures that a base or a member is not used before it has been initialized or used after it has been destroyed. The programmer can defeat this simple and essential rule, but only through deliberate circumvention involving passing pointers to uninitialized variables as arguments. Doing so violates language rules and the results are usually disastrous.

Constructors execute member and base constructors in **declaration order** (not the order of initializers): if two constructors used a different order, the destructor could not (without serious overhead) guarantee to destroy in the reverse order of construction.

```cpp
struct X { X(int); };

struct S2 {
    X x;
};

S2 x1;     // error
S2 x2 {1}; // OK: x2.x is initialized with 1
```
