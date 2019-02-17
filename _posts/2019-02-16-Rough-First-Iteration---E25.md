---
layout: post
title:  "Rough First Iteration - E25"
date:   2019-02-16 19:53:30 -0500
categories: E
---
[Euler problem 25][E25] from [ProjectEuler.net][elink] was a problem I randomly selected from the list. Some of the further problems seemed very complex/more involved than a 10-minute free time project.

The problem is as follows:
>
>The Fibonacci sequence is defined by the recurrence relation:
>
>Fn = Fn−1 + Fn−2, where F1 = 1 and F2 = 1.
>Hence the first 12 terms will be:
>
>F1 = 1
>
>F2 = 1
>
>F3 = 2
>
>F4 = 3
>
>F5 = 5
>
>F6 = 8
>
>F7 = 13
>
>F8 = 21
>
>F9 = 34
>
>F10 = 55
>
>F11 = 89
>
>F12 = 144
>
>The 12th term, F12, is the first term to contain three digits.
>
>What is the index of the first term in the Fibonacci sequence to contain 1000 digits?

1000 digits is a lot. Standard C libraries only allow 20 digits (18,446,744,073,709,551,615 -  max unsigned long int). I've never worked with numbers larger than 2.147e9 - there's usually no need. After some brief thought (maybe trying a different language, different data type, etc) I tried using an array, statically.
```c
#define a1 1
#define a2 1
#define DIGITS 30
//...
int digits[DIGITS]; //sum array
int p1[DIGITS]; //n - 1 array
int p2[DIGITS]; //n - 2 array
```
So we have three 30-digit arrays, of integers, and as long as we keep each index below 10, the 20 digit limit in C is "bypassed". This creates another set of issues.
* Efficiency of adding each individual element of each array
    - Carrying when p1[index] + p2[index] > 10
    - Implementing the carry (recursion makes this easier in theory)
* Nested looping vs. recursion
    - I dislike recursion
* Implementing length of Fibonacci number desired



[E25]: https://projecteuler.net/problem=25
[elink]: https://projecteuler.net/about
