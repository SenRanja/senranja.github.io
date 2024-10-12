---
title: Boolean logic
categories:
  - Math
date: 2024-10-12 13:01:52
tags:
mathjax: true
---

# What is logic

Logic is about **formalizing reasoning** and **defining truth**
- Adding rigour
- Removing ambiguity
- Mechanizing the process of reasoning

## Loose history of logic

(Ancient times): Logic exclusive to philosophy
Mid-19th Century: Logical foundations of Mathematics (Boole, Jevons, Schr¨oder, etc)
1910: Russell and Whitehead’s Principia Mathematica
1928: Hilbert proposes Entscheidungsproblem
1931: G¨odel’s Incompleteness Theorem
1935: Church’s Lambda calculus
1936: Turing’s Machine-based approach
1930s: Shannon develops Circuit logic
1960s: Formal verification; Relational databases

## Logic in Computer Science

Computation = Calculation + Symbolic manipulation

Logic as 2-valued computation (Boolean logic):
- Circuit design
- Code optimization
- Boolean algebra
- Nand game

Logic as symbolic reasoning (Propositional logic, and beyond):

- Formal verification
- Proof assistance
- Knowledge Representation and Reasoning
- Automated reasoning
- Databases


# Boolean Logic

Boolean logic is about performing calculations in a “simple” mathematical structure.
- complex calculations can be built entirely from these simple ones
- can help identify simplifications that improve performance at the circuit level
- can help identify simplifications that improve presentation at the programming level

## The Boolean Algebra $\mathbb{B}$

Definition

The (two-element) **Boolean algebra** is defined to be the set $\mathbb{B}$ ={0,1}, together with the functions 

$$
!: \mathbb{B} \to \mathbb{B}, \&\&: \mathbb{B}^2 → \mathbb{B}, \quad and \quad ||: \mathbb{B}^2 → \mathbb{B}
$$

, defined as follows:

$$
!x = (1−x) \\
x \&\&y =min\{x,y\} \\
x ∥ y =max\{x,y\}
$$

## Alternative notation

For $\mathbb{B}$

$$
\mathbb{B} = \{ \text{false}, \text{true} \} \quad \text{or} \quad \mathbb{B} = \{ F, T \} \quad \text{or} \quad \mathbb{B} = \{ 0, 1 \} \quad \text{or} \quad \mathbb{B} = \{ \bot, \top \}
$$

For $!x$

$$
\overline{x} \quad !x \quad \text{or} \quad x' \quad \text{or} \quad \sim x \quad \text{or} \quad \neg x \quad \text{or} \quad \text{NOT}(x)
$$

For $x \&\& y$

$$
x \land y \quad \text{or} \quad xy \quad \text{or} \quad x \wedge y \quad \text{or} \quad (x \, \text{AND} \, y)
$$

For $x || y$

$$
x \lor y \quad \text{or} \quad x + y \quad \text{or} \quad (x \, \text{OR} \, y)
$$

## Alternative notation

Definition

The (two-element) **Boolean algebra** is defined to be the set $\mathbb{B}$ ={false,true}, together with the functions  $! :\mathbb{B} \to \mathbb{B}, \&\&: B^2 →B, and \quad ∥: B^2 →B$, defined as follows:

| x | !x |
| ----- | ----- |
| false | true |
| true | false |

---

| x | y | x&&y |
| ----- | ----- | ----- |
| false | false | false |
| false | true | false |
| true | false | false |
| true | true | true |

---

| x | y | x or y |
| ----- | ----- | ----- |
| false | false | false |
| false | true | true |
| true | false | true |
| true | true | true |

## Properties:Commutativity,Associativity,Distribution,Identity,Complementation

We observe that !, &&, and ∥ satisfy the following:

For all $x,y,z ∈ \mathbb{B}$:

Commutativity

$$
x ∥ y =y ∥x \\
x \&\&y =y \&\& x
$$

Associativity

$$
(x ∥ y) ∥ z =x ∥(y ∥z) \\
(x \&\& y) \&\&z =x \&\&(y \&\& z)
$$

Distribution

$$
x ∥ (y \&\&z) =(x ∥y)\&\&(x ∥ z) \\
x \&\&(y ∥z) =(x \&\& y)∥(x \&\& z)
$$

Identity

$$
x ∥ 0=x \\
x \&\& 1=x
$$

Complementation

$$
x ∥ (!x) = 1 \\
x \&\&(!x) = 0
$$

# Boolean Functions

Definition

An n-**ary Boolean function** is a map f : $\mathbb{B}^n \to \mathbb{B}$.

Q: How many unary Boolean functions are there? How many binary functions? N-ary?

$\mathbb{B}^2 \to \mathbb{B}$, we found there are 4 possible inputs and **2 possible outputs of each input**, so here is $2*2*2*2=16$ **possible outputs**. 

**Note, the question didnot mention the specific function, so don't think 0,0->0 directly, cause maybe 0,0->1 function. Additionally, the different outputs should multiple as possible results. Just regard different combination of the outputs as a new function.**

![](2024-10-12-13-56-57.png)

4 means how many input here.

How many unary Boolean functions are there? n-ary?

unary means One-dimensional relation. So how many, it's 2+2=4 obviously.

![](2024-10-12-13-57-23.png)

$$
2^{2^{2}}=4
$$

So n-ary is:

$$
2^{2^{n}}=
$$

## example

$!$ is a unary Boolean function

$\&\&, ∥$ are binary Boolean functions

$f(x,y) =!(x \&\& y)$ is a binary boolean function (NAND)

$And(x_0,x_1,...) = (···((x_0 \&\& x_1) \&\& x_2)···)$ is a (family) of Boolean functions

$Or(x_0,x_1,...) = (···((x_0 ∥ x_1) ∥ x_2)···)$ is a (family) of Boolean functions

## Application: Adding two one-bit numbers

**Question**. How can we implement:

$add: \mathbb{B}^2 \to \mathbb{B}^2$

defined as

| x | y | add(x,y) |
| ----- | ----- | ----- |
| 0 | 0 | 00 |
| 0 | 1 | 01 |
| 1 | 0 | 01 |
| 1 | 1 | 10 |


Take Notice

Observe that this is **not** a Boolean function. Boolean functions can output either 0 or 1 (a single bit). This function outputs 2 bits.

**(Short) Answer**. Use two Boolean functions!

Take Notice

Digital circuits are just sequences of Boolean functions.


# Conjunctive and Disjunctive Normal Form

Definition

A **literal** is a unary Boolean function

A **minterm** is a Boolean function of the form $And(l_1(x_1),l_2(x_2),...,l_n(x_n))$ where the $l_i$ are **literals**

A **maxterm** is a Boolean function of the form $Or(l_1(x_1),l_2(x_2),...,l_n(x_n))$ where the $l_i$ are **literals**

A **CNF Boolean function** is a function of the form $And(m_1,m_2, ...)$, where the $m_i$ are **maxterms**.

A **DNF Boolean function** is a function of the form $Or(m_1,m_2, ...)$, where the $m_i$ are **minterms**.

![](2024-10-12-14-45-16.png)

Take Notice

`CNF`: **product** of **sums**; `DNF`: **sum** of **products**

秘笈：“C与D或”,“C与小D或大”，“min AND, max OR, C与积，D或和”

## Examples

Question. Are these functions in CNF? Are they in DNF?

$$
f(x,y,z) = (x \&\& (!y) \&\& z) ∥ (x \&\& (!y) \&\& (!z)) = x \overline{y} z +x \overline{y} \overline{z} 
$$
DNF , but not CNF

$$
g(x,y,z) = (x ∥ (!y) ∥ z) \&\& (x ∥ (!y) ∥ (!z)) = (x +\overline{y} +z)(x +\overline{y} +\overline{z} )
$$
CNF function, but not DNF

$$
h(x,y,z) = (x \&\& (!y) \&\& z) = x \overline{y} z
$$
**both CNF and DNF**

Note! **Paul Hunter** (lecturor):

```
Note: CNF/DNF/minterms/maxterms are only concerned with  how the formula appears as written.  You cannot "change" the formula (e.g. "x can be regarded as x or x") and have things be meaningful.

(x && !y) && z is:

A maxterm

An OR of maxterms: because OR(m) = m, so it is equal to OR((x&&!y)&&z) - therefore it is in DNF

An AND of minterms: because x is a minterm, !y is a minterm, and z is a minterm, so it is equal to AND(x, !y, z) - therefore it is in CNF

Leo Liu:
I'm still confused, why x, !y and z are miniterms? Because x = AND(x)? So x can be considered as literal (x), miniterm (AND(x)), and maxterm (OR(x))? 

Paul Hunter:
Yes, that's correct.

AND and OR are "generalizations" of && and ||.

&& and || are functions that take two inputs, AND and OR are (families of) functions that take arbitrarily many inputs.
```

$$
j(x, y,z) = x +y(z +x)
$$
Neither CNF nor DNF

## 2 Theorems

Every Boolean function can be written as a function in **DNF**.

Every Boolean function can be written as a function in **CNF**.










# Karnaugh Maps








# Boolean Algebras








