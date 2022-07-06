[so库基础知识](https://blog.csdn.net/qq_38998213/article/details/105250051)

[Android调用so库全过程](https://blog.csdn.net/liujian8654562/article/details/78717149)

# NDK 开发

> NDK 全称是 Native Development Kit，是一组可以让你在 Android 应用中编写实现 C/C++ 的工具，可以在项目用自己写源代码构建，也可以利用现有的预构建库。

使用 NDK 的使用目的有：
- 从设备获取更好的性能以用于计算密集型应用，例如游戏或物理模拟  
- 重复使用自己或其他开发者的 C/C++ 库，便利于跨平台。  
- NDK 集成了譬如 OpenSL、Vulkan 等 API 规范的特定实现，以实现在 java 层无法做到的功能如提升音频性能等  
- 增加反编译难度

## JNI 基础
### 数据类型
- 基本数据类型
  

| Java 类型 | Native 类型 | 符号属性 | 字长 |
|--|--|--|-- |
| boolean | jboolean | 无符号 | 8位 |
| byte | jbyte | 无符号 | 8位 |
| char | jchar | 无符号 | 16位 |
| short | jshort | 有符号 | 16位 |
| int | jnit | 有符号 | 32位 |
| long | jlong | 有符号 | 64位 |
| float | jfloat | 有符号 | 32位 |
| double | jdouble | 有符号 | 64位 |

- 引用数据类型

| Java 引用类型	| Native 类型 | Java 引用类型 | Native 类型 |
|--|--|--|-- |
| All objects | jobject | char[] | jcharArray |
| java.lang.Class | jclass | short[] | jshortArray |
| java.lang.String | jstring | int[] | jintArray |
| Object[] | jobjectArray | long[] | jlongArray |
| boolean[] | jbooleanArray | float[] | jfloatArray |
| byte[] | jbyteArray | double[] | jdoubleArray |
| java.lang.Throwable | jthrowable	 |

### String 字符串函数操作
| JNI 函数 | 描述 |
|--|-- |
| GetStringChars / ReleaseStringChars | 获得或释放一个指向 Unicode 编码的字符串的指针（指 C/C++ 字符串） |
| GetStringUTFChars / ReleaseStringUTFChars | 获得或释放一个指向 UTF-8 编码的字符串的指针（指 C/C++ 字符串） |
| GetStringLength | 返回 Unicode 编码的字符串的长度 |
| getStringUTFLength | 返回 UTF-8 编码的字符串的长度 |
| NewString | 将 Unicode 编码的 C/C++ 字符串转换为 Java 字符串 |
| NewStringUTF | 将 UTF-8 编码的 C/C++ 字符串转换为 Java 字符串 |
| GetStringCritical / ReleaseStringCritical | 获得或释放一个指向字符串内容的指针(指 Java 字符串) |
| GetStringRegion | 获取或者设置 Unicode 编码的字符串的指定范围的内容 |
| GetStringUTFRegion | 获取或者设置 UTF-8 编码的字符串的指定范围的内容 |

### 常用 JNI 访问 Java 对象方法
``MyJob.java``
```java
package com.example.myjniproject;

public class MyJob {

    public static String JOB_STRING = "my_job";
    private int jobId;

    public MyJob(int jobId) {
        this.jobId = jobId;
    }

    public int getJobId() {
        return jobId;
    }
}
```
``native-lib.cpp``
```c++
#include <jni.h>

extern "C"
JNIEXPORT jint JNICALL
Java_com_example_myjniproject_MainActivity_getJobId(JNIEnv *env, jobject thiz, jobject job) {

    // 根据实力获取 class 对象
    jclass jobClz = env->GetObjectClass(job);
    // 根据类名获取 class 对象
    jclass jobClz = env->FindClass("com/example/myjniproject/MyJob");

    // 获取属性 id
    jfieldID fieldId = env->GetFieldID(jobClz, "jobId", "I");
    // 获取静态属性 id
    jfieldID sFieldId = env->GetStaticFieldID(jobClz, "JOB_STRING", "Ljava/lang/String;");

    // 获取方法 id
    jmethodID methodId = env->GetMethodID(jobClz, "getJobId", "()I");
    // 获取构造方法 id
    jmethodID  initMethodId = env->GetMethodID(jobClz, "<init>", "(I)V");

    // 根据对象属性 id 获取该属性值
    jint id = env->GetIntField(job, fieldId);
    // 根据对象方法 id 调用该方法
    jint id = env->CallIntMethod(job, methodId);

    // 创建新的对象
    jobject newJob = env->NewObject(jobClz, initMethodId, 10);

    return id;
}
```

## NDK 开发
### 基础开发流程
- 在 java 中声明 native 方法
```java
public class MainActivity extends AppCompatActivity {

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Log.d("MainActivity", stringFromJNI());
    }

    private native String stringFromJNI();
}
```

- 在 ``app/src/main`` 目录下新建 cpp 目录，新建相关 cpp 文件，实现相关方法（AS 可用快捷键快速生成）

``native-lib.cpp``
```
#include <jni.h>

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_myjniproject_MainActivity_stringFromJNI(
        JNIEnv *env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

>- 函数名的格式遵循如下规则：Java_包名_类名_方法名。
>- extern "C" 指定采用 C 语言的命名风格来编译，否则由于 C 与 C++ 风格不同，导致链接时无法找到具体的函数
>- JNIEnv*：表示一个指向 JNI 环境的指针，可以通过他来访问 JNI 提供的接口方法
>- jobject：表示 java 对象中的 this
>- JNIEXPORT 和 JNICALL：JNI 所定义的宏，可以在 jni.h 头文件中查找到

- 通过 CMake 或者 ndk-build 构建动态库

### System.loadLibrary()
``java/lang/System.java``:
```java
@CallerSensitive
public static void load(String filename) {
    Runtime.getRuntime().load0(Reflection.getCallerClass(), filename);
}
```

- 调用 ``Runtime`` 相关 native 方法

``java/lang/Runtime.java``:
```java
private static native String nativeLoad(String filename, ClassLoader loader, Class<?> caller);
```

- native 方法的实现如下：

``dalvik/vm/native/java_lang_Runtime.cpp``:
```cpp
static void Dalvik_java_lang_Runtime_nativeLoad(const u4* args,
    JValue* pResult)
{
    ···
    bool success;

    assert(fileNameObj != NULL);
    // 将 Java 的 library path String 转换到 native 的 String
    fileName = dvmCreateCstrFromString(fileNameObj);

    success = dvmLoadNativeCode(fileName, classLoader, &reason);
    if (!success) {
        const char* msg = (reason != NULL) ? reason : "unknown failure";
        result = dvmCreateStringFromCstr(msg);
        dvmReleaseTrackedAlloc((Object*) result, NULL);
    }
    ···
}
```

- ``dvmLoadNativeCode`` 函数实现如下：

``dalvik/vm/Native.cpp``
```cpp
bool dvmLoadNativeCode(const char* pathName, Object* classLoader,
        char** detail)
{
    SharedLib* pEntry;
    void* handle;
    ···
    *detail = NULL;

    // 如果已经加载过了，则直接返回 true
    pEntry = findSharedLibEntry(pathName);
    if (pEntry != NULL) {
        if (pEntry->classLoader != classLoader) {
            ···
            return false;
        }
        ···
        if (!checkOnLoadResult(pEntry))
            return false;
        return true;
    }

    Thread* self = dvmThreadSelf();
    ThreadStatus oldStatus = dvmChangeStatus(self, THREAD_VMWAIT);
    // 把.so mmap 到进程空间，并把 func 等相关信息填充到 soinfo 中
    handle = dlopen(pathName, RTLD_LAZY);
    dvmChangeStatus(self, oldStatus);
    ···
    // 创建一个新的 entry
    SharedLib* pNewEntry;
    pNewEntry = (SharedLib*) calloc(1, sizeof(SharedLib));
    pNewEntry->pathName = strdup(pathName);
    pNewEntry->handle = handle;
    pNewEntry->classLoader = classLoader;
    dvmInitMutex(&pNewEntry->onLoadLock);
    pthread_cond_init(&pNewEntry->onLoadCond, NULL);
    pNewEntry->onLoadThreadId = self->threadId;

    // 尝试添加到列表中
    SharedLib* pActualEntry = addSharedLibEntry(pNewEntry);

    if (pNewEntry != pActualEntry) {
        ···
        freeSharedLibEntry(pNewEntry);
        return checkOnLoadResult(pActualEntry);
    } else {
        ···
        bool result = true;
        void* vonLoad;
        int version;
        // 调用该 so 库的 JNI_OnLoad 方法
        vonLoad = dlsym(handle, "JNI_OnLoad");
        if (vonLoad == NULL) {
            ···
        } else {
            // 调用 JNI_Onload 方法，重写类加载器。
            OnLoadFunc func = (OnLoadFunc)vonLoad;
            Object* prevOverride = self->classLoaderOverride;

            self->classLoaderOverride = classLoader;
            oldStatus = dvmChangeStatus(self, THREAD_NATIVE);
            ···
            version = (*func)(gDvmJni.jniVm, NULL);
            dvmChangeStatus(self, oldStatus);
            self->classLoaderOverride = prevOverride;

            if (version != JNI_VERSION_1_2 && version != JNI_VERSION_1_4 &&
                version != JNI_VERSION_1_6)
            {
                ···
                result = false;
            } else {
                ···
            }
        }

        if (result)
            pNewEntry->onLoadResult = kOnLoadOkay;
        else
            pNewEntry->onLoadResult = kOnLoadFailed;

        pNewEntry->onLoadThreadId = 0;

        // 释放锁资源 
        dvmLockMutex(&pNewEntry->onLoadLock);
        pthread_cond_broadcast(&pNewEntry->onLoadCond);
        dvmUnlockMutex(&pNewEntry->onLoadLock);
        return result;
    }
}
```

<!-- ### native 方法调用原理

- 虚拟机调用一个方法时，发现如果这是一个 native 方法，则使用 Method 对象中的nativeFunc 函数指针对象调用。

``dalvik2/vm/interp/Stack.cpp``:
```cpp
Object* dvmInvokeMethod(Object* obj, const Method* method,
    ArrayObject* argList, ArrayObject* params, ClassObject* returnType,
    bool noAccessCheck)
{
    ···
    if (dvmIsNativeMethod(method)) {
        TRACE_METHOD_ENTER(self, method);
        (*method->nativeFunc)((u4*)self->interpSave.curFrame, &retval, method, self);
        TRACE_METHOD_EXIT(self, method);
    } else {
        dvmInterpret(self, method, &retval);
    }
    ···
}
​``` -->

## CMake 构建 NDK 项目
> CMake 是一个开源的跨平台工具系列，旨在构建，测试和打包软件，从 Android Studio 2.2 开始，Android Sudio 默认地使用 CMake 与 Gradle 搭配使用来构建原生库。

启动方式只需要在 ``app/build.gradle`` 中添加相关：
​```groovy
android {
    ···
    defaultConfig {
        ···
        externalNativeBuild {
            cmake {
                cppFlags ""
            }
        }

        ndk {
            abiFilters 'arm64-v8a', 'armeabi-v7a'
        }
    }
    ···
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}
```

然后在对应目录新建一个 ``CMakeLists.txt`` 文件：
```txt
# 定义了所需 CMake 的最低版本
cmake_minimum_required(VERSION 3.4.1)

# add_library() 命令用来添加库
# native-lib 对应着生成的库的名字
# SHARED 代表为分享库
# src/main/cpp/native-lib.cpp 则是指明了源文件的路径。
add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        src/main/cpp/native-lib.cpp)

# find_library 命令添加到 CMake 构建脚本中以定位 NDK 库，并将其路径存储为一个变量。
# 可以使用此变量在构建脚本的其他部分引用 NDK 库
find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

# 预构建的 NDK 库已经存在于 Android 平台上，因此，无需再构建或将其打包到 APK 中。
# 由于 NDK 库已经是 CMake 搜索路径的一部分，只需要向 CMake 提供希望使用的库的名称，并将其关联到自己的原生库中

# 要将预构建库关联到自己的原生库
target_link_libraries( # Specifies the target library.
        native-lib

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
···
```
- [CMake 命令详细信息文档](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)

## 常用的 Android NDK 原生 API
| 支持 NDK 的 API 级别 | 关键原生 API | 包括  |
|--|--|-- |
| 3 | Java 原生接口 | 	#include <jni.h> |
| 3 | Android 日志记录 API	| #include <android/log.h> |
| 5 | OpenGL ES 2.0 | #include <GLES2/gl2.h><br>#include <GLES2/gl2ext.h> |
| 8 | Android 位图 API | #include <android/bitmap.h> |
| 9 | OpenSL ES | #include <SLES/OpenSLES.h><br>#include <SLES/OpenSLES_Platform.h><br>#include <SLES/OpenSLES_Android.h><br>#include <SLES/OpenSLES_AndroidConfiguration.h> |
| 9 | 原生应用 API | #include <android/rect.h><br>#include <android/window.h><br>#include<android/native_activity.h><br>··· |
| 18 | OpenGL ES 3.0 | #include <GLES3/gl3.h><br>#include <GLES3/gl3ext.h> |
| 21 | 原生媒体 API | #include <media/NdkMediaCodec.h><br>#include <media/NdkMediaCrypto.h><br>··· |
| 24 | 原生相机 API | #include <camera/NdkCameraCaptureSession.h><br>#include <camera/NdkCameraDevice.h><br>··· |