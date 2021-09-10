---
title: java-learning
date: 2021-09-06 18:40:34
categories:
tags:
---


## 基础语法

* Object 对象
* Class 类
* Methods 方法
* Instance Variables 属性


```java
public class MyFirstJavaProgram {

   /* This is my first java program.
    * This will print 'Hello World' as the output
    */

   public static void main(String []args) {
      System.out.println("Hello World"); // prints Hello World
   }
}
```

* 大小写敏感
* 类名大些字母开头
* 方法名小写字母开头
* 文件名与类名一致，以 `.java` 结尾
* 以 `public static void main(String args[])` 方法开头


#### 修饰符

权限修饰符：`default`，`public`，`protected`，`private`
非权限修饰符：`final`，`abstract`，`strictfp`


#### 变量

* 局域变量
* 类变量
* 实例变量


## 基础数据类型

#### 8 种原始数据类型
* byte 8bit
* short 16bit
* int 32bit
* long 64bit
* float 32bit
* double 64bit
* boolean
* char 16bit

#### 引用类型
* 类的构造函数中定义的变量
* 类，引用数组
* 引用变量默认为 `null`

#### Java 字符


## 变量类型

```java
data type variable [ = value][, variable [ = value] ...] ;
```

### 局部变量
* 在方法，构造函数或者块内部声明
* 在进入方法，构造函数或者块内部创建，在退出方法，构造函数或者块时销毁
* 权限修饰符不能用在局部变量上
* 局部变量仅在方法，构造函数或者块内部可见
* 局部变量是在栈内存上实现（比堆内存快）
* 局部变量没有默认值，因此需要在使用之前完成声明和初始化（也就是声明的时候就初始化）

### 实例变量
* 
### 类/静态变量

* 类变量也可以成为静态变量，都是以 `static` 关键词在类中声明，而在方法，构造器或者块之外。
* 类中的每个类变量仅有一个副本，无论从中创建了多少对象
* 静态变量存储在静态内存当中



## 数字

## 字符

## 数组

## 日期时间

## 正则

## 方法

## 文件I/O

## 异常

# 面向对象

## 继承
```java
class Calculation {
   int z;

   public void addition(int x, int y) {
      // ...
   }

   public void Subtraction(int x, int y) {
      // ...
   }
}


public class My_Calculation extends Calculation {
   public void multiplication(int x, int y) {
      // ...
   } 
   
   public static void main(String args[]) {
      int a = 20, b = 10;
      My_Calculation demo = new My_Calculation();
      demo.addition(a, b);
      demo.Substraction(a, b);
      demo.multiplication(a, b);
   }
}

```

![](https://www.tutorialspoint.com/java/images/inheritance.jpg)

> 注意：子类继承父类所有的成员属性，构造函数不是成员属性，所以不继承。但是可以从子类调用父类的构造函数

```java
super(values);
super.variable
super.method();
```

#### 继承关系
![](https://www.tutorialspoint.com/java/images/types_of_inheritance.jpg)

## 重载

子类可以重载父类的方法（非 `final` 方法），重载必须保持输入输出要求一致

* 参数必须保持一致
* 函数返回类型必须相同或者是父类声明类型的子类型
* 权限访问只能更大不能更小
* 实例方法只有在子类继承时才可以被重载
* `final` 声明的方法是不能被重载的
* 静态方法不能被重载，只能重新声明
* 不能被继承的方法也不能被重载
* 

## 多态

## 抽象

对使用者隐藏实现细节

#### 抽象类
* 抽象类可以没有抽象方法
* 存在抽象方法的类，必须声明为抽象类
* 

## 接口

## 包


## 多线程

![multihtread](https://www.tutorialspoint.com/java/images/Thread_Life_Cycle.jpg)

### 通过实现 `Runnable` 接口创建线程

> 创建 `Runnable` 接口对应的类，实现 `run()` 方法，通过 `Thread` 创建线程实例，执行 `Thread` 对象的 `start()` 方法启动线程，之后等待系统执行引入的接口类中 `run()` 方法中定义的操作

```java
// 创建 Runnale 类
class RunnableDemo implements Runnable {
   private Thread t;
   private String threadName;
   
   RunnableDemo( String name) {
      threadName = name;
      System.out.println("Creating " +  threadName );
   }
   
   // 实现 run() 方法
   public void run() {
      System.out.println("Running " +  threadName );
      try {
         for(int i = 4; i > 0; i--) {
            System.out.println("Thread: " + threadName + ", " + i);
            // Let the thread sleep for a while.
            Thread.sleep(50);
         }
      } catch (InterruptedException e) {
         System.out.println("Thread " +  threadName + " interrupted.");
      }
      System.out.println("Thread " +  threadName + " exiting.");
   }
   
   // 创建 Thread 实例，并调用 Thread 的 start() 方法，注意这里的 start() 方法和 Thread 实例的 start() 的区别。是为了创建线程。
   public void start () {
      System.out.println("Starting " +  threadName );
      if (t == null) {
         t = new Thread (this, threadName);
         t.start ();
      }
   }
}

public class TestThread {

   public static void main(String args[]) {
      RunnableDemo R1 = new RunnableDemo( "Thread-1");
      R1.start();
      
      RunnableDemo R2 = new RunnableDemo( "Thread-2");
      R2.start();
   }   
}


```

### 通过继承 `Thread` 类创建线程

> 创建 `Thread` 继承类，重载 `run()` 方法，通过 `Thread` 创建线程实例，执行 `Thread` 对象的 `start()` 方法启动线程，之后等待系统执行引入的继承类中 `run()` 方法中定义的操作

**通过 Runnable/Thread 我们看到 Thread 这个线程类接收 Runable 和 Thread 两种类，而实际 Thread 类也是实现了 Runnable 接口，所以只要是实现了 Runable 接口的类都可以传入 Thread 类中，这就是依赖注入的形式**

```java
class ThreadDemo extends Thread {
   private Thread t;
   private String threadName;
   
   ThreadDemo( String name) {
      threadName = name;
      System.out.println("Creating " +  threadName );
   }
   
   public void run() {
      System.out.println("Running " +  threadName );
      try {
         for(int i = 4; i > 0; i--) {
            System.out.println("Thread: " + threadName + ", " + i);
            // Let the thread sleep for a while.
            Thread.sleep(50);
         }
      } catch (InterruptedException e) {
         System.out.println("Thread " +  threadName + " interrupted.");
      }
      System.out.println("Thread " +  threadName + " exiting.");
   }
   
   public void start () {
      System.out.println("Starting " +  threadName );
      if (t == null) {
         t = new Thread (this, threadName);
         t.start ();
      }
   }
}

public class TestThread {

   public static void main(String args[]) {
      ThreadDemo T1 = new ThreadDemo( "Thread-1");
      T1.start();
      
      ThreadDemo T2 = new ThreadDemo( "Thread-2");
      T2.start();
   }   
}

```

### 通过 `Callable` 和 `Future` 创建线程
> 创建 `Callable` 接口的实现类， 实现 `call()` 方法,
> 创建 `Callable` 实现类的实例，通过 `FutureTask` 类来包装 `Callable` 对象。

```java
public class CallableThreadTest implements Callable<Integer> {
    public static void main(String[] args) {
        CallableThreadTest ctt = new CallableThreadTest();
        FutureTask<Integer> ft = new FutureTask<>(ctt);

        for (int i = 0; i < 100; i++ ) {
            System.out.println(Thread.currentThread().getName() + " loop i: " + i);
            if (i == 20) {
                new Thread(ft, "has return value'").start();
            }
        }

        try {
            System.out.println("subthread return value: " + ft.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    public Integer call() throws Exception {
        int i = 0;
        for (; i <100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
        return i;
    }
}
