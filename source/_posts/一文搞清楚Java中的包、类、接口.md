---
title: 针对Java中的包、类、接口学习笔记
date: 2024-01-21 20:00:00
categories:
  - Java
tags:
  - Java
  - 包
  - 类
  - 接口
description: 包、类、接口、方法、变量、参数、代码块，这些都是构成Java程序的核心部分，即便最简单的一段代码里都至少要包含里面的三四个内容
cover: https://s2.loli.net/2024/01/21/bljt57GrMwJKeHh.png
---

## 一文搞清楚Java中的包、类、接口

# 写在开头

包、类、接口、方法、变量、参数、代码块，这些都是构成Java程序的核心部分，即便最简单的一段代码里都至少要包含里面的三四个内容，这两天花点时间梳理了一下，理解又深刻了几分。

## Java中的包

Java 定义了一种名字空间，称之为包：package。一个类总是属于某个包，类名（比如Person）只是一个简写，真正的完整类名是包名.类名，这才是唯一确定一个类路径的写法，不同包中的类名允许重复。包名推荐使用倒置的域名，例如org.apache。

包的定义
```
//包名
package hello;

public class Person {
    // 包作用域:
    public void hello() {
        System.out.println("Hello!");
    }
}
```
## 包的作用域

通过访问修饰符，可以确认类中方法与属性相对于包的作用域，这在前面的文章中已经提过了，不再赘言，直接贴图。

![image.png](https://s2.loli.net/2024/01/21/13KCPDL65vHOfgE.png)

Java中包的引入通过import关键字实现，在写import的时候，可以使用*，表示把这个包下面的所有class都导入进来（但不包括子包的class）：
```
import java.util.ArrayList;
public class test {
    public static void main(String[] args) {
        ArrayList<Object> objects = new ArrayList<>();
    }
}
```
当然处理import导入外，也可以通过完成的包名.类名的方式进行引入，但太麻烦了，很少用到。

## Java中的类

Java中有个从1995年就喊出的口号“一切皆对象”，而对象在程序中的具象就是通过类（class）来实现！

# 类的创建

比如有这样一个Person对象，拥有姓名，性别，年龄等特性，行为方式有吃饭，睡觉和跑步，那我们就可以在Java中如下定义：
```
public class Person {
	//姓名
    private String name;
    //年龄
    private int age;
    //性别
    private int sex;
	/**
	* 吃饭
	*/
    private void eat() {
    }
	/**
	* 睡觉
	*/
    private void sleep() {
    }
	/**
	* 跑步
	*/
    private void run() {
    }
}
```

类创建好了，如何用呢？这时候需要通过new关键字去创建一个类对应的对象

**Person person = new Person();**

## 类的初始化赋值

这行代码创建了一个Person对象，并在堆内存中分配一定的空间，person被称为对象Person的引用，通过这个引用可以对对象进行初始化赋值操作
![image.png](https://s2.loli.net/2024/01/21/bljt57GrMwJKeHh.png)

通过引用变量赋值
示例代码：
```
public class Person {
    private String name;
    private int age;
    private int sex;

    public static void main(String[] args) {
        Person person = new Person();
        person.name = "JavaBuild";
        person.age = 18;
        person.sex = 1;
        
        System.out.println(person.name);
        System.out.println(person.age);
        System.out.println(person.sex);
    }
}
```
通过构造方法赋值

示例代码：
```
public class Person {
    private String name;
    private int age;
    private int sex;

    public Person(String name, int age, int sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }

    public static void main(String[] args) {
        Person person = new Person("JavaBuild", 18, 1);

        System.out.println(person.name);
        System.out.println(person.age);
        System.out.println(person.sex);
    }
}
```

## 内部类

根据上面的内容，我们已经熟悉了Java中的类，实际上在类的内部依旧可以创建一个类，这样的类就被称之为：内部类，内部类根据创建的位置，关键字等修饰符分为如下几类：

# 1、成员内部类

编译之后会生成两个class文件：OuterClass.class和OuterClass$InnerClass.class
```
class OuterClass {
    class InnerClass {} //成员内部类
}
```
# 2、方法内部类

编译之后会生成两个class文件：OuterClass.class和OuterClass$1InnerClass.class

只能在定义该内部类的方法内实例化，方法内部类对象不能使用该内部类所在方法的非final局部变量

当一个方法结束，其栈结构被删除，局部变量成为历史。但该方法结束后，在方法内创建的内部类对象可能仍然存在于堆中

```
class OuterClass {
    public void doSomething(){
        class Inner{
        }
    }
}
```

# 3、匿名内部类
编译后生成两个class文件：Fish.class和Fish$1.class
```
public class Fish {
    /**
     * 游泳方法
     */
    public void swim() {
        System.out.println("我在游泳!");
    }

    public static void main(String[] args) {
        //创建鱼对象
        Fish fish = new Fish() {
            //重写swim方法
            public void swim() {
                System.out.println("我在游泳，突然发生海啸，我撤了!");
            }
        };
        
        fish.swim();
    }
}
```
# 4、静态内部类
静态嵌套类，并没有对实例的共享关系，仅仅是代码块在外部类内部

静态的含义是该内部类可以像其他静态成员一样，没有外部类对象时，也能够访问它

静态嵌套类仅能访问外部类的静态成员和方法

在静态方法中定义的内部类也是静态嵌套类，这时候不能在类前面加static关键字
```
class OuterFish {
    static class InnerFish {
    }
}

class TestStaticFish { 
    public static void main(String[] args) {
        //创建静态内部类对象
        OuterFish.InnerFish iFish = new OuterFish.InnerFish();
    }
}
```
## 内部类的特点
- 1、内部类提供了某种进入其继承的类或实现的接口的窗口
- 2、与外部类无关，独立继承其他类或实现接口
- 3、内部类提供了Java的"多重继承"的解决方案，弥补了Java类是单继承的不足
- 4、内部类仍然是一个独立的类，在编译之后内部类会被编译成独立的.class文件，但是前面冠以外部类的类名和$符号
- 5、内部类不能用普通的方式访问。内部类是外部类的一个成员，因此内部类可以自由地访问外部类的成员变量，无论是否是private的
- 6、内部类声明成静态的，就不能随便的访问外部类的成员变量了，此时内部类只能访问外部类的静态成员变量

## Java中的接口

在讲OOP时，我们提到过面向对象的四大特性，其中抽象就是那个第四大特性，而抽象的体现在Java中主要为抽象类和接口！

接口是通过interface 关键字修饰的，用来对一类具有共性对象的一种抽象，通过不同的类进行实现，来满足各自需求。

## 接口的特性
- 1、接口中允许定义变量
- 2、接口中允许定义抽象方法
- 3、接口中允许定义静态方法（Java 8 之后）
- 4、接口中允许定义默认方法（Java 8 之后）
- 5、接口不允许直接实例化
- 6、接口可以是空的
- 7、不要在定义接口的时候使用 final 关键字
- 8、接口的抽象方法不能是 private、protected 或者 final
- 9、接口的变量是隐式 public static final（常量）

## 接口的典型案例

我们在之前聊到对象的浅拷贝与深拷贝时提到过Cloneable接口，这就是一个典型的接口应用案例，Cloneable 和 Serializable 一样，都属于标记型接口，它们内部都是空的。实现了 Cloneable 接口的类可以使用 Object.clone() 方法，否则会抛出 CloneNotSupportedException。

# 接口与抽象类的区别
- 1、抽象类可以有构造方法；接口中不能有构造方法（因为不允许直接实例化）。
- 2、抽象类中可以有普通成员变量；接口中没有普通成员变量。
- 3、抽象类中可以包含非抽象普通方法；JDK1.8 以前接口中的所有方法默认都是抽象的，JDK1.8 开始方法可以有 default 实现和 static 方法。
- 4、抽象类中的抽象方法的访问权限可以是 public、protected 和 default；接口中的抽象方法只能是 public 类型的，并且默认即为 public abstract 类型。
- 5、抽象类中可以包含静态方法；JDK1.8 前接口中不能包含静态方法，JDK1.8 及以后可以包含已实现的静态方法。
- 6、抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量可以是任意访问权限；接口中变量默认且只能是 public static final 类型。
- 7、一个类可以实现多个接口，用逗号隔开，但只能继承一个抽象类。

接口不可以实现接口，但可以继承接口，并且可以继承多个接口，用逗号隔开。

未完待续......

