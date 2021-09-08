---
layout:     post
title:      "Java反射学习之Field类访问和修饰变量"
subtitle:   "Java反射"
date:       2020-03-07 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - Jacoco
    - Java
---

Field类提供访问类成员属性和值一系列方法，最常用的就是访问成员变量值和修改变量值.

众所周知,变量修饰符有pubilc,protected,default,private，修饰符提供的访问权限依次变小，private修饰的只能本类中访问，但是，通过java.lang.reflet.Field类就能访问和修改private修饰的私有变量了，example如下:

 

1. 等待被访问Person类及其变量:

```java
public class Person {
    
    public String name = "graham";
    protected String sex = "男";
    String province = "天津";
    private int age = 26;

}
```

2. 通过Field类的get(Object ojb)和set(Object ojb, Object value),访问和修改Person类的共私有成员:

```java
public class TestClass {
    public String aa ="aaa";
    private String bb = "bbb";
    protected String cc = "ccc";

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        Person person = new Person();

        //访问公有成员用getField或者getDeclaredField
        Field field  = person.getClass().getField("name");
        field.set(person, "grahamliu");
        System.out.println(field.get(person));

        //访问非公有成员必须用getDeclaredField
        Field field1  = person.getClass().getDeclaredField("sex");
        field1.set(person, "女");
        System.out.println(field1.get(person));

        Field field2  = person.getClass().getDeclaredField("province");
        field2.set(person, "南极");
        System.out.println(field2.get(person));

        //访问和修饰private变量，必须设置setAccessible为true
        Field field3  = person.getClass().getDeclaredField("age");
        field3.setAccessible(true);
        field3.set(person, 87);
        System.out.println(field3.get(person));

        //访问本类private变量，不用设置setAccessible
        Field field4 = TestClass.class.getDeclaredField("cc");
        System.out.println(field4.get(new TestClass()));
    }
}
```

3. 打印:

```java
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/bin/java "-javaagent:/Applications/IntelliJ IDEA 
grahamliu
女
南极
87
ccc

Process finished with exit code 0
```



涉及2个知识点:

1. 访问公有成员用getField或者getDeclaredField，访问非公有成员必须用getDeclaredField

2. 访问和修饰private变量，必须设置setAccessible为true，本类除外!
