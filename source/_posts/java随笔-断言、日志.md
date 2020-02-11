---
title: java随笔--断言、日志
date: 2019-11-06 20:54:39
tags:	
	JAVA随笔
categories:
	学习笔记
---

### 断言

断言机制允许在测试期间向代码中插入一些检查语句，当代码发布时，这些插入的检测语句会被自动地移走。Java语言引入了关键字assert，有两种形式：

```java
assert 条件;
assert 条件:表达式;
```

这两种形式都会对条件进行检测， 如果结果为false, 则抛出一个AssertionError 异常。在第二种形式中，表达式将被传人AssertionError 的构造器， 并转换成一个消息字符串。

在默认情况下， 断言被禁用。可以在运行程序时启用断言，断言的启用和禁用不必重新编译程序。启用或禁用断言是类加载器( class loader ) 的功能。当断言被禁用时， 类加载器将跳过断言代码， 因此，不会降低程序运
行的速度。

```
java -enableassertions MyApp
```

不应该使用断言向程序的其他部分通告发生了可恢复性的错误， 或者，不应该作为程序向用户通告问题的手段。断言只应该用于在**测试阶段**确定程序内部的错误位置。

### 日志

日志可以替换测试代码中使用的`System.out.println()`，不像输出语句测试完还需要删除。

```java
//基本日志
Logger.getGlobal().info("File->OPen menu item selected");
//如果在适当的地方（如main开始）调用，会取消所有日志
Logger.getGlobal().setLevel(Level.OFF);

//高级日志（目前学习阶段用不到，以后需要再进行补充）
```

