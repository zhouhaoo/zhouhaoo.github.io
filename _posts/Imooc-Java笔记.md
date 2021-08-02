---
layout:     post
title:      "笔记"
subtitle:   "thread"
date:       2021-08-02
author:     "zhouhaoh"
tags:
    - JAVA
---

imooc课程笔记记录

### JAVA

#### 匿名内部类

没有确定名字的类，但是名字是外部类+$N.N表示第几个内部类。可以用到反射匿名内部类。

```java
public class OuterClass {
    static InnerClass innerClass = new InnerClass() {
        @Override
        void a() {
            System.out.println("hjahaha");
        }
    };

    public static void main(String[] args) {
        try {
            Class<?> aClass = Class.forName("com.zhouhaoh.java.learn.inner.OuterClass$1");
            System.out.println(aClass);
            Method a = aClass.getDeclaredMethod("a");
            a.setAccessible(true);
            a.invoke(innerClass);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

匿名内部类的构造方法

编译器会给内部类加入构造方法，静态和非静态有区别。

静态的（static和interface）不会引用外部的实例，但是构造方法会引用。

参数列表包括：

1. 外部对象
2. 父类的外部对象（非静态）
3. 父类的构造方法对象
4. 外部捕获的变量（方法体内有引入外部final变量）

Lambda转换（SAM类型）Single abstract Method 单一抽象方法

只有接口的有唯一的方法可以转换为lambda表达式。

#### 方法分派

- 方法分派的：调用谁的和哪个方法。

```java
class SuperClass {
    public String getName() {
        return "super";
    }
}

class SubClass extends SuperClass {
    @Override
    public String getName() {
        return "sub";
    }
}

public class Test {

    @Test
    public void test(){
        SuperClass superClass = new SubClass();
            //会执行该方法，重载，静态分派
           //声明类型SuperClass所以调用 printName(SuperClass superClass)
        printName(superClass); //sub
    }
    private void printName(SuperClass superClass){
        //getName被重写，动态分派，根据实际的类型
        System.out.println(superClass.getName());//sub
    }

    private void printName(SubClass subClass){
        System.out.println(subClass.getName());
    }

}
```

##### 静态-方法重载
 编译期确定，根据调用者的声明类型和方法参数类型,静态分派。

##### 动态-方法重写

运行时确定，依据调用者的实际类型分派，动态分派。

#### 泛型实现机制

擦除后变成了Object。

类型擦除好处：

1. 运行时内存方法区负担小，一个类型取代多个。
2. 兼容性，1.5才推出泛型。

类型擦除的问题：

1. 基本类型无法作为泛型实参，有装箱拆箱的开销。
2. 泛型类型无法用作方法的重载。
3. 无法当做真实的类型使用。反射的
4. 类型强转的运行时开销

**附加的签名信息** Signature，``Gson（TypeToken）``，``Retrofit``（获取泛型的类型）

```java
Method.getGenericReturnType();
```

**混淆的时候保留签名信息**

``Kotlin``会附加注解``@Matadata``添加信息。

