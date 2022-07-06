#### Glide 

[推荐](https://blog.csdn.net/guolin_blog/article/details/53759439?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-4&spm=1001.2101.3001.4242)

```
缓存策略
内存缓存：
	1、正在使用的图片缓存放到弱引用的一个map对象里面缓存
	2、不使用的图片缓存使用LRU算法实现LinkedHashMap里面

	活动缓存和LRU缓存共同组成了内存缓存
	
		加载图片时如果从LRU缓存中获取到了图片会把图片从LRU缓存中移除，然后加入到弱引用的缓存中，保护这些图片不会被LruCache算法回收掉
		EngineResource是用一个acquired变量用来记录图片被引用的次数，调用acquire()方法会让变量加1，调用release()方法会让变量减1，当acquired变量大于0的时候，说明图片正在使用中，也就应该放到activeResources弱引用缓存当中。而经过release()之后，如果acquired变量等于0了，说明图片已经不再被使用了，首先会将缓存图片从activeResources中移除，然后再将它put到LruResourceCache当中。这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用LruCache来进行缓存的功能。
		
	流程：加载活动缓存 - 活动缓存存在 - 返回图片 - 活动缓存不存在 - 加载内存缓存 - 内存缓存存在 - remove内存缓存 - 将内存缓存添加到活动缓存 - 返回remove内存缓存
	活动缓存引用计数大于0 - 表示被使用 - 活动缓存引用计数小于等于0 - 删除活动缓存资源 - 将活动缓存资源添加进内存缓存
		
硬盘缓存：
	DiskCacheStrategy.NONE： 表示不缓存任何内容。
	DiskCacheStrategy.DATA： 表示只缓存原始图片。
	DiskCacheStrategy.RESOURCE： 表示只缓存转换过后的图片。
	DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片。
	DiskCacheStrategy.AUTOMATIC： 表示让Glide根据图片资源智能地选择使用哪一种缓存策略（默认选项）。
	
	硬盘缓存的实现也是使用的LruCache算法
	
	1、资源缓存是在把图片转换完之后才缓存；
	2、原始数据是网络请求成功之后就写入缓存；
	流程：加载磁盘缓存 - 资源磁盘缓存存在 - 返回资源图片 - 资源磁盘缓存不存在 - 加载原始数据存在 - 解码转换返回 - 加载原始数据不存在 - 创建request下载图片
	
如果图片的配置信息发生了变化，Glide都会为其存储一份缓存，包括原图

内存缓存的主要作用是防止应用重复将图片数据读取到内存当中，而硬盘缓存的主要作用是防止应用重复从网络或其他地方重复下载和读取数据。

如何监听生命周期变化？
	内部实现一个Fragment，监听Fragment的生命周期变化来监听处理图片的加载和中断销毁逻辑


工厂模式创建的Target

key的生成：生成EngineKey对象，通过重写equals() 和 hashCode() 方法来保证传入的数据生成的key值是唯一的，并且只有所有的入参都一样的情况才是同一个key，Glide会为每一种修改都生成一个key来缓存图片
原始图片的key：getOriginalKey()方法，只使用了id和signature这两个参数来构成缓存Key，原始图片是没有做过修改的，因此不需要那么多参数设置

```

##### LruCache

```
内部使用LinkHashMap来实现，通过new LinkedHashMap<K, V>(0, 0.75f, true)设置accessOrder=true（第三个参数）
LinkHashMap=双向链表+HashMap
使用HashMap来存储数据，使用双向链表来保证数据的有序性
accessOrder=true表示每次get数据的时候都会把当前数据移动到链表的尾部，表示最近访问的对象

put 操作时，会将数据插入到链表的尾部，然后判断数据是否超过负荷，如果超过负荷，会把链表头部的数据删除，因为头部的数据是最先插入的数据，并且最久未访问的数据
```

> 正在使用的图片使用活动内存缓存，使用的弱引用，是在内存不足的时候可以进行回收
> 
> 不使用的图片使用LRU缓存，是避免正在使用的缓存被LRU算法给清除掉
> 
> 为什么不使用的图片不直接销毁掉？是因为刚使用的图片有可能会再次使用，如果直接清除，就会需要重新去将图片放到内存中（猜的）

> DiskLRUCache：会对每一个缓存文件的操作做记录，放到journal文件中，在调用open方法时，会将操作记录加载到LinkedHashMap中
> 
> DiskLRUCache用journal文件来管理所有的操作记录
> 
> 避免操作记录过多，使用redundantOpCount（最大为2000）字段来确定操作次数，超过最大值时会删除掉最久未操作的记录，保证journal文件不会过大，在每次completeEdit之后会判断是否超过限制，如果超过就会从LinkedHashMap中删除队头（队头是最先插入的数据，最久未访问的数据）的值，同时删除对应的文件