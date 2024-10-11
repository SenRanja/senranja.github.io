---
title: Sieve of Eratosthenes
categories:
  - Math
  - Algorithm
date: 2024-10-11 10:21:05
tags:
mathjax: true
---

- [Sieve of Eratosthenes](#sieve-of-eratosthenes)
- [The algorithm](#the-algorithm)
  - [step 1](#step-1)
  - [step 2](#step-2)
  - [step 3](#step-3)
- [Confusion](#confusion)
  - [why external loop's range is range(2, floor(sqrt(n)) + 1)](#why-external-loops-range-is-range2-floorsqrtn--1)
  - [why internal loop from range(p \* p, n + 1, p)](#why-internal-loop-from-rangep--p-n--1-p)
  - [Why Sieve of Eratosthenes could find primes](#why-sieve-of-eratosthenes-could-find-primes)


# Sieve of Eratosthenes

At the first to know Sieve of [Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes), it was a snippet in a quiz of python project in UNSW.

The sieve of Eratosthenes is an ancient algorithm for finding all prime numbers up to any given limit.

The code below would generate a list in the range of [2, n](n included). It is very fast and convenient, with just a little memory and time-consumption.

```python
def sieve_of_primes_up_to(n):
    sieve = [True] * (n + 1)
    # Note, the actual range is inclusive [2, floor(sqrt(n))]
    # the "floor(sqrt(n)) + 1)" is just statisfy python's list range.
    for p in range(2, floor(sqrt(n)) + 1):
        if sieve[p]:
            for i in range(p * p, n + 1, p):
                sieve[i] = False
    sieve[0] = False
    sieve[1] = False
    sieve_primes_list = [i for i in range(n + 1) if sieve[i]]
    return sieve_primes_list
```

# The algorithm

## step 1

Firstly, set all numbers are True:

[2:T, 3:T, 4:T, 5:T, ... , n]

## step 2

Secondly, from 2 to the $\lfloor \sqrt{n} \rfloor$, set the multiples of the least prime as the Composite Number from its perfect number.

E.g. now the existed list is 

[2:T, 3:T, 4:T, 5:T, ... , n]

Then set the current number is 2, set its perfect number (namely 2*2==4), then set 2*3==6, 2*4==8, 2*5==10 ... In conclusion, set 4th, 6th, 8th, 10th, ...,  as composite numbers (namely False status) step by step.

After finishing 2, then find next number, namely 3, hence set 3*3, 3*4, 3*5, ... . set 9th, 12th, 15th, ... numbers as false.

Until the number: $\lfloor \sqrt{n} \rfloor$ [Namely py code expression: floor(sqrt(n))]

## step 3

Thirdly, return the list with boolean values.

# Confusion 

## why external loop's range is range(2, floor(sqrt(n)) + 1)

And the inclusive range `[2, floor(sqrt(n))]` is the actual range.

For the least limit, cause 2 is the least prime number.

For the largest limit, the "range(floor(sqrt(n)) + 1)" is just statisfy python's list range, remeber the last limit number should add 1.

The original code I firstly saw is `for p in range(2, round(sqrt(n)) + 1):`, I think round() is used redundantly.

Because the internal loop **i** starts with $p \times p$, which means the largest number in the internal loop would be $\sqrt{n}$. And $i^2$ shouldn't be beyond the n, that is meaningless. 

For example, if n==5, and $\sqrt{5} \approx 2.24$, we can compute $2 \times 2 = 4$, because it is under 5, and computing $3 \times 3 = 9$ is pointless because it's beyond the n (namely 5).

Obviously, the difference between $n^2$ and $(n+1)^2$ is much larger than 1.

Hence, we'd better use `floor(sqrt(n))` as the largest number of the external range, rather not `round(sqrt(n))`. Round is rounding, which may make one unnecessary number comsumption.

I ran a program to distinguish the nuance of it:

```python
def sieve_of_primes_up_to(n):
    sieve = [True] * (n + 1)

    try:
        # Note Here!
        for p in range(2, round(sqrt(n)) + 1):
        # for p in range(2, floor(sqrt(n)) + 1):
            if sieve[p]:
                for i in range(p * p, n + 1, p):
                    sieve[i] = False
    finally:
        # Note Here!
        print(p)
    sieve[0] = False
    sieve[1] = False
    sieve_primes_list = [i for i in range(n + 1) if sieve[i]]
    return sieve_primes_list
```

When the test cases are [8, 11, 20, 4321, 98_234, 980_234, 50_000_000]

the different outputs are:

```
2
3
4
65
313
990

------

3
3
4
66
313
990
```

PS: Oh, I think the proof is not much useful ... The bound of the parameter wont induce real redundant computing, because it is too simple just like the code below won't run redundantly:

```python
for i in range(4,2,1):
 print()
```

## why internal loop from range(p * p, n + 1, p)

At the first, I was confused by the p * p as the start. Then I realized that the number under p had computed with $p$ before the new $i$ of $p \times p$.

Let's make a diagram.

Let n=20:

The actual prime list is:

[2,3,5,7,11,13,17,19]

We follow the program to compute from 2 to $\sqrt{20} \approx 4.47$ whose floor number is 4.

2: [4,6,8,10,12,14,16,18,20]

3: [9,12,15,18]

4: [16,20]

Ok, if we assume $i$ doesn't start with $p \times p$, $i$ just is from multipling 2,3,4,... .

For example, $i = 4$, we need to compute 4*2, we found $4 \times 2$ has been computed at the step of $i = 4$.

So starting with $p \times p$ is a way to reduce redundancy.


## Why Sieve of Eratosthenes could find primes

Just follow the **definition** of **Prime Number** on Wiki.

  A prime number (or a prime) is a natural number greater than 1 that is not a product of two smaller natural numbers.

We would see **Sieve of Eratosthenes** just followed definition to find primes. It finds every products of numbers that won't become primes, hence the numbers left are just primes.
