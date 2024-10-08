---
title: vacuous_truth
categories:
  - Math
date: 2024-09-27 23:03:00
tags:
mathjax: true
---


# 由空集和子集，想到空洞真理

最开始是因为我上Paul的lecture，他在讲Relation的数学知识，问到“Could one have a releation of itself?”，我在台下想到递归的概念，以及空集的概念，遂答“Recursion! For example, an empty set is a subset of the empty set itself.”当时Paul下台和我聊了一下，可能一下子没有反应过来“空集是不是空集的子集”的概念，遂作罢。

我当时突然也在问我自己，“空集是空集自己的子集”是true还是false。一开始使用ChatGPT查了一下，我分别用English和Chinese查这个statement是true还是false，但是分别得到了“True”和“False”的回答，我疑惑，发Ed Forum上寻tutors帮助。

此处我把关键部分以text形式在此处简述或复制：

“空集是空集的子集”不好判断的原因在于：

1. 子集的定义是，所有元素也在另一个集合中出现；
2. 空集没有元素。

因此，wikipedia上对此statement的描述：

The empty set, written ∅ or {} has no elements, and therefore is vacuously a subset of any set X.

注意，这里的副词**vacuously**(空洞地)，有hyperlink指向“**vacuous truth**”（即数学概念“空洞的真理”）。

我自己的理解与论述（推荐英文不好的同学直接机翻一下，此处“空洞真理”概念很有趣！）：


```
Hi tutor Liwenqing Liu,
Yeah, what you said is right, it is the only math formula, it can give the number of {}'s subsets, but it cannot prove {} is the subset of {}. It is a B->A causal relation rather not A->B relation.
I followed Jiapeng Wang's last reply to understand the vacuous truth, and I sorted out the following steps here.

1. {} has no elements
2. According to the definition of subset
I cannot prove {} is the subset of {}. Because {} has no element, and {} has no element either. We cannot say that every element of {} belongs to {}.

3. there is a paragraph of Vacuous Truth from Wikipedia
[Vacuous truth](https://en.wikipedia.org/wiki/Vacuous_truth)

It is sometimes said that a statement is vacuously true because it does not really say anything.[2] For example, the statement "all cell phones in the room are turned off" will be true when no cell phones are present in the room. In this case, the statement "all cell phones in the room are turned on" would also be vacuously true, as would the conjunction of the two: "all cell phones in the room are turned on and turned off", which would otherwise be incoherent and false.

I think:
{} has the same element of {} ----- Truth

3-1. because {} has no element (Empty set's definition)
3-2. neither has no elements => Both two statements "they have the same elements" and "they have no the same elements" are right. The two statements are both right, but it will be false if combinate them into one statement "they have no the same elements and have the same elements".
So according to Vacuous Truth, it is true that "they have no the same elements" of {} and {}.
One more proverb (just for my understanding), if I say "My cat can do backflips at home" to my friend, and then he/she would have a look with me (True). But a normal person knows such a cat who can do backflips doesn't exist.

4. Ok, I don't how to convince myself of the Vacuous Truth honestly, I am trying to cheat myself. Thank you for seeing this step. 
I don't know how to prove "{} is the subset of {}", it is too hard for me. T_T . I don't struggle anymore.
```

与tutors讨论详情始末详见如图所示。

![COMP9020-–-Ed-Discussion](COMP9020-–-Ed-Discussion.png)

其实想想空洞真理：“空洞”表示你无法使用严密逻辑去证明；“真理”表示他是真的，你甚至无法否定（待会儿我再举例子）。在数学领域，空洞真理常可以用于探索模糊的边界问题，来保证数学逻辑的连贯和计算闭包，比如0的概念。

古代人们数数，都是从1开始作为第一个序数或基数词，没有0的概念。我说我有iphone，只不过数量为0，这句话数学层面是true而非false（注意区分“直觉上认为我没有iphone”和数学性质上“我有iphone”的区别）。人们做加法、减法没有负数的概念，比如B欠A十块钱，A只会说“B，你欠我10快钱，今天先还我3块钱，这样你就只欠我7快钱了。”，而不会想“A应该支付B -10块钱”使用负数来表示。我们现在已经默认了0、正负数计算的概念，甚至已经将其当作了直觉，因此发现直接正负数计算其实很方便算账。如ED Forum的tutors的指正，0的存在，保证了正负数计算的闭包操作。

空洞真理常常适合纯数学领域确认一些概念的极限、模糊问题，放在物理、生物、化学领域反而大概率不会被承认，大家其实也经常听闻数学家和物理学家在计算方面是存在一些差异和分歧的，数学家大概率习惯在边界问题追求公式的闭包计算，物理学家通常不会这么做。

其实在日常生活中，我们也经常相信空洞真理，无法证实但是可以是true的statement，举例：

1. 无法证明历史是否存在，但是你会认为历史人物和故事是真实存在（即true），但是你本身不活在过去的历史，因此你无法证实历史的真实性；
2. 你和朋友说“我家猫会后空翻”，不论你家真的没猫，你的朋友会相信；
3. 我有车，只不过目前车的数量是0；
4. 房间里没手机，但是你可以和领导报告“房间中的全部手机已关机”。