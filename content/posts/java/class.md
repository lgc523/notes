---
title: "Class"
date: 2021-12-12T10:21:27+08:00
draft: true
toc: true
images:
tags: 
  - java
---

Jvm 的跨平台可以让一份 Class 文件运行在不通的平台上，Class 作为异构语言和 JVM 之间的重要桥梁，可以由源代码被编译成 Class 文件，并最终在 JVM 上执行，Clojure、Groovy、Scala、Jython等语言都可以活跃在 JVM 平台上。

## Class File

- 类的属性
- 类的方法
  - 访问标记
  - 名称
  - 描述符
  - 属性
    - 行号属性
    - 局部变量表
    - 栈映射帧
- 类的字段
  - 访问标记
  - 名称
  - 描述符
  - 属性
- 实现的接口
- 父类
- magic number  Class 文件的特征
- 小版本号
- 大版本号
- 常量池
  - 各种常量
  - 整数
  - 字符串
  - Class
- 访问标记
  - public
  - static
- 当前类

虚拟机规范中，Class 文件使用一种类似于 C 语言结构体的方式描述，统一使用无符号整数作为基本数据类型，u1、u2、u3、u4、u8 分别表示无符号的单字节、2字节、4字节、8字节整数，字符串用 u1 数组表示。

```
ClassFile{
	u4 magic;
	u2 minor_version;
	u2 major_version;
	u2 constant_pool_count;
	cp_info constant_pool[constant_pool_count-1];
	u2 access_flags;
	u2 this_class;
	u2 super_class;
	u2 interfaces_count;
	u2 interfaces[interfaces_count]
	u2 fields_count;
	field_info fields[fields_count]
	u2 methods_count;
	method_info methods[methods_count]
	u2 attributes_count;
	attribute_info attributes[attributes_count]
}
```

### Magic Number

魔数作为 Class 文件的标志，告诉虚拟机这个是 Class 文件，4字节无符号整数，固定为 0xCAFEBABE

```java
public class Simple {
    public static final int TYPE = 1;
    private int id;
    private String name;

    public void setId(int id) {
        try {
            this.id = id;
        } catch (IllegalStateException e) {
            System.out.println(e.toString());
        }
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

hexdump 查看 class

```
0000000 ca fe ba be 00 00 00 3d 00 2f 0a 00 02 00 03 07
0000010 00 04 0c 00 05 00 06 01 00 10 6a 61 76 61 2f 6c
......
```

### Version

魔数后面跟着 小版本号和大版本号（都是2字节）

十进制转换十六进制

**echo "obase=16; 52" | bc**  

**printf '%x\n' 52** 

| 小版本 | 大版本  | 编译器版本 |
| ------ | ------- | ---------- |
| 3      | 45      | 1.1        |
| 0      | 46      | 1.2        |
| 0      | 47      | 1.3        |
| 0      | 48      | 1.4        |
| 0      | 49      | 1.5        |
| 0      | 50 0x32 | 1.6        |
| 0      | 51 0x33 | 1.7        |
| 0      | 52 0x34 | 1.8        |
| 0      | 53 0x35 | 1.9        |
| 0      | 54 0x36 | 10         |
| 0      | 55 0x37 | 11         |
| 0      | 56 0x38 | 12         |
| 0      | 57 0x39 | 13         |
| 0      | 58 0x3a | 14         |
| 0      | 59 0x3b | 15         |
| 0      | 60 0x3c | 16         |
| 0      | 61 0x3d | 17         |

Java -target 可是使用指定版本的发行版编译器进行编译。

### constant_pool

版本号后面跟着的是常量池的数量和若干个常量池表项，

``常量池 0 为空缺页，不存放实际内容，常量池的每一项以类型、长度、内容或者类型、内容的格式依次排列。``
