---
title: "Class load"
date: 2021-12-12T13:07:55+08:00
draft: true
toc: true
images:
tags: 
  - java
---

## Class load

jvm 装载 class 分为 加载 -> 连接（验证->准备->解析）-> **初始化**。

### condition

Class 文件只有在必须要使用的时候才会被装载，NoClassDefFoundError 不会在编译的时候出现。

Jvm 规定，一个类或接口在**初次主动使用**前，必须进行初始化。

主动使用

- 创建类实例，new/reflect/clone/deserialize
- 调用类的静态方法，字节码的 invokestatic 指令
- 使用类或接口的静态字段（final 常量除外）
- reflect 反射类的方法
- 初始化子类
- 启动 jvm 的main 方法所在类

其他情况属于被动使用，不会引起类的初始化，都会进行加载、连接。

- 通过子类访问父类的 static 域，只会初始化父类 -XX:+TraceClassLoading 参数可以看到 Loaded 日志
- 访问 final 常量 ，常量被放在常量池中，编译时将常量直接植入目标类，不实用被引用的类。

### load

加载类时，jvm 必须完成的工作

- 通过类全名获取类的二进制数据流
- 解析类的二进制数据流为方法区内的数据结构
- 创建 java.lang.Class 类的实例，表示该类型（元数据）

### verify

连接-验证：保证加载的字节码合法、合理符合规范。

- 格式检查
  - 魔数检查
  - 版本检查
  - 长度检查
- 语义检查
  - 是否继承 final
  - 是否有 super class
  - 是否实现 abstract method
- 字节码验证
  - 跳转指令是否指向正确位置
  - 操作数类型是否合理

前面三次检查，可以排除文件格式错误、语义错误及字节码的不正确，依然不能保证类是没问题的。

- 符号引用验证
  - 符号引用的直接引用是否存在
  - Class 文件在常量池会通过字符串记录自己将要使用的其他类或者方法，这个时候校验器会检查这些类或者方法是否确实存在，并且确定当前类有权限访问这些数据。如果一个需要使用的类无法在系统中找到，会抛出 NoClassDefFoundError，如果一个方法无法在系统中找到，会抛出 NoSuchMethodError。

### prepare

当类验证通过，虚拟机就会进入准备阶段，虚拟机为类分配响应的内存空间，设置初始值。

**如果存在常量字段，常量字段也会在准备阶段被赋上正确的值，赋值行为属于虚拟机行为，属于变量的初始化，准备阶段不会有任何 Java 代码被执行。**

### parse

解析阶段将类、接口、字段和方法的**符号引用转为直接引用**，**得到类或者字段、方法在内存中的指针偏移或者偏移量**。如果直接引用存在，可以肯定系统中存在该类、方法或者字段，只有符号引用存在，不能确定系统中一定存在该对象。

符号引用就是一些字面量的引用（常量池），和虚拟机的内部数据结构与内存布局无关。

解析字节码，找到常量池中的符号引用，在通过类的方法表，通过偏移量找到方法，完成转换，调用方法成功。

### CONSTANT_string

Java 代码中直接使用字符串常量时，会在类中出现 CONSTANT_String，表示字符串常量，会引用一个 CONSTANT_UTF8 常量项。

虚拟机内部运行时的常量池中，会维护一张字符串拘留表（intern），会保存所有出现过的字符串常量，没有重复项。

使用 String.intern() 可以得到一个字符串在拘留表中的引用，

```java
        String a = Integer.toString(1) + Integer.toString(2) + Integer.toString(3);
        String b = "123";
        System.out.println(a.equals(b)); true
        System.out.println(a==b); false
        System.out.println(a.intern()==b); true
```

### init

前面的步骤都没有问题，类就可以顺利装载到系统中，这个时候才开始执行 Java 字节码。

**初始化最重要的工作时执行类的初始化方法  clinit，方法  clinit 是由编译器自动生成的，是由静态成员的赋值语句及 static 语句快共同产生的**。

存在继承的类，父类的 clint 方法总是在子类的 clint 方法之前被调用，并且虚拟机内部保证多线程环境安全（锁），但是还是有可能会出现死锁。

如果类没有 static 赋值语句或者静态代码块，编译器就不会生成 clint 方法，static final 赋值在准备阶段完成。

```java
public class StaticA {
    static {
        try {
            Thread.sleep(1000);

        }catch (Exception e){

        }
        try {
            Class.forName("dev.spider.jvm.clint.StaticB");
        }catch (Exception e){

        }
        System.out.println("StaticB init OK");
    }
}
public class StaticB {
    static {
        try {
            Thread.sleep(1000);

        }catch (Exception e){

        }
        try {
            Class.forName("dev.spider.jvm.clint.StaticA");
        }catch (Exception e){

        }
        System.out.println("StaticB init OK");
    }
}
```

```java
public class StaticDeadLockMain extends Thread{
    private char flag;
    public StaticDeadLockMain(char flag){
        this.flag=flag;
        this.setName("Thread"+flag);
    }

    @Override
    public void run() {
        try {
            Class.forName("dev.spider.jvm.clint.Static"+flag);
        }catch (Exception e){

        }
        System.out.println(getName()+" over");
    }
}
```

```java
public class ClintDeadLock {
    public static void main(String[] args) {
        StaticDeadLockMain loadA = new StaticDeadLockMain('A');
        loadA.start();
        StaticDeadLockMain loadB = new StaticDeadLockMain('B');
        loadB.start();
    }
}
```

## ClassLoader

Classloader 在 Class 装载的加载阶段，从系统外部获得 Class 二进制数据流。
