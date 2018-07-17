---
title: Java中的异常传播(二)
layout: post
date: 2018-07-18 02:49:25
comments: true
post_catalog: true
tags:
- Java
categories:
- Java
keywords: Java
permalink: java-exceptions_2
description:
---

### 什么是异常传播？
当异常发生时，这个异常将会从栈顶往栈底方向下降，如果你没有捕获这个异常，那么这个异常会继续抛向方法的上层调用者，以此类推，直至这个异常被捕获或者异常传播至栈底。异常在栈中的传播过程叫做异常传播，而且异常传播机制发生的前提条件是抛出的异常是UncheckedException(非检查性异常)。

### 非检查性异常的传播机制
下面这段示例代码演示了非检查性异常的传播机制：
~~~java
/**
 * @Author Lanxiaowei
 * @Date 2018-07-17 22:01
 * @Description 非检查性异常的传播机制演示代码
 */
public class ExceptionTest1 {
    void m() {
        int num = 50 / 0; // 此处将会抛出非检查性异常
        // 由于方法m里未对异常进行任何捕获处理，故异常将会传播至方法n
    }

    void n() {
        m();
        // 异常将会继续传播至它的上层调用者方法p
    }

    void p() {
        try {
            n(); // 异常被捕获并处理
        } catch (Exception e) {
            System.out.println("Exception handled");
        }
    }

    public static void main(String args[]) {
        ExceptionTest1 test = new ExceptionTest1();
        test.p();
        System.out.println("Method was in normal flow...");
    }
}
~~~
> 注意：默认情况下，如果你没有对非检查性异常做任何处理，那么非检查性异常会自动在程序的调用链中逐层向当前方法的上层调用者进行传播，直到main方法，若main方法也没有对该异常进行任何处理，那么JVM会调用默认的异常处理器打印堆栈调用轨迹。

### 检查性异常的传播机制
与非检查性异常不同的是，检查性异常不会触发异常的传播机制，编译器会在编译期就强制你要么使用throw关键字将异常抛给上层调用者去处理，要么自己通过try-catch块捕获异常并处理，如果此时你不做任何处理，强行对代码进行编译，将会抛出代码编译错误。
>简单解释一下throw和throws关键字，throw关键字用于向当前方法的上层调用者抛出一个异常，说得形象一点就是，当前方法可能会抛出异常，但是我不想自己处理或者我没有能力处理，然后你甩锅给上层调用者。而throws关键字一般用于方法声明部分，以警示当前方法的上层调用者，调用当前方法可能会抛出throws关键字后面的这些异常(就好比你甩锅之前还暖心的提醒一下人家)。如果你甩的锅是检查性异常，那么接锅的人要么继续向上层甩锅，要么自己解决掉。如果你甩的锅是非检查性异常，那么接锅的人可以选择规规矩矩的处理掉，可以选择继续向上层甩锅，也可以选择视而不见，然后这个锅会自动在调用链中向上(这里说的向上指的是上层方法调用者)传播，这个向上传播的过程就是异常传播机制，这个传播过程也类似水泡从水底向上冒的过程。

理论结合代码示例会更配哦，来看代码：
![](https://github.com/yida-lxw/blog/raw/master/20180718/images/_1531838890_10288.png?raw=true)
![](https://github.com/yida-lxw/blog/raw/master/20180718/images/_1531839357_30549.png?raw=true)
看到这里，有些骚年，估计会心里嘀咕：不要跟我扯什么底层原理机制，我撸代码就是复制粘贴，敲起键盘就是一把梭。我只关心我应该在什么情况下选择使用哪种异常，什么情况下我应该选择抛出异常，我怎么样操作才是使用异常的正确姿势，使用异常会有什么性能影响？
![](https://github.com/yida-lxw/blog/raw/master/20180718/images/_1531841002_26956.jpg?raw=true)

### 我应该选择使用哪种异常？
Java里的异常分两种类型：
1.编译器强制的异常，也称作检查性异常
2.运行时异常，也叫做非检查性异常
检查性异常是Exception类或者其子类的一个实例，RuntimeException类以及它这个分支下的所有子类除外。编译器期望所有检查性异常都被正确处理，采用强制编译报错或者强制使用thrown关键字声明在方法签名部分，目的只有一个：尽最大努力引起编程者注意到这个相对非检查性异常而言严重性更高的异常。或者说编译器认为检查性异常一般是比较严重的异常，不解决此类异常，程序可能无法继续运行下去，故强制编程人员去主动解决它。
我们常见的一个检查性异常就是IOException，顾名思义，当IO输入流或输出流意外中止时都会抛出此异常。我们读取文件里的内容，首先需要加载该文件，然后通过IO输入流读取文件内容，拿到该文件内容再去做其他逻辑处理，即这里的文件读取操作是后续操作的前置条件。比如你可以使用BufferedReader 对象的readLine方法逐行读取文件内容，但是可能在读取文件过程中文件被删除，总之，IO读写操作可能会由于各种原因意外被中断，此时就会抛出IOException。如果你不处理此异常，后续程序可能无法继续。

运行时异常是RuntimeException类或其子类的实例，你不需要通过throws语句在方法签名上显式声明运行时异常，同样的，方法的调用者也不会被强迫去处理运行时异常(当然你也可以选择显式处理运行时异常)，运行时异常通常只在JVM环境内部出现问题(因为运行时异常默认会通过异常传播机制传播至JVM内部)时抛出，但这不意味着编程人员可以随意的“吞掉”异常，因为这些异常有助于JVM更方便的管理异常。比如ArithmeticException异常就是运行时异常的一种，当你拿一个数字除以零，在编译期你可能并没意识到这个异常可能发生，只有等到程序执行到那句代码，然后异常产生并传播至JVM，最后JVM调用默认异常处理器打印堆栈信息，你才会意识到异常的发生，其实运行时异常已经在栈里从栈顶贯穿到栈底了，而对于编译人员可能毫无感知。

决定你是该选择使用检查性异常还是运行时异常，你只需要遵循这个基本原则：如果这个异常属于方法调用者必须处理或者当前你不知道怎么处理的那种，那么你应该选择检查性异常，否则你应该考虑使用运行时异常。

### 什么情况下我应该选择抛出异常？
在Java语言规范里，对于何时应该选择抛出异常，给出了如下定义：
>an exception will be thrown when semantic constraints are violated

翻译过来，意思就是当代码违反了语义约束，你就应该选择抛出异常。这意味着当下面两种情况任意一种发生了，你就应该选择抛出异常：
+通常情况下不太可能发生的事情发生了，所谓不太可能意思就是发生这种情况可能是因为你粗心疏忽导致的，其实这种情况是可以有效避免的。
+某种操作严重违反你的可接收范围，意思就是说某种情况超过了你的忍耐限度，比如你在写一个邮箱注册功能，而用户传给你一个空字符串或null值或非email的字符串
为了有助于你们更好的理解上面两点，请阅读下面的代码示例：
###### 示例1
~~~java
public Passenger getPassenger_1() {
        Passenger passenger = null;
        try {
            //根据乘客姓名查询乘客的航班记录信息
            passenger = searchPassengerFlightRecord("Jane Doe");
        } catch(NoPassengerFoundException npfe) {   //如果找不到该乘客，则抛出NoPassengerFoundException异常
                //do something
        }
        return passenger;
    }
~~~

###### 示例2
~~~java
public Passenger getPassenger_2() {
        Passenger passenger = searchPassengerFlightRecord("Jane Doe");
        if(null == passenger) {  //如果找不到该乘客，这里选择返回null值给上层调用者
            return null;
        }
        return passenger;
    }
~~~
在上面第一个例子中，如果查询找不到结果，将会直接抛出NoPassengerFoundException，然而在第二个例子中，选择了进行null检测然后返回null值给方法调用者。对于程序开发人员来讲，在每天的撸码过程中或多或少都会遇到类似这样的情况。此时你需要考虑的是，查询返回的结果集为空，这是一个正常情况吗？或者说查询返回结果集为空不是一个正常情况，它是一个你无法接收的事实？显然，查询后服务器没有返回任何数据，是很正常的，因为可能确实没有符合你查询条件的数据呀。因此，为了更加明智的使用异常，在上述示例描述的类似情况下，你应该选择采用示例2的方式，而不是示例1。
那么什么情况下我们应该选择抛出异常呢？还是上面查询乘客航班信息的例子，假如调用searchPassengerFlightRecord方法的this对象为null，那么此时就违反了getPassenger_1方法的语义，所谓违法了语义意思就是this对象为null，导致getPassenger_1方法无法正常执行，这违法了getPassenger_1方法的本意。注意：导致方法无法执行和最终方法返回null这是两个概念，显然前者对事态的影响程度更深，故此时你应该选择抛出异常。但是使用异常也不要太过于高频，虽然catch一个异常只有大概0.0x毫秒，但是e.printStackTrace()打印堆栈信息这个操作却是捕获异常操作消耗时间的10倍之多，简直就是性能杀手。

### 我该怎么样操作才是使用异常的正确姿势？
每个Java程序员都应该拥有能够捕获各种异常并知道如何处理它们的能力。当代码必须将底层系统级别且隐藏极深的异常信息转换成一种对用户友好的应用程序级别的错误信息时，这件事就显得复杂起来了。特别是针对API接口编程的情况下，当你将你的代码嵌入到另一个应用程序，而你并没有配套的GUI，此时通常，有以下3种方式来处理异常：
1.捕获并处理所有异常
2.通过throws关键字声明所有异常，将其抛给上层调用者
3.捕获异常，然后在catch块里将其转换成自定义的异常并重新抛出
让我们一起来思考上述每一种方式可能存在的问题，然后尝试选择一种合适可行的解决方案。

###### 示例1
~~~java
public Passenger getPassenger() {
        try {
            Passenger flier = this.searchPassengerFlightRecord("John Doe");
        } catch (MalformedURLException ume) {
            //do something
        } catch (SQLException sqle) {
            //do something
        }
    }
~~~
示例1演示了第一种方式，采用的是一种极端方式，捕获了所有可能发生的异常，然后通知方法调用者。这种方式需要在捕获到异常之后，选择返回null值或者其他约定好的特殊值给方法调用者以通知方法调用者发生了什么异常。这种方式存在明显的缺点，你抛弃了所有编译时支持，而且方法调用者需要测试返回值的每一种情况，而且正常的代码处理逻辑与异常处理逻辑混杂在一起，使得代码看起来混乱不堪，有代码洁癖的童鞋表示会看不下去。

###### 示例2
~~~java
Passenger getPassenger() throws MalformedURLException, SQLException {
        Passenger flier = this.searchPassengerFlightRecord("John Doe");
}
~~~
示例2则走向了另一个极端，getPassenger方法通过throws关键字声明了所有可能发生的异常，不在方法内部对异常做任何处理，直接将异常甩锅给上层方法调用者。然而这种方式并不是一种推荐的方式，因为你将所有异常处理的职责全部推给方法调用者，这可能会引起很多问题。假如getPassenger()方法是航空订票系统提供给你的接口，你在调用他们接口时，他们却要求你处理所有可能发生的异常，这增加了你的处理负担，此外，接口调用方并不关心MalformedURLException和SQLException异常，接口调用方关心的是查询成功还是失败，仅此而已。也就是说在抛出异常之前，你需要搞清楚接口编写者和接口调用者在处理异常方面的相关职责划分。

###### 示例3
~~~java
Passenger getPassenger() throws TravelException {
        try {
            Passenger flier = this.searchPassengerFlightRecord("Gary Smith");
        } catch (MalformedURLException ume) {
            //do something
            throw new TravelException("Search Failed", ume);
        } catch (SQLException sqle) {
            //do something
            throw new TravelException("Search Failed", sqle);
        }
    }
~~~
示例3与示例1和示例2相比，就显得中规中矩。示例3是通过自定义了一个TravelException异常，然后捕获所有异常然后转换成TravelException异常抛给方法调用者。这个类有个显著特点就是将实际系统级别异常转成应用程序级别且对用户更加友好的异常，并且通过TravelException类构造函数的第二个参数将源异常传递了下去，这样做是为了方便后续debug。这种方式使得在异构系统中，你的API接口设计显得更加优雅。
下面是上面示例中出现的TravelException类的示例代码，它的构造函数有两个参数，第一个参数用于显式具体的异常提示信息，而第二个参数表示实际的异常。这个自定义异常的示例代码演示了如何将自定义的对用户友好的异常提示信息和原始异常一起包装成一个自定义异常，这种封装操作的好处是，假如方法调用者需要了解底层实际抛出的异常，他们可以调用getHiddenException方法即可。这样方便于方法调用者自己决定如何处理指定的原始异常还是只关心TravelException。
~~~java
public class TravelException extends Exception {
    private Exception hiddenException_;
    public TravelException(String error, Exception excp) {
        super(error);
        hiddenException_ = excp;
    }
    public Exception getHiddenException() {
        return(hiddenException_);
    }
}
~~~
### 异常处理存在的性能影响
异常处理是有代价的，为了更好的理解其中的一些问题，让我们一起来看看其中的一些异常处理机制。在JVM内部，维护着一个方法执行栈(也有人将其叫做调用栈)，该方法执行栈中维护管理着线程执行过的方法列表，方法列表里第一个方法就是线程执行到的第一个方法，而最后方法就是当前方法，方法执行栈演示了一个线程执行到当前方法经历过的方法执行轨迹。
![](https://github.com/yida-lxw/blog/raw/master/20180718/images/_1531850825_8096.png?raw=true)
上面以图片的形式演示了我们上述示例代码中的方法执行栈情况。在JVM内部，方法是通过方法执行栈来维护自己的执行状态的，每个方法都有一个属于自己的栈帧(栈帧里保存着方法的本地变量、方法参数、方法的返回值以及其他JVM需要的关键信息。)，当一个方法将要被执行之前，会被压栈。当该方法执行完毕，则会被弹出栈。同时下面紧挨着的那个栈帧会自动来到栈顶，并指向当前执行方法。
因此，为了便于比较，请思考一下，在上述示例1中，执行栈中将会发生什么呢？如果在执行searchPassengerFlightRecord方法时，抛出一个NoPassengerFoundException异常(此时searchPassengerFlightRecord方法的栈帧处于栈顶位置)，后续的代码会被迫中止，JVM会根据Java里的异常处理机制来管理控制异常。JVM首先会在当前searchPassengerFlightRecord方法中查找是否包含了catch代码块(即判断你是否有对异常进行处理)，并且该catch代码块捕获的是NoPassengerFoundException异常，如果没有找到这样的语句，那么JVM将会把当前方法的栈帧从方法执行栈中弹出，然后当前方法的调用者将会自动移动到栈顶成为新的当前方法。然后JVM继续在当前栈顶方法中查找是否有使用catch代码块捕获NoPassengerFoundException异常。如果JVM仍然没有找到，继续将当前方法的栈帧弹出，然后下一个方法继续自动上升到栈顶成为新的当前方法，JVM再次在当前方法中查找，依次类推，直到找到为止。否则，如果当前方法的执行线程是最后一个非守护线程，那么应用程序将会被终止，同时会将执行栈里的方法调用轨迹信息打印到终端控制台里。简而言之，抛出的异常交给JVM去处理需要很大的开销，比如中断一个执行中的方法相比于让方法正常执行完毕，前者是一个非常昂贵的操作。

### 最后结束语
为了提高异常的使用效率以及良好的编程实践，充分理解异常的处理机制是很有必要的，异常处理应该是开发过程中必不可少的一部分，并不是可有可无的一部分。通过异常提供的处理机制你可以开发出更健壮更独立的应用程序，而不是只靠运气。希望大家以后在写代码时不再迷茫于该如何处理异常的问题上，也希望我的文字能给你带来一点学习上的帮助。













