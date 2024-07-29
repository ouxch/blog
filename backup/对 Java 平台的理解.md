## `JRE`和`JDK`

`Java`是一种面向对象的语言，通常学习`Java`首先接触的就是`JRE(Java Runtime Environment)`或者`JDK(Java Development Kit)`。


`JRE`：`Java`运行环境，包含了`Java`虚拟机(`JVM`)和`Java`类库，以及一些模块等，它不能⽤于构建新的程序；
`JDK`：可以看作是`JRE`的一个超集，它拥有JRE所拥有的⼀切，提供了更多工具，比如编译器(`javac`)、文档工具(`javadoc`)、调试/诊断/监控工具(`jdb` `jstat` `jmap` `jstack` `jhat`)等，它能够创建和编译程序。

理论上如果只是为了运⾏⼀下`Java`程序的话，安装`JRE`就可以了。要做`Java`编程⽅⾯的⼯作，才需要安装`JDK`。但是这个也不绝对，如果要运行一些`JSP`的`Web`应⽤程序，就是以前那种前后端不分离的，那还是需要安装`JDK`，因为它需要把`JSP`转换为`Java`的`Servlet`，然后用JDK来编译`Servlet`。还有运行时的动态代理也需要`JDK`动态编译。

## `Java`最显著的特性

一是所谓的“书写一次，到处运行”(Write once, run anywhere)，能够非常容易地获得跨平台能力；
二是垃圾收集(GC, Garbage Collection)，`Java`通过垃圾收集器(Garbage Collector)回收分配内存，大部分情况下，程序员不需要自己操心内存的分配和回收。

## `Java`是解释执行的语言

这个说法现在来看已经不太准确。

通常把`Java`分为编译期和运行时，这里说的`Java`的编译和`C/C++`是有着不同的意义的，`Javac`编译`Java`源码生成的`.class`文件里面实际是字节码，而不是可以直接执行的机器码。`Java`通过字节码和`Java`虚拟机`JVM`这种跨平台的抽象，屏蔽了操作系统和硬件的细节，这也是实现“一次编译，到处执行”的基础。

我们开发的`Java`的源代码，首先通过`Javac`编译成为字节码`bytecode`，然后，在运行时通过`Java`虚拟机`JVM`内嵌的解释器将字节码转换成为最终的机器码。

```flow
st=>inputoutput: .java 源代码文件
op1=>operation: javac 编译
io1=>inputoutput: .class 字节码文件
op2=>operation: jvm 解释器逐行解释
io2=>inputoutput: 二进制机器码(机器可以直接执行)

st->op1(right)->io1->op2(right)->io2
```

机器码的运⾏效率肯定是⾼于逐行解释执行的。所以主流的JVM比如我们大多数情况使用的`Oracle JDK`提供的`Hotspot JVM`，都提供了`JIT(Just-In-Time)`编译器，也就是通常所说的动态编译器。`JIT`能够在运行时将热点代码编译成本地机器码，并将字节码对应的机器码保存下来，下次可以直接使⽤。这种情况下部分热点代码就属于编译执行，而不是解释执行了。

至于哪部分属于热点代码，`HotSpot`采⽤的是惰性评估(Lazy Evaluation)的做法，根据⼆⼋定律，消耗⼤部分系统资源的只有那⼀⼩部分的代码(热点代码)。

所以主流`Java`版本，实际是解释和编译混合的一种模式，即所谓的混合模式`-Xmixed`。通常运行在`server`模式的`JVM`，会进行上万次调用以收集足够的信息进行高效的编译，`client`模式这个门限是`1500`次。`Oracle Hotspot JVM`内置了两个不同的`JIT compiler`，`C1`对应前面说的`client`模式，适用于对于启动速度敏感的应用，比如普通`Java`桌面应用；`C2`对应`server`模式，它的优化是为长时间运行的服务器端应用设计的。默认是采用所谓的分层编译`TieredCompilation`。

Java 虚拟机启动时，可以指定不同的参数对运行模式进行选择。 比如，指定`-Xint`，就是告诉`JVM`只进行解释执行，不对代码进行编译，这种模式抛弃了`JIT`可能带来的性能优势。毕竟解释器`interpreter`是逐条读入，逐条解释运行的。与其相对应的，还有一个`-Xcomp`参数，这是告诉`JVM`关闭解释器，不要进行解释执行，或者叫作最大优化级别。但这种模式未必就是最高效的，`-Xcomp`会导致`JVM`启动变慢非常多，同时有些`JIT`编译器优化方式，比如分支预测，如果不进行`profiling`，往往并不能进行有效优化。

除了上面提到的`JIT`，其实还有一种新的编译方式，即所谓的`AOT(Ahead-of-Time Compilation)`，直接将字节码编译成机器代码，这样就避免了`JIT`预热等各方面的开销，比如`Oracle JDK 9`就引入了实验性的`AOT`特性 ，并且增加了新的`jaotc`工具。
利用下面的命令把某个类或者某个模块编译成为`AOT`库：

```bash
jaotc --output libHelloWorld.so HelloWorld.class
jaotc --output libjava.base.so --module java.base
```
然后，在启动时直接指定编译好的 AOT 库：
```bash
java -XX:AOTLibrary=./libHelloWorld.so,./libjava.base.so HelloWorld
```
目前`AOT`编译器的编译质量肯定是比不上`JIT`编译器的，`Oracle JDK`支持分层编译和`AOT`协作使用，这两者并不是二选一的关系，相关文档：[http://openjdk.java.net/jeps/295](http://openjdk.java.net/jeps/295)。`AOT`也不仅仅是只有这一种方式，业界早就有第三方工具(如`GCJ`、`Excelsior JET`)提供相关功能。

另外，`JVM`作为一个强大的平台，不仅仅只有`Java`语言可以运行在`JVM`上，本质上合规的字节码都可以运行，`Java`语言自身也为此提供了便利，所以我们可以看到类似`Kotlin、Clojure、Scala、Groovy、JRuby、Jython`等大量`JVM`语言，活跃在不同的场景。

目前程序主要有两种运行方式：静态编译与动态解释；
静态编译的程序在执行前全部被翻译为机器码，通常将这种方式称为`AOT(Ahead of time)`，即“提前编译”；
解释执行的程序则是一句一句的边翻译边运行，通常将这种方式称为`JIT(Just-in-time)`，即“即时编译”；
`AOT`程序的典型代表是用`C/C++`开发的应用，它们必须在执行前编译成机器码，而`JIT`的代表则非常多，如`JavaScript、Python`等；
事实上，所有脚本语言都支持`JIT`模式。但需要注意的是`JIT`和`AOT`指的是程序运行方式，和编程语言并非强关联的。有些语言既可以以`JIT`方式运行也可以以`AOT`方式运行，如`Java、Python`。

---


## 进一步理解`Java`平台


### `Java`语言特性

泛型、Lambda 等。

### `Java`基础类库

集合、IO/NIO、网络、并发、安全等。

### `Java`类加载机制

常用版本`JDK(如 JDK 8)`内嵌的`Class-Loader`，例如`Bootstrap`、`Application`和`Extension Class-loader`；
自定义`Class-Loader`。

### `Java`类加载过程

加载、验证、链接、初始化(参考周志明的《深入理解Java虚拟机》)。

### `Java`垃圾收集

垃圾收集的基本原理；
常见的垃圾收集器，如`SerialGC、Parallel GC、 CMS、 G1`等，分别适用于什么样的工作负载；

### `Java`工具

`JDK`包含的工具以及`Java`领域内其他工具，编译器、运行时环境、安全工具、诊断和监控工具等。

<!-- ##{"timestamp":1571891640}## -->
<!-- ##{"script":"<script src='https://blog.meekdai.com/Gmeek/plugins/GmeekTOC.js'></script>"}## -->