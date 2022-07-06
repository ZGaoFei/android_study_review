## kotlin

[协程](协程.md)

### 扩展函数

    fun receiveType.functionName(params) {
        body
    }
    receiveType：表示扩展函数的接收者，也就是扩展函数的对象
    functionName：扩展函数名称
    params：扩展函数的参数

    例如：
    Class User(var name : String)

    fun User.print(val name: String) {
        print(name)
    }

### 标准库的扩展函数

> let 

    表示以this值作为参数调用指定的函数[block]并返回结果。
    例如：
        fun letTest() {
            val string = "hello world";
            val let = string.let {
                val startsWith = it.startsWith("h")
                it.length
            }
        }
    调用T类型对象的let函数，则该对象为该函数的参数。用it表示当前对象，调用该对象的方法，最后一行为返回值。
    也可以使用 ?. 表示当前对象不为空时调用

    使用场景：
    1、处理一个可为null的对象，统一做判空处理
    if(a != null) {
        todo
    }
    ==
    a?.let{
        todo
    }
    2、需要明确一个变量所处的特定作用域范围内可使用

> with

    一个非拓展函数，上下文对象作为参数传递，但是在lambda内部，它作为[receiver] (this)可用，返回值是lambda结果。当你需要的一个对象在一个特定的作用域范围内多次使用到其方法时，可以省去对象名，直接访问对象的公有属性和方法。

    将当前对象作为第一个参数传递进来，第二个参数是一个方法体（lambda表达式）

    例如：
    class User(val name: String, val age: Int) {
        var height = 0
        var sex = "男"

        fun printWith() {
            print("name: $name, age: $age, height: $height, sex: $sex")
        }
    }
    fun withTest() {
        val withClass = WithClass("name", 18)
        val with = with(withClass, {
            printWith()
            print("name: $name, age: $age, height: $height, sex: $sex")
            name.length
        })
    }

    使用场景：
    适用于同一个对象的多个方法时，可以省去类名重复，直接调用类的方法即可。
    1、建议在不提供lambda结果的情况下调用上下文对象上的函数，在代码中，with可以理解为“使用这个对象，执行以下的操作”。
    2、with()函数的另一个用例是引入一个helper对象，它的属性或者函数将用于计算值。

> run

    适用于let函数和with函数的任何场景。run函数其实就是 let和with两个函数的结合体，准确来说它弥补了let函数在函数内必须适用it参数替代对象；另一方面它弥补了with函数传入对象判空问题。所以run函数可以像with函数那样省略对象参数直接访问对象的公有属性和方法，同时像let函数那样对对象做空判断处理。

    例如：
    fun runTest() {
        val runClass = User("name", 18)
        val run = runClass.run {
            printWith()
            print("name: $name, age: $age, height: $height, sex: $sex")
            name.length
        }

        runClass?.run {

        }
    }

    是let和with的结合体

> also

    上下文对象可以作为参数it使用，返回值是对象本身。也适用于执行一些将上下文作为参数对象的操作，也可用于需要引用对象而不是引用对象的属性和函数的操作，或者当你不想从外部作用域隐藏该引用时。你可以理解为“并对该对象执行以下操作”。

    例如：
    fun alsoTest() {
        val alsoClass = User("name", 18)
        val also = alsoClass?.also {
            it.printWith()
            print("name: $it.name, age: $it.age, height: $it.height, sex: $it.sex")
            it.name.length
        }
        also.printWith()
    }

    适用于let函数的任何场景，与let函数不同的是let函数以闭包的形式返回函数块最后一行的值，如果最后一行值为空则返回一个Unit类型的默认值，而also函数返回的是传入对象本身。同时可以对传入的对象进行操作，一般用于多个拓展函数的链式调用。

    例如：
    val list = mutableListOf("one", "two", "three")
    list.also {
        for (i in it.indices) {
            Log.e(TAG, "apply == element：" + it[i])
        }
    }.add("four")

> apply

    上下文对象可用作接收者(this)，返回值是对象本身。对于没有返回值并且主要对接收方对象的成员进行操作的代码块使用apply，apply函数的常见情况是对象配置，这样的调用可以理解为“对对象的应用以下赋值”。

    例如：
    fun applyTest() {
        val applyClass = User("name", 18)
        val apply = applyClass?.apply {
            printWith()
            print("name: $name, age: $age, height: $height, sex: $sex")
            name.length
        }
        apply.printWith()
    }

    apply函数整体上和run函数相似，唯一不同就是它的返回值是对象本身。apply函数一般用于对象实例初始化的时候，需要对对象中的属性进行赋值；或者动态inflate一个View的时候需要给View绑定数据。
    例如：
    mHeadView = View.inflate(activity, R.layout.head_task_view, null).apply {
        tv_name?.text = "姓名XXX"
        tv_age?.text = "20"
        tv_sex?.text = "女"
        tv_name?.setOnClickListener(this@KExampleActivity)
    }

    apply函数通过将接收方[receiver] this作为返回值，可以轻松地将apply包含到调用链中，以便进行更复杂的处理。


| 函数 | 函数块对象引用 | 返回值 | 是否拓展函数 | 使用场景 |
|--|--|--|--|--|
| let |	it | Lambda表达式结果 |	是 | 1.适用于处理不为null的操作场景；2.明确一个变量所处的特定作用域范围内可使用。|
| with | this | Lambda表达式结果 | 否(上下文对象作为参数) | 适用于同一个对象的公有属性和函数调用。|
| run | this/无 | Lambda表达式结果 | 是/否(调用时没有上下文对象) | 适用于let函数和with函数的任何场景。对对象中的属性进行赋值和计算结果；或者在需要表达式的地方运行语句。
| apply | this | 返回this(对象本身) | 是 | 1.一般用于对象实例初始化的时候，需要对对象中的属性进行赋值；2.动态inflate一个View的时候需要给View绑定数据。|
| also | it | 返回this(对象本身) | 是 | 适用于let函数的任何场景，对传入的对象进行操作，一般用于多个拓展函数的链式调用。|


> takeIf 和 takeUnless

    在提供某条件的对象上调用，如果与某条件匹配，则takeIf返回该对象，否则它返回null，takeIf是针对单个对象的过滤函数。反过来，如果不匹配某条件，则takeUnless返回对象，如果匹配则返回null，对象可以作为lambda参数（it）使用。

    例如：
    fun takeTest() {
        val takeClass = User("name", 18)
        val takeIf = takeClass.takeIf {
            it.name.length > 10
        }
        
        takeClass.takeUnless { 
            it.name.length > 10
        }
    }

    takeIf和takeUnless函数与作用域函数一起使用特别有用，当在takeIf和takeUnless之后链接其他函数，必须执行空检查或者安全调用(?.)，因为它们的返回值可能为空（null）。

    例如：

    private fun synForKotlin(str: String) {
        str.takeIf { !it.isNullOrEmpty() }?.let {
            Log.e(TAG, "syn == result：" + it.toUpperCase())
        }
    }
    //调用
    synForKotlin("HelloWord")
    synForKotlin("")

    takeIf表示如果满足给定条件，则返回this值(对象本身)，如果不满足则返回null。takeUnless表示如果不满足给定条件，则返回this值(对象本身)，如果满足则返回null。两个正好相反。

> repeat

    第一个参数为重复的次数，第二个参数为一个代码块，即重复执行第二个参数里面的代码次数为第一个参数
    
    例如：
    fun repeatTest() {
        repeat(5) {
            print(it)
        }
    }


> LAZY

    Lazy 只能修饰 val 不可变参数，等同于 java 的 final ，其次 Lazy 后跟一个 {} 复制表达式，本质是在一个工厂函数，只有在第一次调用时生效(既首次创建对象时)，常用于单例模式
    默认的线程类型为LazyThreadSafetyMode.SYNCHRONIZED，即会进行双重检查锁，是线程安全的

    fun lazyTest1(): String {
        val string: String by lazy {
            "hello world"
        }
        return string
    }

    fun lazyTest2() {
        val string: String by lazy {
            ->
            ""
        }
    }

    fun lazyTest3() {
        val string: String by lazy("hello") {
            "hello world"
        }
    }

    // 指定线程类型
    fun lazyTest4() {
        val string: String by lazy(LazyThreadSafetyMode.NONE) {
            "hello world"
        }
    }


> lateinit 

    只能用来修饰类属性，不能用来修饰局部变量，
    只能用来修饰对象，不能用来修饰基本类型(因为基本类型的属性在类加载后的准备阶段都会被初始化为默认值)。
    只能修饰var

    lateinit var str: String
    fun lateInitTest() {
        // 判断是否初始化
        if (::str.isLateinit) {
            print(str)
        }
        str = "hello world"
        print(str)
    }


