---
title: What is the use of having destructor as private?
date: 2021-04-19 13:12:57
tags: c++
---

> https://stackoverflow.com/questions/631783/what-is-the-use-of-having-destructor-as-private

Basically, any time you want some other class to be responsible for the life cycle of your class' objects, or you have reason to prevent the destruction of an object, you can make the destructor `private`.

For instance, if you're doing some sort of reference counting thing, you can have the object (or manager that has been `friend`ed) responsible for counting the number of references to itself and delete it when the number hits zero. A `private` dtor would prevent anybody else from deleting it when there were still references to it.

For another instance, what if you have an object that has a manager (or itself) that may destroy it or may decline to destroy it depending on other conditions in the program, such as a database connection being open or a file being written. You could have a `request_delete` method in the class or the manager that will check that condition and it will either delete or decline, and return a status telling you what it did. That's far more flexible that just calling "delete".

Such an object **can only be created on the stack in the scope of a `friend` or itself**. Additionally it cannot ba a static or global object. And deletion has to be done via a `friend` or a member. A product may use a single Object hierarchy and a custom memory-manager -- such scenarios may use a `private` dtor. **Such an object can be created on the stack by using placement `new`.**

```cpp
#include <iostream>

class a {
    ~a() {}
    friend void delete_a(a* p);
};


void delete_a(a* p)  {
    delete p;
}

int main()
{
    a *p = new a;
    delete_a(p);

    return 0;
}
```

Example from The C++ Programming Language 17.2.4 Calling Constructors and Destructors.

```cpp
class Nonlocal {
public:
    void destroy() { this−> ̃Nonlocal(); }
private:
     ̃Nonlocal();
};

void user() {
    Nonlocal x;            // error: cannot destory a Nonlocal
    X* p = new Nonlocal;   // OK
	
	delete p;              // error: cannot destroy a Nonlocal
	p.destroy();           // OK
}
```
