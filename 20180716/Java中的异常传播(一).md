---
title: Go安装
layout: post
date: 2018-06-18 17:18:16
comments: true
post_catalog: true
tags:
- Go
categories:
- Go
keywords: Go
permalink: install-go
description:
---
## 安装Go
1. 下载Go安装包
GO安装包下载地址：[https://studygolang.com/dl](https://studygolang.com/dl)
![](https://github.com/yida-lxw/blog/blob/master/20180716/images/_1529314830_16105.png?raw=true)
2.
---
title: Java中的异常传播(一)
layout: post
date: 2018-07-16 23:48:06
comments: true
post_catalog: true
tags:
- Java
categories:
- Java
keywords: Java,Exception
permalink: java-exceptions_1
description:
---

Java里的异常其实是一种信号，该信号表明了在代码执行过程中发生的一些重要的或未预测到的情况。举个例子，比如一个被请求的文件找不到了，或者一个数组的索引越界了，又或者某个网络连接失败了。在代码里针对上述情况进行显式检查很容易导致代码变得令人费解。Java提供了一种异常处理机制来系统性处理诸如此类的错误情况。

这种异常机制是围绕着try-catch这种形式来构建的。throw一个异常就等价于发送了一个未预测到的错误情况发生了的信号。catch一个异常是为了采取合适的方式来处理这个异常。异常会被异常处理器捕获，在同一个上下文环境里，已经被抛出的异常不会再被捕获。程序运行时的行为决定了什么类型的异常将会被抛出，以及该如何捕获它们。throw-catch原理是嵌入在try-catch-finally结构里。

在JVM里可以同时执行多个线程。每个线程拥有它各自的运行时栈空间(有时也称之为“栈”或“执行栈”)，这些栈用于协助处理方法的执行。线程栈里的每个元素(这些元素被称之为活跃记录或者栈帧)对应着一个方法调用。每个包含于当前活跃记录的新产生的调用结果将会被压栈，而这些调用结果存储了关于线程的本地变量的所有相关信息。这个方法连同当前处理栈顶的栈帧一起表示着当前的方法执行。当这个方法执行完毕，该方法的活跃记录将会被弹出栈。而那些处于栈顶仍未覆盖到的活跃记录对应的那些方法将会紧接着被执行。当该方法的调用还未执行完毕，那么就认为栈中的该方法是活跃的。在任何时刻，在运行时栈中，这些活跃的方法组成了线程执行过程的栈轨迹即stack trace。

下面这个简单程序演示了方法的执行。它计算一组Interger类型数字的平均值，传入所有Integer类型数字的总和以及这些数字的总个数。这里定义了3个函数：
+ main函数调用printAverage函数，printAverage函数需要传入所有Integer类型数字的总和以及所有Integer类型数字的总个数。(1)
+ printAverage函数内部会反过来调用computeAverage函数。(3)
+ computeAverage()用于计算数字的平均值，并返回计算结果。(7)
~~~java
public class Average1 {

public static void main(String[] args) {
printAverage(100,0);                                  // (1)
System.out.println("Exit main().");                   // (2)
}

public static void printAverage(int totalSum, int totalNumber) {
int average = computeAverage(totalSum, totalNumber);  // (3)
System.out.println("Average = " +                     // (4)
totalSum + " / " + totalNumber + " = " + average);
System.out.println("Exit printAverage().");           // (5)
}

public static int computeAverage(int sum, int number) {
System.out.println("Computing average.");             // (6)
return sum/number;                                    // (7)
}
}
~~~
上述程序执行后会输出如下结果：
~~~java
Computing average.
Average = 100 / 20 = 5
Exit printAverage().
Exit main().
~~~
![方法执行流程](images/_方法执行流程_1531497866_22167.jpg?raw=true)
~~~java
printAverage(100, 20);                                // (1)
~~~
替换为
~~~java
printAverage(100,  0);                                // (1)
~~~
然后再次运行程序，程序的输出结果将会如下所示：
~~~java
Computing average.
Exception in thread "main" java.lang.ArithmeticException: / by zero
at Average1.computeAverage(Average1.java:18)
at Average1.printAverage(Average1.java:10)
at Average1.main(Average1.java:5)
~~~
下图演示了上述程序的执行流程。代码一切执行正常，直到运行至computeAverage函数中标记(7)处的语句，在计算sum/number这个算式时发生了错误，因为拿一个数字除以零是一个非法操作。这个错误是JVM通过抛出一个ArithmeticException异常来发出的。然后JVM通过运行时栈向上层传播这个异常。
![异常传播](images/_异常传播_1531499702_20289.gif?raw=true)

若在对一个二进制表达式的左操作数进行解析求值时抛出了异常，则右操作数将会跳过不处理。类似的，若对一系列表达式(比如，方法调用时实际参数列表)进行解析处理时抛出了异常，然后剩下的参数列表将会被跳过不处理。
若类似先前展示的终端输出结果的线程执行栈的调用轨迹信息中不打印行号，这时你需要添加一个JVM参数，如下所示：
~~~
-XX:-OmitStackTraceInFastThrow
~~~