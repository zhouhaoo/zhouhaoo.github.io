---
layout:     post
title:      "ClassLoader类型"
date:       2021-03-16
author:     "zhouhaoh"
catalog: true
tags:
- Android
- framework
---

ClassLoader在android中用来加载dex文件，而java中用来加载jar包和.class文件。android中的classLoader主要有三种：BootClassLoader、PathClassLoader和DexClassLoader。



### 双亲委托模式
其中涉及到双亲委派模式：判断该Class是否已经加载，先从父类开始查找缓存，找不到就向下查找。模式的好处：

1. 避免重复加载。
2. 安全，避免自定义加载替换掉系统的类。

### 主要类型

![1](\img\ClassLoader\1.png)

#### BootClassLoader

BootClassLoader是ClassLoader的内部类，并继承自ClassLoader。BootClassLoader是一个单例类，需要注意的是BootClassLoader的访问修饰符是默认的，只有在同一个包中才可以访问，因此我们在应用程序中是无法直接调用的。

```java
class BootClassLoader extends ClassLoader {
    private static BootClassLoader instance;
    @FindBugsSuppressWarnings("DP_CREATE_CLASSLOADER_INSIDE_DO_PRIVILEGED")
    public static synchronized BootClassLoader getInstance() {
        if (instance == null) {
            instance = new BootClassLoader();
        }
        return instance;
    }
...
}
```

#### PathClassLoader

PathClassLoader来加载系统类和应用程序的类，在**ZygoteInit**中SystemServer中createPathClassLoader初始化

#### DexClassLoader

加载dex文件以及包含dex的压缩文件（apk和jar文件），DexClassLoader的构造方法有四个参数：

- dexPath：dex相关文件路径集合，多个路径用文件分隔符分隔，默认文件分隔符为‘：’
- optimizedDirectory：解压的dex文件存储路径，这个路径必须是一个内部存储路径，一般情况下使用当前应用程序的私有路径：`/data/data/<Package Name>/...`。
- librarySearchPath：包含 C/C++ 库的路径集合，多个路径用文件分隔符分隔分割，可以为null。
- parent：父加载器。

DexClassLoader 继承自BaseDexClassLoader ，方法实现都在BaseDexClassLoader中。

```java
public class DexClassLoader extends BaseDexClassLoader {
    public DexClassLoader(String dexPath, String optimizedDirectory,
            String librarySearchPath, ClassLoader parent) {
        super(dexPath, new File(optimizedDirectory), librarySearchPath, parent);
	    }
}
```

