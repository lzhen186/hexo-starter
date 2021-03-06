---
title: Java Java SE 20.集合前言
date: 2022-07-04T10:11:47.146Z
tags: [javase]
---
# 1.为什么需要Java集合

为了方便存储操作多个对象，虽然对象数组也可以存储操作多个对象，但是数组的长度是不可变的。

# 2.数组和集合的区别

* 长度的区别
  * 数组的长度是固定的
  * 集合的长度是不固定
* 元素的数据类型
  * 数组可以存储基本数据类型，也可以存储引用数据类型
  * 集合只能存储引用数据类型(存储的int类型，它会自动装箱为Integer)

# 3.如何入门学习Java集合

Java集合的学习应该从**Java集合基本用法(各种API的学习)**和**以面向对象的角度去理解Java集合**这两个方面入手。

**Java集合的基本用法**就是学习集合中每个API的用法及具体的效果。

**以面向对象的角度去理解Java集合**：当你对Java集合的API使用有一定的了解之后，就应该以面向对象的角度去理解Java集合，其实在学习Java其他知识时也应该以面向对象的角度去思考为什么这么设计。那么**以面向对象的角度去理解Java集合**要思考哪些问题呢？我思考的是为什么会抽象出多个接口，以及每个接口有什么特性。这也是为什么要先去学习Java集合的各种API，这样你才有可能回答这些问题。当然不是你学好了各种API后就能回答这些问题的，我们还学要学习一些源码，在学习源码的过程中就会用到数据结构的知识了，当你了解了它的实现使用了那种数据结构以及你了解这一数据结构的特性的时候，相信你对它为什么这么设计会有自己的认识了。那么我们学习Java集合时要有哪些数据结构的知识呢？需要学习的 数据结构有：数组、链表、散列表、红黑树。当你学习了解各个实现类的数据结构之后，相信你会有以下的结论：

* 如果是集合类型，有List和Set供我们选择。List的特点是插⼊有序的，元素是可重复的。Set的特点是插⼊⽆序的，元素不可重复的。⾄于选择哪个实现类来作为我们的存储容器，我们就得看具体的应⽤场景。是希望可重复的就得⽤List，选择List下常⻅的⼦类。是希望不可重复，选择Set下常⻅的⼦类。
* 如果是 Key-Value 型，那我们会选择Map。如果要保持插⼊顺序的，我们可以选择LinkedHashMap，如果不需要则选择HashMap，如果要排序则选择TreeMap。

# 4.集合进阶与面试

如果你在写代码的时候懂得选择什么样的集合来作为我们的容器的时候，说明你已经入门了Java集合。但是如果想弄懂Java集合，接下来就要更深入的阅读源码和知道它运用了哪些数据结构以及为什么使用这些数据结构时，你就完全弄懂了Java集合。

Java集合是⾯试的重点，我在⾯试的时候⼏乎每家公司都会问集合的问题，从基础到源码，⼀步⼀步深⼊。Java集合⾯试的知识点就不限于基本的⽤法了。可能⾯试官会问你：

* HashMap的数据结构是什么？他是怎么扩容的？底层有没有⽤红⿊树？取Key Hash值是JDK源码是怎么实现的？为什么要这样做？
* HashMap是线程安全的吗？什么是线程安全？有什么更好的解决⽅案？那线程安全的HashMap是怎么实现的？
* HashSet是如何判断Key是重复的？
* ......