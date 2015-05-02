---
layout: post
title: Java访问修饰符
categories: java基础
tags: 笔记
---

#### 访问修饰符

java提供了四种访问修饰符，从大到小的权限依次为：public、protected、包访问权限（没有关键词）、private。

访问修饰符可以修饰类的成员和方法。

#### 包访问权限

默认访问权限就是包访问权限，包访问权限没有任何关键字。对于同一个包当中的类对这个成员有访问权限，其他包的无访问权限。

#### public访问权限

被public所修饰的属性和方法可以被所有类访问。

#### private访问权限

被private所修饰的属性和方法只能在该类内部使用。

#### protected访问权限

被protected所修饰的属性和方法可以在类内部、相同包以及该类的子类内部所访问。

子类内部所访问的含义：相当于protected的属性或者方法被继承下来了，可以通过子类对象访问，但是在子类中new一个父类对象同样不可以使用（不在同一个包的时候,并且是在子类当中）。

|可见/访问性|在同一类中|同一包中|不同包中|同一包子类中|不同包子类中 |
|:--|:--|:--|:--|:--|:--|
| public | yes | yes  | yes | yes | yes |
| protected  | yes | yes | no | yes | yes |
| package  | yes | yes | no | yes | no |
| private | yes | no | no | no | no |


---EOF---

