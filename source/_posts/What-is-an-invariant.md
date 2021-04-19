---
title: What is an invariant?
date: 2021-04-19 12:34:09
tags: c++
---

> https://stackoverflow.com/questions/112064/what-is-an-invariant

> The word seems to get used in a number of contexts. The best I can figure is that they mean a variable that can't change. Isn't that what constants/finals (darn you Java!) are for?

An **invariant** is more "conceptual" than a variable. In general, it's a property of the program state that is always true. A function or method that ensures that the invariant holds is said to maintain the invariant.

For instance, a binary search tree might have the invariant that for every node, the key of the node's left child is less than the node's own key. A correctly written insertion function for this tree will maintain that invariant.

As you can tell, that's not the sort of thing you can store in a variable: it's more a statement about the program. By figuring out what sort of invariants your program should maintain, then reviewing your code to make sure that it actually maintains those invariants, you can avoid logical errors in your code.

Reference: [Wikipedia Invariants in computer science](https://en.wikipedia.org/wiki/Invariant_(mathematics)#Invariants_in_computer_science)
