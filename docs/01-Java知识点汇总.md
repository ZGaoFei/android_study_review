# JVM
## JVM 工作流程
![](https://user-gold-cdn.xitu.io/2019/6/23/16b833f4a4906226?w=448&h=592&f=jpeg&s=44057)

## 运行时数据区（Runtime Data Area）
![](https://user-gold-cdn.xitu.io/2019/6/23/16b833f4a499f6fe?w=868&h=497&f=webp&s=46378)

### 程序计数器
**程序计数器（Program Counter Register）** 是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。

字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

由于 Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，**在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）都只会执行一条线程中的指令**。

因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

- 如果线程正在执行的是一个 Java 方法，这个计数器记录的是正在执行的**虚拟机字节码指令的地址**。
- 如果线程正在执行的是一个 Native 方法，这个计数器值则为空（Undefined）。

**此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。**

### Java 虚拟机栈
**Java 虚拟机栈（Java Virtual Machine Stacks**）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是 Java 方法执行的内存模型，每个方法在执行的同时都会创建一个**栈帧（Stack Frame）** 用于存储**局部变量表、操作数栈、动态链接、方法出口**等消息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

**局部变量表**存放了编译器可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）和 returnAddress 类型（指向了一条字节码指令的地址）。

其中 64 位长度的 long 和 double 类型的数据会占用两个局部变量空间（Slot），其余的数据类型只占用一个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

在 Java 虚拟机规范中，对这个区域规定了两种异常状态：
- 如果线程请求的栈深度大于虚拟机所允许的的深度，将抛出 **StackOverflowError** 异常。
- 如果虚拟机栈可以动态扩展（当前大部分的Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈），如果扩展时无法申请到足够的内存，就会抛出 **OutOfMemoryError** 异常。

### 本地方法栈
**本地方法栈（Native Method Stack）** 与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。

在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（例如：Sun HotSpot虚拟机）直接就把虚拟机栈和本地方法栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出 StackOverflowError 和 OutOfMemoryError 异常。

### Java 堆
对于大多数应用来说，**Java 堆（Java Heap）** 是 Java 虚拟机所管理的的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。

Java堆是垃圾收集器管理的主要区域，从内存回收的角度来看，由于现在收集器基本采用分代收集算法，所以Java堆中还可以细分为：新生代和老年代；再细致一点的有 Eden 空间、From Survivor 空间、To Survivor 空间等。

从内存分配的角度来看，线程共享的Java堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB）。不过无论如何划分，都与存放内容无关，无论哪个区域，存储的仍然是对象实例，进一步划分的目的是为了更好地回收内存，或者更快地分配内存。

### 方法区
**方法区（Method Area**）与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

**运行时常量池（Runtime Constant Pool）** 是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是**常量池（Constant Pool Table）**，用于存放编译器生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时就会抛出 OutOfMemoryError 异常。

## 方法指令
| 指令 | 说明|
|----------|-----|
| invokeinterface | 用以调用接口方法 |
| invokevirtual | 指令用于调用对象的实例方法 |
| invokestatic | 用以调用类/静态方法 |
| invokespecial | 用于调用一些需要特殊处理的实例方法，包括实例初始化方法、私有方法和父类方法 |

## 类加载器
| 类加载器 | 说明|
|----------|:-----|
| BootstrapClassLoader | Bootstrap 类加载器负责加载 rt.jar 中的 JDK 类文件，它是所有类加载器的父加载器。Bootstrap 类加载器没有任何父类加载器，如果你调用 String.class.getClassLoader()，会返回 null，任何基于此的代码会抛出 NUllPointerException 异常。Bootstrap 加载器被称为初始类加载器 |
| ExtClassLoader | 而 Extension 将加载类的请求先委托给它的父加载器，也就是Bootstrap，如果没有成功加载的话，再从 jre/lib/ext 目录下或者 java.ext.dirs 系统属性定义的目录下加载类。Extension 加载器由 sun.misc.Launcher$ExtClassLoader 实现 |
| AppClassLoader | 第三种默认的加载器就是 System 类加载器（又叫作 Application 类加载器）了。它负责从 classpath 环境变量中加载某些应用相关的类，classpath 环境变量通常由 -classpath 或 -cp 命令行选项来定义，或者是 JAR 中的 Manifest 的 classpath 属性。Application 类加载器是 Extension 类加载器的子加载器 |

| 工作原理 | 说明|
|----------|:------|
| 委托机制 | 加载任务委托交给父类加载器，如果不行就向下传递委托任务，由其子类加载器加载，保证 java 核心库的安全性 |
| 可见性机制 | 子类加载器可以看到父类加载器加载的类，而反之则不行 |
| 单一性机制 | 父加载器加载过的类不能被子加载器加载第二次 |


> BootstrapClassLoader是C/C++实现，java中并不能直接访问，ClassLoader的初始化是在Zygote进程的实例化类ZygoteInit的入口方法main()中初始化的。

## 垃圾回收 gc
### 对象存活判断
- **引用计数**

每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。 

- **可达性分析**

从 GC Roots 开始向下搜索，搜索所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可用的。不可达对象。

> 在Java语言中，GC Roots包括：
> - 虚拟机栈中引用的对象。
> - 方法区中类静态属性实体引用的对象。
> - 方法区中常量引用的对象。
> - 本地方法栈中 JNI 引用的对象。

### 垃圾收集算法
- **标记 -清除算法**
  

“标记-清除”（Mark-Sweep）算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的。

它的主要缺点有两个：一个是效率问题，标记和清除过程的效率都不高；另外一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

- **复制算法**
  

“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

这样使得每次都是对其中的一块进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半，持续复制长生存期的对象则导致效率降低。

- **标记-整理算法**
  

复制收集算法在对象存活率较高时就要执行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

- **分代收集算法**

GC 分代的基本假设：绝大部分对象的生命周期都非常短暂，存活时间短。

“分代收集”（Generational Collection）算法，把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记-清理”或“标记-整理”算法来进行回收。

### 垃圾收集器
- **CMS收集器**
> CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的 Java 应用都集中在互联网站或B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。

从名字（包含“Mark Sweep”）上就可以看出CMS收集器是基于“标记-清除”算法实现的，它的运作过程相对于前面几种收集器来说要更复杂一些，整个过程分为4个步骤，包括：

- 初始标记（CMS initial mark）
- 并发标记（CMS concurrent mark）
- 重新标记（CMS remark）
- 并发清除（CMS concurrent sweep）

其中初始标记、重新标记这两个步骤仍然需要“Stop The World”。初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，并发标记阶段就是进行GC Roots Tracing的过程，而重新标记阶段则是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。

由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以总体上来说，CMS收集器的内存回收过程是与用户线程一起并发地执行。老年代收集器（新生代使用ParNew）

- **G1收集器**
  

与CMS收集器相比G1收集器有以下特点：

1、空间整合，G1收集器采用标记整理算法，不会产生内存空间碎片。分配大对象时不会因为无法找到连续空间而提前触发下一次GC。

2、可预测停顿，这是G1的另一大优势，降低停顿时间是G1和CMS的共同关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为N毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，这几乎已经是实时 Java（RTSJ）的垃圾收集器的特征了。

使用G1收集器时，Java堆的内存布局与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔阂了，它们都是一部分（可以不连续）Region 的集合。

G1的新生代收集跟 ParNew 类似，当新生代占用达到一定比例的时候，开始出发收集。和 CMS 类似，G1 收集器收集老年代对象会有短暂停顿。

### 内存模型与回收策略
![](https://mmbiz.qpic.cn/mmbiz_png/qdzZBE73hWsbhfAng9ibqfcbjrqgyRWqAKiaJ2U75SGYwQhs2tuNbXtu8KIpaUsBOaHRKXf7esuuFoMjELFxibIVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Java 堆（Java Heap）是JVM所管理的内存中最大的一块，堆又是垃圾收集器管理的主要区域，Java 堆主要分为2个区域-年轻代与老年代，其中年轻代又分 Eden 区和 Survivor 区，其中 Survivor 区又分 From 和 To 2个区。

- **Eden 区**
  

大多数情况下，对象会在新生代 Eden 区中进行分配，当 Eden 区没有足够空间进行分配时，虚拟机会发起一次 Minor GC，Minor GC 相比 Major GC 更频繁，回收速度也更快。
通过 Minor GC 之后，Eden 会被清空，Eden 区中绝大部分对象会被回收，而那些无需回收的存活对象，将会进到 Survivor 的 From 区（若 From 区不够，则直接进入 Old 区）。

- **Survivor 区**

Survivor 区相当于是 Eden 区和 Old 区的一个缓冲，类似于我们交通灯中的黄灯。Survivor 又分为2个区，一个是 From 区，一个是 To 区。每次执行 Minor GC，会将 Eden 区和 From 存活的对象放到 Survivor 的 To 区（如果 To 区不够，则直接进入 Old 区）。Survivor 的存在意义就是减少被送到老年代的对象，进而减少 Major GC 的发生。Survivor 的预筛选保证，只有经历第16次 Minor GC 还能在新生代中存活的对象，才会被送到老年代。
新生代在进行第一次GC操作的时候，会把Eden区存活的对象放到From或者To区，From和To只会选择一个，在进行以后的GC操作时，会把Eden区和From或To中选择的其中的一个区中的存活对象放到From或To的另一个区中区。

- **Old 区**
  

老年代占据着2/3的堆内存空间，只有在 Major GC 的时候才会进行清理，每次 GC 都会触发“Stop-The-World”。内存越大，STW 的时间也越长，所以内存也不仅仅是越大就越好。由于复制算法在对象存活率较高的老年代会进行很多次的复制操作，效率很低，所以老年代这里采用的是标记——整理算法。


# Object
## equals 方法
对两个对象的地址值进行的比较（即比较引用是否相同）
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

String类中重写了此方法，在判断对象不同后会判断内容是否相同

## hashCode 方法
hashCode() 方法给对象返回一个 hash code 值。这个方法被用于 hash tables，例如 HashMap。

它的性质是：
- 在一个Java应用的执行期间，如果一个对象提供给 equals 做比较的信息没有被修改的话，该对象多次调用 hashCode() 方法，该方法必须始终如一返回同一个 integer。

- 如果两个对象根据 equals(Object) 方法是相等的，那么调用二者各自的 hashCode() 方法必须产生同一个 integer 结果。

- 并不要求根据 equals(Object) 方法不相等的两个对象，调用二者各自的 hashCode() 方法必须产生不同的 integer 结果。然而，程序员应该意识到对于不同的对象产生不同的 integer 结果，有可能会提高 hash table 的性能。

在 JDK 中，Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法直接返回对象的 内存地址。在 String 类，重写了 hashCode 方法
```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

# static
- static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名去进行访问。
- 静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。
- 能通过 this 访问静态成员变量吗?
所有的静态方法和静态变量都可以通过对象访问（只要访问权限足够）。
- static是不允许用来修饰局部变量

# final
- 可以声明成员变量、方法、类以及本地变量
- final 成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误
- final 变量是只读的
- final 申明的方法不可以被子类的方法重写
- final 类通常功能是完整的，不能被继承
- final 变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销
- final 关键字提高了性能，JVM 和 Java 应用都会缓存 final 变量，会对方法、变量及类进行优化
- 方法的内部类访问方法中的局部变量，但必须用 final 修饰才能访问

> 内部类访问方法中的局部变量，实际是通过内部类的构造器传入的，传入的参数作为内部类的成员变量存在，如果不用final修饰，在方法中改变了局部变量的值，内部类是不知道的，有可能会导致运行结果不同，所以必须用final修饰。

# String、StringBuffer、StringBuilder
- String 是 final 类，不能被继承。对于已经存在的 Stirng 对象，修改它的值，就是重新创建一个对象
- StringBuffer 是一个类似于 String 的字符串缓冲区，使用 append() 方法修改 Stringbuffer 的值，使用 toString() 方法转换为字符串，是线程安全的
- StringBuilder 用来替代于 StringBuffer，StringBuilder 是非线程安全的，速度更快
- 字符串直接拼接，底层是使用StringBuilder的append来实现的

# 异常处理
- Exception、Error 是 Throwable 类的子类
- Error 类对象由 Java 虚拟机生成并抛出，不可捕捉  
- 不管有没有异常，finally 中的代码都会执行
- 当 try、catch 中有 return 时，finally 中的代码依然会继续执行

| 常见的Error | | |
|------|-----|-----|
| OutOfMemoryError | StackOverflowError | NoClassDeffoundError |

| 常见的Exception | | |
|------|-----|-----|
| 常见的非检查性异常 |  |
| ArithmeticException | ArrayIndexOutOfBoundsException | ClassCastException |
| IllegalArgumentException | IndexOutOfBoundsException | NullPointerException |
| NumberFormatException | SecurityException | UnsupportedOperationException |
| 常见的检查性异常 |  |
| IOException | CloneNotSupportedException | IllegalAccessException |
| NoSuchFieldException | NoSuchMethodException | FileNotFoundException |

# 内部类
- 非静态内部类没法在外部类的静态方法中实例化。
- 非静态内部类的方法可以直接访问外部类的所有数据，包括私有的数据。
- 在静态内部类中调用外部类成员，成员也要求用 static 修饰。
- 创建静态内部类的对象可以直接通过外部类调用静态内部类的构造器；创建非静态的内部类的对象必须先创建外部类的对象，通过外部类的对象调用内部类的构造器。

## 匿名内部类
- 匿名内部类不能定义任何静态成员、方法
- 匿名内部类中的方法不能是抽象的
- 匿名内部类必须实现接口或抽象父类的所有抽象方法
- 匿名内部类不能定义构造器
- 匿名内部类访问的外部类成员变量或成员方法必须用 final 修饰

# 多态
- 父类的引用可以指向子类的对象
- 创建子类对象时，调用的方法为子类重写的方法或者继承的方法
- 如果我们在子类中编写一个独有的方法，此时就不能通过父类的引用创建的子类对象来调用该方法
- 运行时多态：即重写，是指Java运行根据调用该方法的类型决定调用哪个方法。
- 设计时多态：即重载，是指Java允许方法名相同而参数不同（返回值可以相同也可以不相同）。

# 抽象和接口
- 抽象类不能有对象（不能用 new 关键字来创建抽象类的对象）
- 抽象类中的抽象方法必须在子类中被重写
- 接口中的所有属性默认为：public static final ****；
- 接口中的所有方法默认为：public abstract ****；

# 反射
```java
try {
    Class cls = Class.forName("com.jasonwu.Test");
    //获取构造方法
    Constructor[] publicConstructors = cls.getConstructors();
    //获取全部构造方法
    Constructor[] declaredConstructors = cls.getDeclaredConstructors();
    //获取公开方法
    Method[] methods = cls.getMethods();
    //获取全部方法
    Method[] declaredMethods = cls.getDeclaredMethods();
    //获取公开属性
    Field[] publicFields = cls.getFields();
    //获取全部属性
    Field[] declaredFields = cls.getDeclaredFields();
    Object clsObject = cls.newInstance();
    Method method = cls.getDeclaredMethod("getModule1Functionality");
    Object object = method.invoke(null);
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InstantiationException e) {
    e.printStackTrace();
} catch (NoSuchMethodException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

# 单例
## 饿汉式
```java
public class CustomManager {
    private Context mContext;
    private static final Object mLock = new Object();
    private static CustomManager mInstance;

    public static CustomManager getInstance(Context context) {
        synchronized (mLock) {
            if (mInstance == null) {
                mInstance = new CustomManager(context);
            }

            return mInstance;
        }
    }

    private CustomManager(Context context) {
        this.mContext = context.getApplicationContext();
    }
}
```
## 双重检查模式
```java
public class CustomManager {
    private Context mContext;
    private volatile static CustomManager mInstance;

    public static CustomManager getInstance(Context context) {
        // 避免非必要加锁
        if (mInstance == null) {
            synchronized (CustomManger.class) {
                if (mInstance == null) {
                    mInstacne = new CustomManager(context);
                }
            }
        }

        return mInstacne;
    }

    private CustomManager(Context context) {
        this.mContext = context.getApplicationContext();
    }
}
```
## 静态内部类模式
```java
public class CustomManager{
    private CustomManager(){}
 
    private static class CustomManagerHolder {
        private static final CustomManager INSTANCE = new CustomManager();
    }
 
    public static CustomManager getInstance() {
        return CustomManagerHolder.INSTANCE;
    } 
}
```
静态内部类的原理是：

当 SingleTon 第一次被加载时，并不需要去加载 SingleTonHoler，只有当 getInstance() 方法第一次被调用时，才会去初始化 INSTANCE，这种方法不仅能确保线程安全，也能保证单例的唯一性，同时也延迟了单例的实例化。getInstance 方法并没有多次去 new 对象，取的都是同一个 INSTANCE 对象。

虚拟机会保证一个类的 ``<clinit>()`` 方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的 ``<clinit>()`` 方法，其他线程都需要阻塞等待，直到活动线程执行 ``<clinit>()`` 方法完毕

缺点在于无法传递参数，如Context等

```
枚举实现
enum SingleInstance04 {
    INSTANCE;
}
```

```
破坏单例的方式
1、反射
2、序列化
3、克隆

解决方案如下：
1、防止反射
  定义一个全局变量，当第二次创建的时候抛出异常
2、防止克隆破坏
   重写clone(),直接返回单例对象
3、防止序列化破坏
  添加readResolve(),返回Object对象
```

# 引用类型
强引用 > 软引用 > 弱引用 

| 引用类型 | 说明 |
|------|:-----|
| StrongReference（强引用） | 当一个对象具有强引用，那么垃圾回收器是绝对不会回收和销毁它的，**非静态内部类会在其整个生命周期中持有对它外部类的强引用** |
| WeakReference （弱引用）| 在垃圾回收器运行的时候，如果对一个对象的所有引用都是弱引用的话，该对象会被回收 |
| SoftReference（软引用）| 如果一个对象只具有软引用，若内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，才会回收这些对象的内存|
| PhantomReference（虚引用） | 一个只被虚引用持有的对象可能会在任何时候被 GC 回收。虚引用对对象的生存周期完全没有影响，也无法通过虚引用来获取对象实例，仅仅能在对象被回收时，得到一个系统通知（只能通过是否被加入到 ReferenceQueue 来判断是否被GC，这也是唯一判断对象是否被 GC 的途径）。|

# 元注解
@Retention：保留的范围，可选值有三种。

| RetentionPolicy | 说明 |
|------|:-----|
| SOURCE | 注解将被编译器丢弃（该类型的注解信息只会保留在源码里，源码经过编译后，注解信息会被丢弃，不会保留在编译好的class文件里），如 @Override|
| CLASS | 注解在class文件中可用，但会被 VM 丢弃（该类型的注解信息会保留在源码里和 class 文件里，在执行的时候，不会加载到虚拟机中），请注意，当注解未定义 Retention 值时，默认值是 CLASS。|
| RUNTIME | 注解信息将在运行期 (JVM) 也保留，因此可以通过反射机制读取注解的信息（源码、class 文件和执行的时候都有注解的信息），如 @Deprecated|

@Target：可以用来修饰哪些程序元素，如 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER等，未标注则表示可修饰所有

@Inherited：是否可以被继承，默认为false  

@Documented：是否会保存到 Javadoc 文档中

