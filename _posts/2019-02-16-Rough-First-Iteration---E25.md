---
layout: post
title:  "Rough First Iteration - E25"
date:   2019-02-16 19:53:30 -0500
categories: E
---
[Euler problem 25][E25] from [ProjectEuler.net][elink] was a problem I randomly selected from the list. Some of the further problems seemed very complex/more involved than an hour or two free time project.

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
So we have three 30-digit arrays, of integers, and as long as we keep the value at each index below 10, the 20 digit limit in C is "bypassed". This creates another set of issues.
* Efficiency of adding each individual element of each array
    - Carrying when p1[index] + p2[index] > 10
    - Implementing the carry (recursion makes this easier in theory)
* Nested looping vs. recursion
    - I dislike recursion
* Implementing length/digits of Fibonacci number desired


At first, I implemented an iterative attempt, and avoided recursion. I was immediately led to reversing the array. One of my first attempts calculated the correct number, until double digits. Then it assigned the numbers backwards, until it stopped working again.
i.e.
```
0001
0001
0002
0003
0005
0008
0031
0012
```
The first look I took was brain bending to me. "That doesn't make any sense...", so I reversed the arrays, and started indexing at [DIGITS - 1], and changed my print function to work forwards.

```c
    for (int i = 0; i < DIGITS; i++) //initialize arrays (just in case)
    {
        digits[i] = 0;
        p1[i] = 0;
        p2[i] = 0;
    }
    p1[DIGITS - 1] = 1; //first value
    p2[DIGITS - 1] = 1; //second value
```
```c
void printArray(int *initialArray, int i) //prints 'i' (DIGITS) digits of array
{
    printf("Array: ");
    for(int k = 0; k < i ; k++)
    {
        printf("%d", initialArray[k]);
    }
    printf("\n");
}
```
Next step was to implement the carry logic. I got this idea initially from my Digital Systems Design course - to add bits together, you implement the following table:

| A | B | C | S |
| :-: | :-: | :-: | :-: |
| 0 | 0 | 0 | 0 | 
| 0 | 1 | 0 | 1 | 
| 1 | 0 | 0 | 1 | 
| 1 | 1 | 1 | 0 |
{:.mbtablestyle}

Where C and S are the carry bit, and sum bit, respectively. Only when our two inputs (A and B) are 1, is there a carry. Applying this to our base-10 system, we would carry only when the sum of our inputs (the two indexes we are adding) are greater than 10. Consider adding the numbers 8 and 13

| |  [1] | [0] | Carry |
| :-: | :-: | :-: | :-: |
| **p1** | 0 | 8 | 0 |
| **p2** | 1 | 3 | 0 |
|Sum of Col.|  1 | 1 | **1** |
| :-: | :-: | :-: | :-: |
{:.mbtablestyle}
We need to get that carry to p2[1], because 8 + 13 is NOT 11. The good news was I knew exactly where the issue was.


|*Note about recursion:*
A recursive solution seems fairly straightforward. Just call the function, increment the index from p1[0] to p1[1], for example, and then add the carry. I tried implementing this, struggled, and decided to go back to an iterative method. The recursive method worked, but I was hung up on the logic of the carry, and recursion made it more complicated. In addition, I looked into it, and recursion is not necessarily as efficient as iteration, despite being more simple to program, in general. This is due to ["allocation of a new stack frame"][rec vs it].|

{:.mbtablestyle}

The general idea is that we _only_ want to mess with the carry if we need to.
```c
if (p1[i] + p2[i] + carry < 10)
{
    temp2 = p1[i] + p2[i] + carry;
    digits[f] = temp2;
    temp2 = 0;
    carry = 0;
}
```
The first if condition says if the number at index i of p1, p2, and the carry sum to _less than_ 10, we can outright set the value of that index for our digits array. In other words, if we don't have a carry that makes our sum greater than 10, we don't set another carry!
```c
carry = p1[i] + p2[i] + carry;
digits[f] = (carry % 10);
carry = carry / 10;  
```
The else condition is when our carry, as well as our two digits we are adding is greater than 9. Consider a carry of 2, p1[1] of 7 and p2[1] of 9. Our sum (just uses carry variable here) is 2 + 7 + 9 = 18. We assign digits[f] to be 8, using the modulus operator, and our carry becomes the next 1 (value of 10). On the next loop, the carry will be 1, so when adding, it will be taken into account for p1[2] and p2[2], and so on.

```c
for (int k = 0; k < 250000; k++) //fib(250,000)
    {
        //...
        memcpy(p2, p1, DIGITS * sizeof(int)); //p2 becomes p1
        memcpy(p1, digits, DIGITS * sizeof(int)); //p1 becomes digits
        //printArray(digits, DIGITS);
    }
    return 0;
}
```

Each iteration of k copies the arrays according to the Fibonacci sequence. Memcpy copies the block of memory at pointer p1 of DIGITS * sizeof(int) long, assigns it to the location of the pointer p2. The data at each index is an integer, so the length of the arrays requires a multiplication of the size of an integer, to ensure a contiguous block of memory. Probably best we don't lop off half of our hard work by copying the array incorrectly!

Putting everything together, we have a relatively efficient method for calculating almost any number of the Fibonacci sequence, given enough time. On my machine, fib(250,000) takes ~30 seconds.

```c
//Have you ever had gnocchi? It's delicious.
int fibbognocchi(int *digits, int *p1, int *p2) 
{
    int i = 0; //index for iteration through p1 and p2 DIGITS - 1 times
    int f = DIGITS - 1; //index for digits (for clarity)
    int carry = 0;
    int temp2 = 0;
    for (int k = 0; k < 250000; k++) //fib(250,000)
    {
        f = DIGITS - 1; //
        for(i = DIGITS - 1; i >= 0; i--)
        {
            if (p1[i] + p2[i] + carry < 10)
            {
                temp2 = p1[i] + p2[i] + carry;
                digits[f] = temp2;
                temp2 = 0;
                carry = 0;
            }
            else
            {
                carry = p1[i] + p2[i] + carry;
                digits[f] = (carry % 10);
                carry = carry / 10;                   
            }
            f--;
        }
        //Fibonacci algorithm logic
        memcpy(p2, p1, DIGITS * sizeof(int)); //p2 becomes p1
        memcpy(p1, digits, DIGITS * sizeof(int)); //p1 becomes digits
        //printArray(digits, DIGITS);
    }
    printArray(digits, DIGITS); //prints final array (fib(250,000))
    return 0;
}
```
I added a timing feature, to measure how long the program takes to run, as well as a percentage complete printout:

```c
#include <time.h> //defining cpu clock-time library
    #if defined(HAVE_SYS_TIME_H) || defined(WOLF_C99)
        #include <sys/time.h>
    #endif
    //...
```
```c
clock_t start = clock();
clock_t end;
double time_elapsed_in_seconds;
//...calculation code
end = clock();
//math for time taken to calculate
time_elapsed_in_seconds = (end - start)/(double)CLOCKS_PER_SEC;
printf("Seconds elapsed from start of program: %.2f", time_elapsed_in_seconds);
```
```c
int fibNum = 250000; //fib(x)
int fibDiv = fibNum / 100; //used for % printing
//...
if(k == divTemp) //gives % completion inside loop
{
    divTemp = fibDiv + divTemp;
    count++;
    printf("%d%%\n", count);
}
```
Further notes/future plans:
* Configuring code/variables for user input (custom lengths or fibNum) could prove useful
* Optimizing code as a means for observing efficiency
* Self-generating executable
* Writing to output (i.e. writes 100,000 Fibonacci numbers to a text file)

[2000 Fibonacci values.][fib2000]


[fib2000]: /files/fib(2000).txt
[rec vs it]: https://stackoverflow.com/questions/2651112/is-recursion-ever-faster-than-looping
[E25]: https://projecteuler.net/problem=25
[elink]: https://projecteuler.net/about
