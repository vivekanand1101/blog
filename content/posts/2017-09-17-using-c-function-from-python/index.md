---
title: "Using C function from Python"
date: 2017-09-17
categories: 
  - "tech"
---

c_types_ is a python library which allows using C data types, functions from a python script. It's in the standard python library. To use C functions using ctypes, you will need to compile the C code and create a shared library.

**add.c**

\[code language="cpp"\]

#include <stdio.h>

int add\_two\_numbers(int num1, int num2) { return num1 + num2; }

\[/code\]

I will be using a very simple C function in this case which adds two given numbers

Now compile this file using: _gcc -fPIC -shared -o libadd2nums.so add.c_

This will create a shared library named libadd2nums.so which, for now, contains only one function.

**add.py**

\[code language="python"\]

\# coding=utf-8

import ctypes

\_add = ctypes.CDLL('/home/vivek/ctypestuts/libadd2nums.so') \_add.add\_two\_numbers.argtypes = (ctypes.c\_int, ctypes.c\_int)

def add\_two\_numbers(num1, num2): ''' Adds two numbers '''

return \_add.add\_two\_numbers(ctypes.c\_int(num1), ctypes.c\_int(num2))

\[/code\]

I am using fedora 26. If you are using Windows, you will need to use _ctypes.WinDLL._

_\_add_ here, is the shared library and we can access the C function using dot(.) .

**main.py**

\[code language="python"\]

\# coding=utf-8

import add

num1 = int(raw\_input("Enter num1: ")) num2 = int(raw\_input("Enter num2: ")) print add.add\_two\_numbers(num1, num2)

\[/code\]

That's it.
