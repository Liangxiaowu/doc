最近因为公司的业务需求，为了解决一定的并发，所以使用到了线程相关的编程，公司的某个同事比较好奇，就问了我相关线程的知识点。俗话说教一些不善于理解的人，直接拿案例展示比较能让他通俗易懂，于是乎我就给他写了一些小案例，顺便总结一些自己的笔记。

#### 什么线程

线程是操作系统能够进行运算的调度的最小单位，它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。进程的内存空间是共享的，每个线程都可以使用这些共享内存。

#### 为什么用线程

前面我有说过线程能做到一定的并发处理。多线程还能满足编程充分利用 CPU来达到高效率的目的。

这时候我们顺便来了解一下并发和并行的概念。

并发：指短时间间隔内，执行了多个任务。

并行：指在同一个时刻，执行了多个任务。

单核 CPU在同一个时刻只能执行一个任务点，为什么我们电脑会程序和程序之间是同时在运行，那是因为我们伟大的cpu切换的速度足够快，快到我们凡人是感觉不出来他在切换处理中。

所以并行的前提，常理来说要多核运行才行。

这个时候这位同事就问我，那我每次的任务都开启多线程，我的程序不就变的飞快了吗？

非也，线程的使用是有消耗的，越多的线程消耗就会越多，如果你执行是一些不怎么消耗时间的任务，你执意要去使用多线程就会变得物极必反的效果。而且我们还需要考虑到线程安全的问题等。

#### 创建线程

- 通过继承Thread类

  ```java
  public class ThreadTest extends java.lang.Thread {
  
      // 线程入口
      @Override
      public void run(){
          // 线程执行体
          for (int i=0;i <= 100; i++){
              System.out.println("Thread线程执行-----"+i);
          }
  
      }
  
  
      public static void main(String[] args){
  
          // 创建线程对象
          ThreadTest threadTest = new ThreadTest();
  
          // 启动线程
          threadTest.start();
  
          for (int i=0;i<=100;i++){
              System.out.println("ThreadTest程序正在执行----"+i);
          }
      }
  
  
  }
  ```

  

- 通过实现Runnable接口【推荐】

  ```java
  public class RunnableTest implements Runnable{
  
      // 线程入口
      @Override
      public void run(){
          // 线程执行体
          for (int i=0;i <= 100; i++){
              System.out.println("Runnable线程执行-----"+i);
          }
  
      }
  
  
      public static void main(String[] args){
  
          // 创建线程对象
          RunnableTest runnableTest = new RunnableTest();
  
          Thread thread = new Thread(runnableTest);
  
          // 启动线程
          thread.start();
  
          for (int i=0;i<=100;i++){
              System.out.println("RunnableTest程序正在执行----"+i);
          }
      }
  
  }
  ```

  

#### 小案例

假如银行里只有一个窗口那么办理业务，有个老奶奶来了办卡，窗口就会受理老奶奶的办卡需求，这时候来了个小伙子也要办卡，老奶奶还没办理完成小伙子只能等待老奶奶完成之后再到窗口办理。

```java
public class SynBank  {

    public static void run(String name){
        for (int i=1;i<=10;i++){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
          	//老奶奶的动作缓慢需要消耗的时间长点
            if(name.equals("老奶奶")){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(name+"完成了办卡第"+i+"步");
        }
    }

    public static void main(String[] args){
        long startTime = System.currentTimeMillis();
        run("老奶奶");
        run("小伙机");
        long endTime = System.currentTimeMillis();
        System.out.println("总共耗时："+(endTime-startTime));
    }
}
```

通过以上程序的模拟得到，这种情况下需要的时间会在25秒以上

```	java
老奶奶完成了办卡第1步
老奶奶完成了办卡第2步
...
小伙机完成了办卡第9步
小伙机完成了办卡第10步
总共耗时：25110

Process finished with exit code 0

```

于是银行觉得这种效率太慢了，决定开启多窗口受理方式来接待银行的顾客。

那么老奶奶第一个来的时候，会被分配到窗口一去受理，这时候小伙子过来了，就会被分配到窗口二去办理。

```java
public class Bank implements Runnable {

    @Override
    public void run(){
        long startTime = System.currentTimeMillis();
        for (int i=1;i<=10;i++){
            // 假如每一步的流程都要消耗1秒钟

            if(Thread.currentThread().getName().equals("老奶奶")){
                System.out.println(Thread.currentThread().getName()+"完成了办卡第"+i+"步");
                try {
                    Thread.sleep(1500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }else {
                System.out.println(Thread.currentThread().getName()+"完成了办卡第"+i+"步");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }
        if(Thread.currentThread().getName().equals("老奶奶")){
            long endTime = System.currentTimeMillis();
            System.out.println("总共耗时："+(endTime-startTime));
        }

    }

    public static void main(String[] args){
        Bank bank = new Bank();
        new Thread(bank,"老奶奶").start();
        new Thread(bank,"小伙机").start();
    }
}

```

通过以上程序的模拟得到，总共消耗的时间为15秒，我们会发现小伙子最终完成的比老奶奶要快，他们的任务互不干扰，所以不需要等待老奶奶办理完成才开始受理其他人，这样大大提升了银行的效率。

```java
老奶奶完成了办卡第1步
小伙机完成了办卡第1步
...
老奶奶完成了办卡第8步
小伙机完成了办卡第10步
老奶奶完成了办卡第9步
老奶奶完成了办卡第10步
总共耗时：15039
Process finished with exit code 0
```

#### 总结

在某种耗时任务的时候，我们适当去使用多线程模式，时间上不单单会快很多，还解决我们一部分的并发。但是需要注意的是线程的开启是对资源有一定的消耗，所以不能盲目去使用线程做开发。

