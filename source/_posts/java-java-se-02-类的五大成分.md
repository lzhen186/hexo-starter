---
title: Java Java SE 02.类的五大成分
date: 2022-07-04T01:54:24.698Z
tags: [javase]
---
类中有且仅有5大成分（五大金刚）

- 成员变量Field：描述类或者对象的属性信息的。
- 成员方法Method：描述类或者对象的行为的。
- 构造器（构造方法,Constructor）: 初始化类的一个对象返回。
- 代码块：代码块按照有无static可以分为静态代码块和实例代码块。
- 内部类：将一个类定义在另一个类里面或者一个方法里面，这样的类称为内部类。

# 1. 成员变量Field

在Java中对象的属性称为成员变量。为了了解成员变量，在下面的代码中首先定义一个图书类，成员变量对应于类对象的属性，在Book类中设置3个成员变量，分别为id,name和category,分别对应于图书编号，图书名称和图书类别3个图书属性。

```java
public class Book{
 private String name;    //定义一个String类型的成员变量
 public String getName(){  //定义一个getName()方法
   int id=0;               //局部变量
   setName=("java");       //调用类中其他方法
   return id+this.name;  //设置方法返回值
   }
   public void setName(String name){   //定义一个setName()方法
     this.name=name;         //将参数赋值于类中的成员变量
     }
   public Book getBook(){
     return this;          //返回Book类引用
     }
 }
```

成员变量可以设置初始值，也可以不设置，如果不设置初始值，则会有默认值。

# 2. 成员方法Method

在Java语言中使用成员方法对应于类对象的行为。以上面代码中Book类为例，它包含getName()和setName()两个方法。

一个成员方法可以有参数，这个参数可以是对象，也可以是基本数据类型的变量。同时成员方法有返回值和不返回任何值的选择，如果需要返回值，可以在方法体中使用return关键字，返回值可以是计算结果，也可以是其他想要的数值和对象，无返回值可以使用void关键字表示。

在成员方法中可以调用其他成员方法和类成员变量，例如上述代码中getName()方法中就调用了setName()方法将图书名称赋予一个值。

注：关于权限修饰符

如果一个类的成员变量或成员方法被修饰为private，则只能在本类中使用，在子类中不可使用，并且在其他包的类中是不可见的。如果被修饰为public，则在子类和其他包的类中可以使用。

# 3. 构造器(构造方法,Constructor)

## 3.1 构造器的作用

通过调用构造器可以返回一个类的对象，构造器同时负责帮我们把对象的数据（属性和行为等信息）初始化好。

## 3.2 构造器的格式

```java
修饰符 类名(形参列表) {
    // 构造体代码，执行代码
}
```

## 3.3 构造器应用

首先定义一个学生类，代码如下：

```java
public class Student {
    // 1.成员变量
    public String name;
    public int age;

    // 2.构造器
    public Student() {
		System.out.println("无参数构造器被调用")；
    }
}
```

接下来通过调用构造器得到两个学生对象。

```java
public class CreateStu02 {
    public static void main(String[] args) {
        // 创建一个学生对象
        // 类名 变量名称 = new 类名();
        Student s1 = new Student();
        // 使用对象访问成员变量，赋值
        s1.name = "张三";
        s1.age = 20 ;

        // 使用对象访问成员变量 输出值
        System.out.println(s1.name);
        System.out.println(s1.age); 

        Student s2 = new Student();
        // 使用对象访问成员变量 赋值
        s2.name = "李四";
        s2.age = 18 ;
        System.out.println(s2.name);
        System.out.println(s2.age);
    }
}
```

# 4. 代码块

## 4.1 静态代码块

**静态代码块**
​         必须有static修饰，必须放在类下。与类一起加载执行。

**格式**

```java
static{
     // 执行代码
}
```

**特点**：

- 每次执行类，加载类的时候都会先执行静态代码块一次。
- 静态代码块是自动触发执行的，只要程序启动静态代码块就会先执行一次。
- 作用：在启动程序之前可以做资源的初始化，一般用于初始化静态资源。

**案例演示**

```java
public class DaimaKuaiDemo01 {
    public static String name ;

    // 1.静态代码块
    static {
        // 初始化静态资源
        name = "张三";
        System.out.println("静态代码块执行！");
    }

    public static void main(String[] args) {
        System.out.println("main方法执行");
        System.out.println(name);
    }
}

```

## 4.2 实例代码块

**实例代码块**
​         没有static修饰，必须放在类下。与对象初始化一起加载。

**格式**

```java
{
     // 执行代码
}
```

**特点**：

- 无static修饰。属于对象，与对象的创建一起执行的。
- 每次调用构造器初始化对象，实例代码块都要自动触发执行一次。
- 实例代码块实际上是提取到每一个构造器中去执行的。
- 作用：实例代码块用于初始化对象的资源。

**案例演示**

```java
public class DaimaKuaiDemo02 {
   
    private String name ;

    // 实例代码块。 无static修饰。
    {
        System.out.println("实例代码块执行");
        name = "dl";
    }

    // 构造器
    public DaimaKuaiDemo02(){
        //System.out.println("实例代码块执行");
    }

    // 有参数构造器
    public DaimaKuaiDemo02(String name){
        //System.out.println("实例代码块执行");
    }

    public static void main(String[] args) {
        // 匿名对象，创建出来没有给变量。
        new DaimaKuaiDemo02();
        new DaimaKuaiDemo02();
        new DaimaKuaiDemo02("xulei");
    }
}
// 输出三次：实例代码块执行
```

# 5. 内部类

请看这一节[Java内部类详解](./03.Java内部类详解.md)