---
title: What is the difference between a concrete class and an abstract class?
date: 2021-04-19 11:28:51
tags: c++
---

> https://stackoverflow.com/questions/2149207/what-is-the-difference-between-a-concrete-class-and-an-abstract-class

> I am learning C++, but I am confused about abstract class and concrete class. Some real world examples would be appreciated.
> what is the difference between normal derived class and concrete class?

An **abstract class** is a class for which one or more methods are declared but not defined, meaning that the compiler knows _these methods are part of the class, but not what code to execute for that method_. These are called abstract methods. Here is an example of an abstract class.

```cpp
class shape {
public:
  virtual void draw() = 0;
};
```

This declares an abstract class which specifies that any descendants of the class should implement the `draw` method if the class is to be **concrete**. You cannot instantiate this class because it is abstract, after all, the compiler wouldn't know what code to execute if you called member `draw`. So you can not do the following:

```cpp
shape my_shape();
my_shape.draw();
```

To be able to actually use the draw method you would need to _derive_ classes from this abstract class, which do implement the `draw` method, making the classes concrete:

```cpp
class circle : public shape {
public:
  circle(int x, int y, int radius) {
    /* set up the circle */
  }
  virtual draw() {
    /* do stuff to draw the circle */
  }
};

class rectangle : public shape {
public:
  rectangle(int min_x, int min_y, int max_x, int max_y) {
    /* set up rectangle */
  }
  virtual draw() {
    /* do stuff to draw the rectangle */
  }
};
```

Now you can instantiate the concrete objects `circle` and `rectangle` and use their `draw` methods:

```cpp
circle my_circle(40, 30, 10);
rectangle my_rectangle(20, 10, 50, 15);
my_circle.draw();
my_rectangle.draw();
```

Now of course the question is, why would you want to do this? Couldn't you just as well have defined the `circle` and `rectangle` classes and have done away with the whole shape class? You could, but then you wouldn't be able to take advantage of their inheritance:

```
std::vector<shape*> my_scene;
my_scene.push_back(new circle(40, 30, 10));
my_scene.push_back(new rectangle(20, 10, 50, 15));
std::for_each(my_scene.begin(), my_scene.end(), std::mem_fun_ref(&shape::draw)
```

This code let's you collect all your shapes into one container. This makes it a lot easier if you have a lot of shapes and many different shapes in your scene. For example we can now draw all the shapes in one go, and the code that does so doesn't even need to know about the different types of shapes we have.

Now finally we need to know why the `draw` function of `shape` is abstract, and not just an empty function, i.e. why didn't we just define:

```cpp
class shape {
public:
  virtual void draw() {
    /* do nothing */
  }
};
```

The reason for this is that we don't really want objects of type `shape`, they wouldn't be real things anyway, they would be abstract. So it doesn't make any sense to define an implementation for the `draw` method, even an empty one. Making the `shape` class abstract prevents us from mistakenly instantiating the `shape` class, or mistakenly calling the empty `draw` function of the base class instead of the `draw` function of the derived classes. In effect we define an interface for any class that would like to behave like a shape, we say that any such class should have a `draw` method that looks like we have specified it should.

To answer you last question, there isn't any such thing as a 'normal derived class' _every class is either abstract or concrete_. A class that has any abstract methods is abstract, any class that doesn't is concrete. It's just a way to differentiate the two types of classes. A base class can be either abstract or concrete and a derived class can be either abstract or concrete:

```cpp
class abstract_base {
public:
  virtual void abstract_method1() = 0;
  virtual void abstract_method2() = 0;
};

class concrete_base {
public:
  void concrete_method1() {
    /* do something */
  }
};

class abstract_derived1 : public abstract_base {
public:
  virtual void abstract_method3() = 0;
};

class abstract_derived2 : public concrete_base {
public:
  virtual void abstract_method3() = 0;
};

class abstract_derived3 : public abstract_base {
public:
  virtual abstract_method1() {
    /* do something */
  }
  /* note that we do not provide an implementation for
     abstract_method2 so the class is still abstract */
};

class concrete_derived1 : public concrete_base {
public:
  void concrete_method2() {
    /* do something */
  }
};

class concrete_derived2 : public abstract_base {
public:
  virtual void abstract_method1() {
    /* do something */
  }
  virtual void abstract_method2() {
    /* do something */
  }
  /* This class is now concrete because no abstract methods remain */
};
```
