---
title: "Dataclasses in Python and C++"
description: How to use dataclasses in python and their closest equivalent in C++
slug: dataclasses-python
date: 2023-10-24T18:34:22-07:00
categories:
  - python
  - c++
  - recipes
tags:
  - python
  - c++
  - recipes
---

## Introduction

If you write class definitions and then proceed to write their getters, setters, string representations and generate hundreds of lines of code, this post is for you.

### One Weird Trick to Reduce Your Code Size

`dataclasses` was introduced in Python3.8. You can also look at the (official documentation)[https://docs.python.org/3/library/dataclasses.html]. dataclasses decorator gets you the getters and setters and the `__repr__` methods for free. In addition, it also generates equality and comparison operations in case you need to order them. And finally, it also generates the `__hash__` method so that we can also use them in case we need to store them in kv datastructures.

## How to use it?

#### Let us define our class this way --

```
import dataclasses

@dataclasses.dataclass()
class MyAmazingClass:
  arg_A: int
  arg_B: float
  arg_C: str = "default_name"
```

#### Different Ways of Creating an Object

```
# define an instance of the class
mac1 = MyAmazingClass(1, 1.1, "mac1_name")

# take advantage of the default arguments
mac2 = MyAmazingClass(2, 2.2)
assert mac2.arg_C == "default_name"
```

#### Instantiation using Dictionaries

```
myd3 = {"arg_A": 3, "arg_B": 3.3, "arg_C": "instantiaed_from_myd3"}
mac3 = MyAmazingClass(**myd3)
assert mac3.arg_C == "instantiaed_from_myd3"
```
#### Instantiating from Another Object

```
myd4 = dataclasses.asdict(mac2)
assert myd4["arg_A"] == 2
myd4["arg_C"] = "jumped_a_few_hoops"
mac4 = MyAmazingClass(myd4)
assert mac4.arg_B == 2.2
assert mac4.arg_C == "jumped_a_few_hoops"
```

## Something Similar in C++

### Designated Initializers in C++20

Please read this documentation on `designated initializers` in C++20. And let us create something similar in C++ as well.

Let us define our struct like this -

```
struct MyAmazingStruct
{
  int arg_A;
  float arg_B;
  std::string arg_C = "default_name";
};
```

The simplest way of using it is -

```
MyAmazingSturct mas1{.arg_A = 1, .arg_B = 1.1};
MyAmazingStruct meh(1, 1.1);
```

Note that parameters have to be supplied in the same order as specified in the struct. Look at the two definitions for the instances. It is clear that `mas1` is more descriptive than `meh`. It is descriptive but we can also skip the names using elison

```
// skip everything
MyAmazingStruct mas2 = {2, 2.2, "name_is_mas2"};
```

#### What if the struct is nested?

If our struct is defined this way --

```
struct MyNestedStruct
{
  int arg1;
  struct ChildStruct1
  {
    float carg1;
    float carg2;
  } arg2;
  string arg3;
};
```

We can instantiate our objects in these ways --

```
MyNestedStruct mns1 = {1, {1.414, 1.732}, "mns1_sq_roots"}; // nothing is defined but they are nested
MyNestedStruct mns2 = {2, 1.414, 1.732, "mns2_sq_roots"}; // nothing is defined and they are all flat!
MyNestedStruct mns3{.arg1 = 3, arg2 = {1.414, 1.732}, arg3 = "mns3_sq_roots"}; // a mix of both
```

## Conclusion

`dataclasses` are decorators and need to be added in the python code above the class definition to use them. It's not a standard python feature. They help us get rid of writing boilerplate code. One can take it a step further and define them using dictionaries. Similarly, we can get the dictionary representatio of the class. I can already imagine this to be very useful when parsing json files.

`designated initializers` is part of C++20 and you'll have to pass the appropriate args in your compiler to enable C++20 features. Their usage is like a dial. You can get extremely descriptive code when turned all the way to the left. Turn the dial all the way to the right and you can define instances using a very flat (albeit confusing) set of values inside braces.

Thanks for reading this post!
