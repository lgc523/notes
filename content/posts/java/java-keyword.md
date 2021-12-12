---
title: "Java-KeyWord"
date: 2021-12-12T09:37:51+08:00
draft: true
toc: true
images:
tags: 
  - java
---

final 关键字可以用在 类、方法、参数上面，来告知编译器一块数据是恒等不变的。

final 让基本数据类型数值恒定不变，引用对象(包括数组)引用恒定不变。

被 static final 修饰的域只占据一段不能改变的存储空间，只有 final 修饰的域只在有效的生命周期内不变。

## blank final

编译器会保证 final 修饰的域在使用前必须被初始化，blank final 提供了更大的灵活性。

如果只是 final 修饰符 ，所有的 【构造函数】 都要进行初始化，static final 需要直接初始化或者在静态代码块里面初始化。

## final param

方法参数列表中将参数用 final  修饰，约束方法内的代码块不能更改参数引用/数值，只能读取，引用对象可以修改引用内的数据。

## final method

- 锁定方法，防止任何继承类修改，约束方法行为
- 早期编译器 inline 优化，消除方法调用寄存器、栈开销。
- private method 默认 final 修饰，无意义。

## final class

Final 类 禁止继承，所有方法都隐式指定为 final。

## Oriented future

不用 final 修饰类，方便后面进行拓展和改造，eg Vector、HashTable。

## static

所有 static 对象和代码块都会在加载时按照程序中的顺序加载进行初始化。

static 方法不能访问非 static 修饰的方法（静态对象的引用，可以传入静态方法中，但是一般这个时候对象的状态都是各自类型的零值，当然可以继续修改引用）。

