---
title: When to use virtual destructors?
date: 2021-04-19 13:41:17
tags: c++
---

> https://stackoverflow.com/questions/461203/when-to-use-virtual-destructors

> I have a solid understanding of most OOP theory but the one thing that confuses me a lot is virtual destructors.
> I thought that the destructor always gets called no matter what and for every object in the chain.
> When are you meant to make them virtual and why?

Virtual destructors are useful when you might potentially delete an instance of a derived class through a pointer to base class:

```cpp
class Base 
{
    // some virtual methods
};

class Derived : public Base
{
    ~Derived()
    {
        // Do some important cleanup
    }
};
```

Here, you'll notice that I didn't declare Base's destructor to be `virtual`. Now, let's have a look at the following snippet:

```cpp
Base *b = new Derived();
delete b; // Here's the problem!
```

Since Base's destructor is not `virtual` and `b` is a `Base*` pointing to a Derived object, `delete b` has undefined behaviour:

> [In `delete b`], if the static type of the object to be deleted is different from its dynamic type, the static type shall be a base class of the dynamic type of the object to be deleted and **the static type shall have a virtual destructor or the behavior is undefined**.

In most implementations, the call to the destructor will be resolved like any non-virtual code, meaning that the destructor of the base class will be called but not the one of the derived class, resulting in a resources leak.

To sum up, always make base classes' destructors `virtual` when they're meant to be manipulated polymorphically.

If you want to prevent the deletion of an instance through a base class pointer, you can make the base class destructor protected and nonvirtual; by doing so, the compiler won't let you call `delete` on a base class pointer.

You can learn more about virtuality and `virtual` base class destructor in [this article from Herb Sutter](http://www.gotw.ca/publications/mill18.htm).
