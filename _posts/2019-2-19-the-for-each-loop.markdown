---
layout : post
title : "谈谈 foreach 内部实现"
categories : Java
author : ABei
---

* content
{:toc}

说到遍历数组或者集合，估计我们都会想到使用下面这种遍历方式：

```java
int[] integerArr = [1, 2, ,3, 4];

for(int i : integerArr) {
    // do something...
}
```

但是，Java 内部是如何实现的？




先来看下，For-Each 出现之前，我们是怎么遍历集合的。
对集合进行迭代比实际需要的要糟糕得多。考虑下面这个方法，`cancelAll` 方法接受一组计时器任务并取消它们:

```java
void cancelAll(Collection<TimerTask> c) {
    for (Iterator<TimerTask> i = c.iterator(); i.hasNext(); )
        i.next().cancel();
}
```

这样写功能上没什么问题，但是看起来很杂乱。此外，迭代器变量 `i` 在每次循环中出现 3 次。这种写法可能会增加代码出错的几率。For-Each 这种构造能够消除前面写法带来的弊端。下面来看一下，For-Each 写法：

```java
void cancelAll(Collection<TimerTask> c) {
    for (TimerTask t : c)
        t.cancel();
}
```

For-Each 中的冒号 `(:)` 可以理解为 `in`。上面的循环可以读作 “for each TimerTask t in c.”。如你所见，For-Each 结构与泛型完美结合，与此同时，它保留了集合中所有元素类型的安全性，同时消除了代码的混乱。因为你不需要声明迭代器，并且不用为迭代器提供泛型声明（编译器在背后会为你做这些，但你不需要关心它）。

下面是人们在尝试对两个集合进行嵌套迭代时，经常犯的一个错误：

```java
List suits = ...;
List ranks = ...;
List sortedDeck = new ArrayList();

// BROKEN - throws NoSuchElementException!
for (Iterator i = suits.iterator(); i.hasNext(); )
    for (Iterator j = ranks.iterator(); j.hasNext(); )
        sortedDeck.add(new Card(i.next(), j.next()));
```

你能发现 bug 吗？如果你做不到，不要难过。许多专业程序员都曾犯过这样或那样的错误。问题是：`i.next()` 这个方法在 `ranks.iterator();` 的迭代器中被调用了太多次了。

知道了问题所在，解决起来就很简单了。

```java
// Fixed, though a bit ugly
for (Iterator i = suits.iterator(); i.hasNext(); ) {
    Suit suit = (Suit) i.next();
    for (Iterator j = ranks.iterator(); j.hasNext(); )
        sortedDeck.add(new Card(suit, j.next()));
}
```

那么所有这些与 For-Each 结构有什么关系呢？它是为嵌套迭代量身定做的！

上面的写法可以用 For-Each 简洁的写出：

```java
for (Suit suit : suits)
    for (Rank rank : ranks)
        sortedDeck.add(new Card(suit, rank));
```

For-Each 构造也适用于数组，它隐藏 **索引** 变量而不是 **迭代器**。下面的方法返回 int 数组中值的和:

```java
int sum(int[] a) {
    int result = 0;
    for (int i : a)
        result += i;
    return result;
}
```

那么什么时候应该使用 for-each 循环呢？任何时候都可以。它确实美化了代码。不过，它并不是适应所用的是用场景。请看下面的代码：

```java
// Removes the 4-letter words from c
static void expurgate(Collection<String> c) {
    for (Iterator<String> i = c.iterator(); i.hasNext(); )
      if (i.next().length() == 4)
        i.remove();
}
```

为了删除当前元素，程序需要访问迭代器。for-each 循环隐藏了迭代器，因此不能调用 `remove` 方法。因此，for-each 循环不能用于过滤。

类似地，对于需要在遍历列表或数组时替换其中元素的循环，它也不适用。