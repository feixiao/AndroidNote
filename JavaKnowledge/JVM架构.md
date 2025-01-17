# JVM架构

Java的主要优势在于，它被设计为可在具有WORA(write once, run anywhere)概念的各种平台上运行-“一次编写，可在任何地方运行”。

Java源代码会被JDK内置的Java编译器(javac)编译成称为字节码(即.class文件)的中间状态，

这些字节码是带有操作码操作数的十六进制格式，并且JVM可以将这些指令（无需进一步重新编译）解释为操作系统和底层硬件平台可以理解的本地语言。因此，字节码充当独立于平台的中间状态，该状态可在任何JVM之间移植，而与底层操作系统和硬件体系结构无关。

但是，由于JVM是为运行基础硬件和OS结构并与之通信而开发的，因此我们需要为我们的OS版本（Windows，Linux，Mac）和处理器体系结构（x86，x64）选择适当的JVM版本。

我们大多数人都知道Java的上述故事，这里的问题是该过程中最重要的组成部分-JVM被当作一个黑匣子教给我们，它可以神奇地解释字节码并执行许多运行时活动，例如JIT（程序执行期间进行实时）编译和GC（垃圾收集）。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/JVM_Architecture.png)

## Class Loader子系统

JVM驻留在内存(RAM)上。在执行过程中，会使用Class Loader子系统将class文件加载到内存中。这称为Java的动态类加载(dynamic class loading)功能。当它在运行时(而非编译时)首次引用类时，它将加载、链接和初始化类文件(.class)。

- 加载

    Class Loader的主要任务就是把编译后的.class文件加载到内存中。通常，class加载过程会从加载main类(有声明static main()的类)。所有后续的类加载尝试都是根据已运行的类中的类引用完成的，如以下情况所述：

    - 当字节码静态引用一个类时
    - 当字节码创建一个类对象时(例如: Person person = new Person("John"))

    有3种类型的类加载器： 

    - Bootstrap Class Loader

        加载来自rt.jar的标准JDK类，例如引导路径中存在的核心Java API类-$ JAVA_HOME/jre/lib目录（例如java.lang。*包类）。它以C / C ++之类的本地语言实现，并是Java中所有类加载器的父级。

    - Extension Class Loader

        将类加载请求委托给其父类Bootstrap，如果不成功，则从扩展路径中的扩展目录（例如，安全扩展功能）中加载-JAVA_HOME/jre/ext或java.ext.dirs系统指定的任何其他目录的类。该类加载器由Java中的sun.misc.Launcher$ExtClassLoader类实现。

    - System/Application Class Loader

        从系统类路径加载应用程序特定的类，可以使用-cp或-classpath命令来在程序执行时动态设置。它在内部使用映射到java.class.path的环境变量。该类加载器由sun.misc.Launcher$AppClassLoader类用Java实现。

    除了上面讨论的3个主要的类加载器，程序员还可以直接在代码本身上创建用户定义的类加载器。这通过类加载器委托模型保证了应用程序的独立性。这种方法用于Tomcat之类的Web应用程序服务器中，以使Web应用程序和企业解决方案独立运行。

    每个类加载器都有自己的名称空间来保存已加载的类。当类加载器加载类时，它将基于存储在名称空间中的完全合格的类名称（FQCN：Fully Qualified Class Name）搜索该类，以检查该类是否已被加载。即使该类具有相同的FQCN但具有不同的名称空间，也将其视为不同的类。不同的名称空间意味着该类已由另一个类加载器加载。

    它们遵循4个主要原则： 

    - 可见性原则

        该原则指出子类加载器可以看到父类加载器加载的类，但是父类加载器找不到子类加载器加载的类。

    - 唯一性原则

        该原则指出，父类加载的类不应再由子类加载器加载，并确保不会发生重复的类加载。

    - 委托层次结构原则

        为了满足上述2个原则，JVM遵循一个委托层次结构来为每个请求装入的类选择类加载器。首先从最低的子级别开始，Application Class Loader将接收到的类加载请求委托给Extension Class Loader，然后Extension Class Loader将该请求委托给Bootstrap Class Loader。如果在Bootstrap路径中找到了所请求的类，则将加载该类。否则，该请求将再次被转移回Extension Class Loader加载器中从该Loader的扩展路径或自定义指定的路径中查找类。如果它也失败，则请求将返回到Application Class Loader中从该Loader的System类路径中查找该类，并且如果Application Class Loader也未能加载所请求的类，则将获得运行时异常— java.lang.ClassNotFoundException。

    - 无法卸载原则

        即使类加载器可以加载类，但是无法卸载已加载的类。替代卸载功能的是可以删除当前的类加载器，并创建一个新的类加载器。

    ![](https://raw.githubusercontent.com/CharonChui/Pictures/master/java_class_loaders.png)

- 链接

    在遵循以下属性的同时，链接涉及验证和准备已加载的类或接口，其直接父类和实现的接口以及其元素类型。

    - 在链接一个类或接口之前，必须将其完全加载。
    - 在初始化类或接口之前，必须对其进行完全验证和准备（在下一步中）。
    - 如果在链接过程中发生错误，则会将其抛出到程序中的某个位置，在该位置，程序将采取某些操作，这些操作可能直接或间接地需要链接到错误所涉及的类或接口。

    链接分为以下三个阶段： 

    - Verification

        确保.class文件的正确性（代码是否根据Java语言规范正确编写？它是由有效的编译器根据JVM规范生成的吗？）。这是类加载过程中最复杂的测试过程，并且耗时最长。即使链接减慢了类加载过程的速度，它也避免了在执行字节码时多次执行这些检查的需要，从而使整体执行高效而有效。如果验证失败，则会引发运行时错误（java.lang.VerifyError）。例如，执行以下检查: 

        ```java
        - consistent and correctly formatted symbol table
        - final methods / classes not overridden
        - methods respect access control keywords
        - methods have correct number and type of parameters
        - bytecode doesn’t manipulate stack incorrectly
        - variables are initialized before being read
        - variables are a value of the correct type
        ```

        

    - Preparation

        为静态存储和JVM使用的任何数据结构（例如方法表）分配内存。静态字段已创建并初始化为其默认值，但是，在此阶段不执行任何初始化程序或代码，因为这是初始化的一部分。

    - Resolution

        用直接引用替换类型中的符号引用。通过搜索方法区域以找到引用的实体来完成此操作。

- 初始化

    在这里，将执行每个加载的类或接口的初始化逻辑（例如，调用类的构造函数）。由于JVM是多线程的，因此应在适当同步的情况下非常仔细地进行类或接口的初始化，以避免其他线程尝试同时初始化同一类或接口（即使其成为线程安全的）。

    这是类加载的最后阶段，所有静态变量都分配有代码中定义的原始值，并且将执行静态块（如果有）。这是在类中从上到下，从类层次结构中的父级到子级逐行执行的。

### 对象的创建

 Java 对象的创建过程： 
1. 类加载检查
    虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。
2. 分配内存
    在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。分配方式有 “指针碰撞” 和 “空闲列表” 两种，选择那种分配方式由 Java 堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。
3. 初始化零值
    内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。
4. 设置对象头
    初始化零值完成之后，虚拟机要对对象进行必要的设置，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希吗、对象的 GC 分代年龄等信息。 这些信息存放在对象头中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。
5. 执行init方法
    在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，<init> 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 <init> 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。


### 对象的内存布局
在 Hotspot 虚拟机中，对象在内存中的布局可以分为3快区域：对象头、实例数据和对齐填充。
Hotspot虚拟机的对象头包括两部分信息，第一部分用于存储对象自身的自身运行时数据（哈希吗、GC分代年龄、锁状态标志等等），另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。
实例数据部分是对象真正存储的有效信息，也是在程序中所定义的各种类型的字段内容。
对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。 因为Hotspot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。


### 对象的访问定位
建立对象就是为了使用对象，我们的Java程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式有虚拟机实现而定，目前主流的访问方式有①使用句柄和②直接指针两种：
1. 句柄
    如果使用句柄的话，那么Java堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；
    
    ![](https://raw.githubusercontent.com/CharonChui/Pictures/master/java_refe_jubing.png)
    
2. **直接指针：** 如果使用直接指针访问，那么 Java 堆对像的布局中就必须考虑如何防止访问类型数据的相关信息，reference 中存储的直接就是对象的地址。

    ![](https://raw.githubusercontent.com/CharonChui/Pictures/master/java_refe_direct.png)

这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。



## Runtime Data Area

运行时数据区是当JVM程序在OS上运行时分配的存储区。除了读取.class文件之外，Class Loader子系统还会生成相应的二进制数据，并将以下信息分别保存在每个类的Method区域中： 

- 加载的类及其直接父类的全限定名称
- .class文件是否与Class / Interface / Enum相关
- 修饰符，静态变量和方法信息等。

然后，对于每个已加载的.class文件，它都会按照java.lang包中的定义，恰好创建一个Class对象来表示堆内存中的文件。稍后，在我们的代码中，可以使用此Class对象读取类级别的信息（类名称，父名称，方法，变量信息，静态变量等）。

[具体可参考Java内存模型](./JavaKnowledge/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.md)



## Execution Engine

字节码的实际执行在这里进行。执行引擎通过读取分配给上述运行时数据区域的数据逐行执行字节码中的指令。



### Interpreter

解释器解释字节码并一对一执行指令。因此，它可以快速解释一个字节码行，但是执行解释后的结果是一项较慢的任务。缺点是，当多次调用一个方法时，每次都需要新的解释和较慢的执行速度。

### Just-In-Time (JIT) Compiler

如果只有解释器可用，则当多次调用一个方法时，每次也会进行解释，如果有效处理，这将是多余的操作。使用JIT编译器已经可以做到这一点。首先，它将整个字节码编译为本地代码（机器代码）。然后，对于重复的方法调用，它直接提供了本机代码，使用本机代码的执行比单步解释指令要快得多。本机代码存储在缓存中，因此可以更快地执行编译后的代码。

但是，即使对于JIT编译器，编译所花费的时间也要比解释器所花费的时间更多。对于仅执行一次的代码段，最好对其进行解释而不是进行编译。同样，本机代码也存储在高速缓存中，这是一种昂贵的资源。在这种情况下，JIT编译器会在内部检查每个方法调用的频率，并仅在所选方法发生超过特定时间级别时才决定编译每个方法。自适应编译的想法已在Oracle Hotspot VM中使用。

当JVM供应商引入性能优化时，执行引擎有资格成为关键子系统。在这些工作中，以下4个组件可以大大提高其性能： 

- 中间代码生成器生成中间代码。
- 代码优化器负责优化上面生成的中间代码。
- 目标代码生成器负责生成本机代码（即机器代码）。
- Profiler是一个特殊的组件，负责查找性能热点（例如，多次调用一种方法的实例）

### 供应商的优化方法

- Oracle Hotspot VMs

    Oracle通过流行的JIT编译器模型Hotspot Compiler实现了其标准Java VM的2种实现。通过分析，它可以确定最需要JIT编译的热点，然后将代码的那些性能关键部分编译为本机代码。随着时间的流逝，如果不再频繁调用这种已编译的方法，它将把该方法标识为不再是热点，并迅速从缓存中删除本机代码，并开始在解释器模式下运行。这种方法可以提高性能，同时避免不必要地编译很少使用的代码。此外，Hotspot Compiler可以即时确定使用lining等技术来优化已编译代码的最佳方式。编译器执行的运行时分析使它可以消除在确定哪些优化将产生最大性能收益方面的猜测。
    这些虚拟机使用相同的运行时（解释器，内存，线程），但是将自定义构建JIT编译器的实现，如下所述。

    Oracle Java Hotspot Client VM是Oracle JDK和JRE的默认VM技术。它通过减少应用程序启动时间和内存占用量而在客户端环境中运行应用程序时进行了优化，以实现最佳性能。
    Oracle Java Hotspot Server VM旨在为在服务器环境中运行的应用程序提供最高的程序执行速度。此处使用的JIT编译器称为“高级动态优化编译器”，它使用更复杂和多样化的性能优化技术。通过使用服务器命令行选项（例如Java服务器MyApp）来调用Java HotSpot Server VM。

    Oracle的Java Hotspot技术以其快速的内存分配，快速高效的GC以及易于在大型共享内存多处理器服务器中扩展的线程处理能力而闻名。

- IBM AOT (Ahead-Of-Time) Compiling

    这里的特长是这些JVM共享通过共享缓存编译的本机代码，因此已经通过AOT编译器编译的代码可以由另一个JVM使用，而无需编译。另外，IBM JVM通过使用AOT编译器将代码预编译为JXE（Java可执行文件）文件格式，提供了一种快速的执行方式。



### Garbage Collector (GC)

只要引用了一个对象，JVM就会认为它是活动的。一旦不再引用对象，因此应用程序代码无法访问该对象，则垃圾收集器将其删除并回收未使用的内存。通常，垃圾回收是在后台进行的，但是我们可以通过调用System.gc()方法来触发垃圾回收（同样，无法保证执行。因此，请调用Thread.sleep（1000）并等待GC完成）。[具体内存回收部分请看JVM垃圾回收机制](./JavaKnowledge/JVM%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6.md)



### JVM线程

我们讨论了如何执行Java程序，但没有具体提及执行程序。实际上，为了执行我们前面讨论的每个任务，JVM并发运行多个线程。这些线程中的一些带有编程逻辑，并且是由程序（应用程序线程）创建的，而其余的则是由JVM本身创建的，以承担系统中的后台任务（系统线程）。

主应用程序线程是作为调用公共静态void main（String []）的一部分而创建的主线程，而所有其他应用程序线程都是由该主线程创建的。应用程序线程执行诸如执行以main（）方法开头的指令，在Heap区域中创建对象（如果它在任何方法逻辑中找到新关键字）之类的任务等。

主要的系统线程有： 

- Compiler threads

    在运行时，这些线程将字节码编译为本地代码。

- GC threads

    所有与GC相关的活动均由这些线程执行。

- Periodic task thread

    计划周期性操作执行的计时器事件（即中断）由该线程执行。

- Signal dispatcher thread

    该线程接收发送到JVM进程的信号，并通过调用适当的JVM方法在JVM内部对其进行处理。

- VM thread

    前提条件是，某些操作需要JVM到达安全点才能执行，在该点不再进行对Heap区域的修改。这种情况的示例是“世界停止”垃圾回收，线程堆栈转储，线程挂起和有偏向的锁吊销。这些操作可以在称为VM线程的特殊线程上执行。



