---
title: What uses are there for "placement new"?
date: 2021-04-19 12:59:00
tags: c++
---

> https://stackoverflow.com/questions/222557/what-uses-are-there-for-placement-new

> Has anyone here ever used C++'s "placement new"? If so, what for? It looks to me like it would only be useful on memory-mapped hardware.

Placement new allows you to **construct an object in memory that's already allocated**.

You may want to do this for optimization when you need to construct multiple instances of an object, and it is faster not to re-allocate memory each time you need a new instance. Instead, it might be more efficient to perform a single allocation for a chunk of memory that can hold multiple objects, even though you don't want to use all of it at once.

DevX gives a [good example](http://www.devx.com/tips/Tip/12582):

> Standard C++ also supports placement new operator, which constructs an object on a pre-allocated buffer. This is useful when building a memory pool, a garbage collector or simply when performance and exception safety are paramount (there's no danger of allocation failure since the memory has already been allocated, and constructing an object on a pre-allocated buffer takes less time):

```cpp
char *buf  = new char[sizeof(string)]; // pre-allocated buffer
string *p = new (buf) string("hi");    // placement new
string *q = new string("hi");          // ordinary heap allocation
```

You may also want to be sure there can be _no allocation failure at a certain part of critical code_ (for instance, in code executed by a pacemaker). In that case you would want to allocate memory earlier, then use placement new within the critical section.

Deallocation in placement new
You should not deallocate every object that is using the memory buffer. Instead you should `delete[]` only the original buffer. You would have to then call the destructors of your classes manually. For a good suggestion on this, please see Stroustrup's FAQ on: [Is there a "placement delete"?](https://www.stroustrup.com/bs_faq2.html#placement-delete).
