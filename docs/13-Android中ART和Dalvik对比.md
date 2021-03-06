Android4.4版本以前是Dalvik虚拟机，4.4版本开始引入ART虚拟机（Android Runtime）。在4.4版本上，两种运行时环境共存，可以相互切换，但是在5.0版本以后，Dalvik虚拟机则被彻底的丢弃，全部采用ART。

#### JVM/DVM

```
JVM本质上就是一个软件，是计算机硬件的一层软件抽象，在这之上才能够运行Java程序，JAVA在编译后会生成类似于汇编语言的JVM字节码，与C语言编译后产生的汇编语言不同的是，C编译成的汇编语言会直接在硬件上跑，但JAVA编译后生成的字节码是在JVM上跑，需要由JVM把字节码翻译成机器指令，才能使JAVA程序跑起来。
JVM运行在操作系统上，屏蔽了底层实现的差异，从而有了JAVA吹嘘的平台独立性和Write Once Run Anywhere。JVM的作用是把平台无关的.class里面的字节码翻译成平台相关的机器码，来实现跨平台。

JVM是Java Virtual Machine，而DVM就是 Dalvik Virtual Machine，是安卓中使用的虚拟机，所有安卓程序都运行在安卓系统进程里，每个进程对应着一个Dalvik虚拟机实例。他们都提供了对象生命周期管理、堆栈管理、线程管理、安全和异常管理以及垃圾回收等重要功能，各自拥有一套完整的指令系统。

JAVA虚拟机运行的是JAVA字节码，Dalvik虚拟机运行的是Dalvik字节码
Dalvik可执行文件体积更小
JVM基于栈开发，DVM是基于寄存器开发

JVM比DVM慢的原因：
	1、按需加载，导致加载不够实时。
	2、基于栈开发，对应的二进制指令更加复杂。

JVM：.java 经过 java compiler 编译成 .class 文件交由JVM运行
DVM：.java 经过 java compiler 编译成 .class 文件，然后经过 dex compiler 编译成 .dex 文件交由DVM运行
```

#### Dalvik

```
Dalvik 虚拟机采用的是JIT（Just-In-Time）编译模式，意思为即时编译，我们知道apk被安装到手机中时，对应目录会有dex或odex和apk文件，apk文件存储的是资源文件，而dex或odex（经过优化后的dex文件内部存储class文件）内部存储class文件，每次运行app时虚拟机会将dex文件解释翻译成机器码，这样才算是本地可执行代码，之后被系统运行。

Dalvik虚拟机可以看做是一个Java VM，他负责解释dex文件为机器码，如果我们不做处理的话，每次执行代码，都需要Dalvik将dex代码翻译为微处理器指令，然后交给系统处理，这样效率不高。为了解决这个问题，Google在2.2版本添加了JIT编译器，当App运行时，每当遇到一个新类，JIT编译器就会对这个类进行编译，经过编译后的代码，会被优化成相当精简的原生型指令码（即native code），这样在下次执行到相同逻辑的时候，速度就会更快。

因为65535这个问题，导致我们在应用冷启动的时候有一个合包的过程，最后导致的一个结果就是我们的app启动慢。
```

#### ART

```
ART 是一种执行效率更高且更省电的运行机制，执行的是本地机器码，这些本地机器码是从dex字节码转换而来。ART采用的是AOT（Ahead-Of-Time）编译，应用在第一次安装的时候，字节码就会预先编译成机器码存储在本地。在App运行时，ART模式就较Dalvik模式少了解释字节码的过程，所以App的运行效率会有所提高，占用内存也会相应减少。谷歌在5.0以后的Android版本中默认了ART模式启动，就是希望Android能摆脱卡顿这个毛病。

Android Runtime
```

##### JIT/AOT

```
JIT（Just In Time，即时编译技术）
	运行时进行字节码到本地机器码的编译
	优点：
		安装速度快，不需要将字节码编译成机器码
		不需要占用额外的存储空间
	缺点：
		每次启动应用都需要重新编译
		运行时比较耗电（因为经常有编译的过程）
		
AOT(Ahead Of Time，预编译技术)
	在应用安装时就将字节码编译成本地机器码
	优点：
		启动和运行速度快，直接加载机器码运行
		比较省电，不需要运行时再编译成机器码
	缺点：
		应用安装和系统升级之后的应用优化比较耗时（重新编译，把程序代码转换成机器语言）
		优化后的文件会占用额外的存储空间（缓存转换结果，用内存换时间）
```

#### ART/Dalvik对比

```
Dalvik
Android4.4及以前使用的都是Dalvik虚拟机，我们知道Apk在打包的过程中会先将java等源码通过javac编译成.class文件，但是我们的Dalvik虚拟机只会执行.dex文件，这个时候dex会将.class文件转换成Dalvik虚拟机执行的.dex文件。Dalvik虚拟机在启动的时候会先将.dex文件转换成快速运行的机器码，又因为65535这个问题，导致我们在应用冷启动的时候有一个合包的过程，最后导致的一个结果就是我们的app启动慢，这就是Dalvik虚拟机的JIT特性（Just In Time）。
ART
ART虚拟机是在Android5.0才开始使用的Android虚拟机，ART虚拟机必须要兼容Dalvik虚拟机的特性，但是ART有一个很好的特性AOT（ahead of time），这个特性就是我们在安装APK的时候就将dex直接处理成可直接供ART虚拟机使用的机器码，ART虚拟机将.dex文件转换成可直接运行的.oat文件，ART虚拟机天生支持多dex，所以也不会有一个合包的过程，所以ART虚拟机会很大的提升APP冷启动速度。

两者的区别
	1、Dalvik每次都要编译再运行，Art只会安装时启动编译
	2、Art占用空间比Dalvik大（原生代码占用的存储空间更大），就是用“空间换时间”
	3、Art减少编译，减少了CPU使用频率，使用明显改善电池续航
	4、Art应用启动更快、运行更快、体验更流畅、触感反馈更及时

ART优点：
①系统性能显著提升
②应用启动更快、运行更快、体验更流畅、触感反馈更及时
③续航能力提升
④支持更低的硬件

ART缺点
①更大的存储空间占用，可能增加10%-20%
②更长的应用安装时间
```

>Android 为什么要使用虚拟机？
>
>Android 使用虚拟机作为其运行环境是为了运行 APK 文件构成的 Android 应用。它的优点有：
>
>- 应用代码和核心的操作系统分离。所以即使任意一个程序中包含恶意的代码也不会直接影响系统文件。这使得 Android 操作系统更稳定可靠。
>- 提高了跨平台兼容性或者说平台独立性。这意味着即使某一个应用是在 PC 上编译的，它也可以通过虚拟机在移动平台上执行。
>- 每个应用都有有一个独立的进程，系统会为每个应用开启一个虚拟机，即使当前app出现崩溃，也不会影响系统和其他app的运行

[推荐阅读](https://juejin.cn/post/6875482712078024711?utm_source=gold_browser_extension%3Futm_source%3Dgold_browser_extension)