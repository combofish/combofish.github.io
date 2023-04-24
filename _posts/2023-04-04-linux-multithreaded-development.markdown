---
layout: post
title: "[学习笔记] Linux 多线程开发"
date: "2023-04-04 08: 21: 22"
tag: cpp linux system programming multi-thread
---
- [Linux 多线程开发](#orge192900)
  - [线程概述](#org10265ca)
    - [线程和进程区别](#orgc4f68bd)
    - [线程之间共享和非共享资源](#orgd4e4816)
    - [NPTL](#org9a9c2db)
  - [创建线程](#orgf0c58d8)
  - [线程终止](#orgc8ea943)
  - [线程操作](#org377f3a7)
  - [线程的分离](#org93d730f)
  - [线程的取消](#org40e19ec)
  - [线程属性](#orge9acb23)
  - [线程同步](#org6be4b06)
  - [互斥锁](#org185e408)
  - [死锁](#org3934c9f)
  - [读写锁](#orgfc88959)
  - [生产者和消费者模型](#org7c54141)
  - [条件变量](#org7344390)
  - [信号量](#org647f85f)


<a id="orge192900"></a>

# Linux 多线程开发


<a id="org10265ca"></a>

## 线程概述

与进程 （process） 类似，线程（thread）是允许应用程序并发执行多个任务的一种机制。一个进程可以包含多个线程。 
同一个程序中的所有线程均会独立执行相同的程序，且共享同一份全局内存区域，其中包括初始化数据段、未初始化数据段、以及堆内存段。 
（传统意义上的 UNIX 进程只是多线程程序的一个特例，该进程只包含一个线程）

-   进程是CPU分配资源的最小单位，线程是操作系统调度执行的最小单位。
-   线程是轻量级的进程 (LWP: Light Weight Process)，在 Linux 环境下线程的本质仍是进程。
-   查看指定进程的 LWP 号： ps -Lf pid


<a id="orgc4f68bd"></a>

### 线程和进程区别

进程间的 `信息难以共享`。由于除去只读代码段外，父子进程并未共享内存，因此必须采用一些进程间通信方式，在进程间进行信息交换。

调用 `fork()` 来创建进程的`代价相对较高`，即便利用写时复制技术，仍需要复制诸如内存页表和文件描述符之类的多种进程属性，这意味着 fork() 调用在时间上的开销任然不菲。

线程之间能够`方便、快速的共享信息`。只需将数据复制到共享（全局或堆）变量中即可。

创建线程比创建进程通常快 10 倍甚至更多。线程是`共享虚拟地址空间的`，无需采用写时复制来复制内存，也无需复制页表。

<a id="orgd4e4816"></a>

### 线程之间共享和非共享资源

-   共享资源
    -   进程 ID 和 父进程 ID
    -   进程组 ID 和 会话 ID
    -   用户 ID 和 用户组 ID
    -   文件描述符表
    -   信号处置
    -   文件系统相关信息： 文件权限掩码（umask）、当前工作目录
    -   虚拟地址空间（除 栈、 .text）

-   非共享资源
    -   线程 ID
    -   信号掩码
    -   线程特有的数据
    -   error 变量
    -   实时调度策略和优先级
    -   栈，本地变量和函数的调用链接信息


<a id="org9a9c2db"></a>

### NPTL

NPTL， Native POSIX Thread Library，是 Linux 线程的一个新实现，它克服了 LinuxThreads 的缺点，同时也符合 POSIX 的需求。

-   查看当前 pthread 库版本， getconf GNU_LIBPTHREAD_VERSION


<a id="orgf0c58d8"></a>

## 创建线程

一般情况下， main 函数所在的线程我们称之为`主线程（main 线程）`，其余创建的线程称之为`子线程`。 
- 程序中默认只有一个进程，fork() 函数调用，2个进程。 
- 程序中默认只有一个线程，pthread_create() 函数调用，2个线程。

```c
/**
   NAME
   pthread_create - create a new thread

   SYNOPSIS
   #include <pthread.h>

   int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
   void *(*start_routine) (void *), void *arg);

   Compile and link with -pthread.

   - 功能： 创建一个子进程
   - 参数： thread: 传出参数，线程创建成功后，子线程的线程ID被写到该变量中。
   - attr: 设置线程的属性，一般使用默认值 NULL。
   - start_routine: 函数指针，这个函数是子线程需要处理的逻辑代码
   - arg: 给第三个参数使用, 传参
   - 返回值：
   - 成功 0
   - 失败：返回错误号，这个错误号和之前的 errno 不太一样。
   获取错误号的信息， char* strerror(int errnum);

*/

#include <pthread.h>
#include<stdio.h>
#include <string.h>
//#include <stdlib.h>
//#include <time.h>
#include <unistd.h>

void *callback(void *arg) {
  printf("child thread...\n");
  printf("arg value : %d\n", *(int *) arg);
  return NULL;
}

int main() {
  pthread_t tid;
  int num = 10;

  // 创建一个子线程
  int ret = pthread_create(&tid, NULL, callback, (void *) &num);
  if (ret != 0) {
    char *err_str = strerror(ret);
    printf("errno: %s\n", err_str);
  }

  for (int i = 0; i < 5; i++)
    printf("%d\n", i);

  sleep(1);
  return 0;
}
```

<a id="orgc8ea943"></a>

## 线程终止

-   pthread_exit()

```c
//
// Created by larry on 23-4-2.
//

/**
   NAME
   pthread_exit - terminate calling thread

   SYNOPSIS
   #include <pthread.h>

   void pthread_exit(void *retval);
   - 功能： 终止一个线程，在那个线程中调用，就表示终止那个线程。
   - 参数： retval: 需要传递一个指针，作为一个返回值，可以在 pthread_join() 中获取到。

   NAME
   pthread_self - obtain ID of the calling thread

   SYNOPSIS
   #include <pthread.h>

   pthread_t pthread_self(void);

   Compile and link with -pthread.


   NAME
   pthread_equal - compare thread IDs

   SYNOPSIS
   #include <pthread.h>

   int pthread_equal(pthread_t t1, pthread_t t2);
   - 功能： 比较两个线程ID是否相等
   不同的操作系统， pthread_t 类型的实现不一样，有的是无符号的长整型，有的是使用结构体去实现。

   Compile and link with -pthread.

*/

#include <pthread.h>
#include <stdio.h>
#include <string.h>

void *callback(void *arg) {
  printf("child thread id: %ld\n", pthread_self());
  return NULL; // pthread_exit(NULL);
}

int main() {
  // 创建一个子线程

  pthread_t tid;
  int ret = pthread_create(&tid, NULL, callback, NULL);

  if (ret != 0) {
    char *err_str = strerror(ret);
    printf("errno: %s\n", err_str);
  }

  // main thread
  for (int i = 0; i < 20; i++)
    printf("%d\n", i);


  printf("tid:%ld, main thread id :%ld\n", tid, pthread_self());

  // 让主线程退出，当主线程退出时，不会影响其他正常运行的线程。
  pthread_exit(NULL);

  printf("main thread exit.\n");

  return 0;
}
```


<a id="org377f3a7"></a>

## 线程操作

-   pthread_join()

```c
//
// Created by larry on 23-4-2.
//
/**
   NAME
   pthread_join - join with a terminated thread

   SYNOPSIS
   #include <pthread.h>

   int pthread_join(pthread_t thread, void **retval);
   - 功能： 和一个已经终止的线程进行连接。
   回收子线程的资源，
   这个函数是一个阻塞函数，调用一次只能回收一个子线程。
   一般在主线程中使用。

   - 参数： thread 要回收的子线程的ID
   - retval： 接收子线程退出时的返回值。

   - 返回值：成功 0， 失败： errno.

   Compile and link with -pthread.

*/
#include <string.h>
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

int value = 10; // 局部变量

void *callback(void *arg) {
  printf("child thread: tid: %ld\n", pthread_self());
  // sleep(3);
  // return NULL;

  // int value = 10; // 局部变量
  pthread_exit((void *) &value); // return (void*) &value;
}

int main() {
  pthread_t tid;
  int ret = pthread_create(&tid, NULL, callback, NULL);
  if (ret != 0) {
    char *err_str = strerror(ret);
    printf("err_str: %s\n", err_str);
  }

  // main thread
  for (int i = 0; i < 5; i++)
    printf("%d\n", i);

  printf("tid: %ld, main thread tid: %ld\n", tid, pthread_self());

  int *thread_return_value;
  ret = pthread_join(tid, (void *) &thread_return_value);
  if (ret != 0) {
    char *err_str = strerror(ret);
    printf("join err_str:%s\n", err_str);
  }

  printf("exit code:%d\n", *thread_return_value);
  printf("回收子线程资源成功！\n");

  pthread_exit(NULL);
  return 0;
}
```

<a id="org93d730f"></a>

## 线程的分离

-   pthread_detach()

```c
//
// Created by larry on 23-4-2.
//

/**
   NAME
   pthread_detach - detach a thread

   SYNOPSIS
   #include <pthread.h>

   int pthread_detach(pthread_t thread);
   - 功能： 分离一个线程。被分离的线程在终止的时候，会自动释放资源给系统。
   1、不能多次分离，会产生不可预料的行为。
   2、不能去连接一个已经分离的线程，会报错。

   - 参数：需要分离的线程的ID
   - 返回值： 成功 1， 失败 返回错误号

   Compile and link with -pthread.

*/

#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>

void *callback(void *arg) {
  printf("child pthread:%ld\n", pthread_self());
  return NULL;
}

int main() {
  pthread_t tid;
  int ret = pthread_create(&tid, NULL, callback, NULL);
  if (ret != 0) {
    char *err_str = strerror(ret);
    printf("err_str %s\n", err_str);
  }

  printf("tid: %ld, main thread tid: %ld\n", tid, pthread_self());

  // 设置子线程分离
  pthread_detach(tid);

  // 设置分离以后，对分离的线程进行连接
  ret = pthread_join(tid, NULL);
  if (ret != 0) {
    char *err_str = strerror(ret);
    printf("detach and then join, with:%s\n", err_str);
  }

  for (int i = 0; i < 5; i++)
    printf("%d\n", i);

  pthread_exit(NULL);
  return 0;
}
```

<a id="org40e19ec"></a>

## 线程的取消

-   pthread_cancel()

```c
//
// Created by larry on 23-4-2.
//

/**
   NAME
   pthread_cancel - send a cancellation request to a thread

   SYNOPSIS
   #include <pthread.h>

   int pthread_cancel(pthread_t thread);
   - 功能： 取消线程（让线程终止）
   取消某个线程，可以终止某个线程的运行
   但是并不是立马终止，而是当子线程执行到一个取消点，线程才会终止。
   取消点：系统规定号的一些系统调用，我们可以粗略理解为从用户区到内核区的切换，这个位置称之为取消点。

   Compile and link with -pthread.

*/

#include <stdio.h>
#include <string.h>
#include <pthread.h>

void *callback(void *arg) {
  printf("in child thread: tid: %ld\n", pthread_self());

  for (int i = 0; i < 50; i++)
    printf("child %d\n", i);

  return NULL;
}

int main() {
  pthread_t tid;
  int ret = pthread_create(&tid, NULL, callback, NULL);
  if (ret != 0) {
    char *err_str = strerror(ret);
    printf("err_str: %s\n", err_str);
  }

  // 取消线程
  pthread_cancel(tid);

  for (int i = 0; i < 5; i++)
    printf("%d\n", i);

  printf("pid: %ld, main thread id: %ld\n", tid, pthread_self());

  pthread_exit(NULL);
  return 0;
}
```


<a id="orge9acb23"></a>

## 线程属性

```c
//
// Created by larry on 23-4-2.
//
/**
   NAME
   pthread_attr_init, pthread_attr_destroy - initialize and destroy thread attributes object

   SYNOPSIS
   #include <pthread.h>

   int pthread_attr_init(pthread_attr_t *attr);
   - 功能： 初始化线程属性变量

   int pthread_attr_destroy(pthread_attr_t *attr);
   - 释放线程属性资源

   Compile and link with -pthread.

   NAME
   pthread_attr_setdetachstate, pthread_attr_getdetachstate - set/get detach state attribute in thread attributes object

   SYNOPSIS
   #include <pthread.h>

   int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
   - 设置线程分离的状态属性

   int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
   - 获取线程分离的状态属性

   Compile and link with -pthread.
*/

#include <stdio.h>
#include <pthread.h>
#include <string.h>

void *callback(void *arg) {
  printf("in child thread: %ld", pthread_self());
  return NULL;
}

int main() {
  pthread_attr_t attr;  // 创建一个线程属性变量
  pthread_attr_init(&attr);  // 初始化线程属性变量
  pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED); // 设置属性

  pthread_t tid;
  int ret = pthread_create(&tid, &attr, callback, NULL);
  if (ret != 0) {
    char *str_err = strerror(ret);
    printf("str_err: %s", str_err);
  }
  
  size_t sz;
  pthread_attr_getstacksize(&attr, &sz); // 获取线程栈的大小
  printf("get stack size: %ld\n", sz); // 8388608
  printf("tid: %ld， main thread id:%ld", tid, pthread_self());
  
  pthread_attr_destroy(&attr); // 释放线程属性资源
  pthread_exit(NULL);
  return 0;
}
```


<a id="org6be4b06"></a>

## 线程同步

```c
//
// Created by larry on 23-4-2.
//

/**
   使用多线程实现买票案例
   有三个窗口，一共是100张票。
*/

#include <pthread.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int tickets = 100; // 全局变量，所有线程共享

void *sell_tickets(void *arg) {
  // 卖票
  // int tickets = 100; // 局部变量
  while (tickets > 0) {
    usleep(6000);
    printf("%ld, 正在卖第 %d 张票\n", pthread_self(), tickets);
    tickets--;
  }

  return NULL;
}

int main() {

  // 创建3个子线程
  pthread_t tid1, tid2, tid3;

  pthread_create(&tid1, NULL, sell_tickets, NULL);
  pthread_create(&tid2, NULL, sell_tickets, NULL);
  pthread_create(&tid3, NULL, sell_tickets, NULL);

  // 回收子线程的资源, 阻塞
  pthread_join(tid1, NULL);
  pthread_join(tid2, NULL);
  pthread_join(tid3, NULL);

  // 设置线程分离
  //    pthread_detach(tid1);
  //    pthread_detach(tid2);
  //    pthread_detach(tid3);

  pthread_exit(NULL);
  return 0;
}
```

线程的主要优势在于，`能够通过全局变量来共享信息`。不过，这种便捷的共享是有代价的： 
- 必须保证多个线程不会同时修改同一变量，或者某一线程不会读取正在由其他线程修改的变量。

`临界区`是指访问某一共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一共享资源的其他线程不应中断该片段的执行。

`线程同步`：即当有一个线程正在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，
直到该线程完成操作，其他线程才能对该内存地址进行操作，而其他线程则处于等待状态。


<a id="org185e408"></a>

## 互斥锁

-   互斥量

为了避免线程更新共享变量时出现问题，可以使用`互斥量` mutex( mutual exclusion ) 的缩写来确保同时仅有一个线程可以访问某项共享资源。 
可以使用互斥量来保证对任意共享资源的原子访问。

互斥量有两种状态，（已锁定 `locked`） , 和 (未锁定 `unlocked`)。任何时候，至多只有一个线程可以锁定该互斥量。
试图对已经锁定的某一互斥量再次加锁， 将可能阻塞线程或者报错失败，具体取决于加锁时使用的方法。

一旦线程锁定互斥量，随机成为该互斥量的所有者，只有所有者才能给互斥量解锁。
一般情况下，对每一共享资源（可能有多个相关变量组成）会使用不同的互斥量， 每一线程在访问统一资源时将采用如下协议：
针对共享资源锁定互斥量，访问共享资源，对互斥量解锁。

如果多个线程试图执行这一块代码（一个临界区），事实上只有一个线程能够持有该互斥量（其他线程将遭到阻塞），即同时只有一个线程能够进入这段代码区域。 

![互斥量](/images/tiny-web-server/chapter-3-thread/mutual_exclusion.png)

-   互斥量的类型 pthread_mutex_t

```c
//
// Created by larry on 23-4-3.
//

#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <unistd.h>

int tickets = 1000;
pthread_mutex_t mutex; // 创建一个互斥量

void *sell_tickets(void *arg) {
  // sale
  while (1) {
    pthread_mutex_lock(&mutex);  // 加锁

    if (tickets > 0) {
      usleep(600);
      printf("child thread: %ld are selling %d ticket\n", pthread_self(), tickets);
      --tickets;
    } else {
      pthread_mutex_unlock(&mutex);  // 解锁
      break;
    }
    
    pthread_mutex_unlock(&mutex);  // 解锁
  }
  return NULL;
}

int main() {
  pthread_mutex_init(&mutex, NULL);  // 初始化互斥量
 
  pthread_t tid1, tid2, tid3;

  pthread_create(&tid1, NULL, sell_tickets, NULL);
  pthread_create(&tid2, NULL, sell_tickets, NULL);
  pthread_create(&tid3, NULL, sell_tickets, NULL);

  pthread_join(tid1, NULL);
  pthread_join(tid2, NULL);
  pthread_join(tid3, NULL);
  
  pthread_mutex_destroy(&mutex);  // 释放互斥量资源
  pthread_exit(NULL);
  return 0;
}
```


<a id="org3934c9f"></a>

## 死锁

有时，一个线程需要同时访问两个或更多不同的共享资源，而每个资源又都由不同的互斥量管理。当超过一个线程加锁同一组互斥量时，就有可能发生`死锁`。

两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于`死锁状态`或系统产生了死锁。

死锁的几种场景：

-   忘记释放锁
-   重复加锁
-   多线程多锁，抢占锁资源

```c
//
// Created by larry on 23-4-3.
//

/**
   构成死锁的情况
   1. 忘记解锁
   2. 重复加锁
   3.
*/
#include <stdio.h>
#include <pthread.h>

int tickets = 100;

pthread_mutex_t mutex;

void *sale_tickets(void *arg) {
  while (1) {
    // add lock
    pthread_mutex_lock(&mutex);
    pthread_mutex_lock(&mutex);

    if (tickets > 0) {
      printf("child thread %ld, selling %d ticket\n", pthread_self(), tickets);
      --tickets;
    } else {
      pthread_mutex_unlock(&mutex);
      break;
    }

    pthread_mutex_unlock(&mutex);
  }
  return NULL;
}

int main() {
  pthread_mutex_init(&mutex, NULL);

  pthread_t tid1, tid2, tid3;
  pthread_create(&tid1, NULL, sale_tickets, NULL);
  pthread_create(&tid2, NULL, sale_tickets, NULL);
  pthread_create(&tid3, NULL, sale_tickets, NULL);

  pthread_join(tid1, NULL);
  pthread_join(tid2, NULL);
  pthread_join(tid3, NULL);

  pthread_mutex_destroy(&mutex);

  pthread_exit(NULL);
  return 0;

}
```

```c
//
// Created by larry on 23-4-3.
//

// 死锁的情况 3

#include <pthread.h>
#include <stdio.h>
#include<unistd.h>

// 创建两个互斥量
pthread_mutex_t mutex1, mutex2;

void *workA(void *arg) {

  pthread_mutex_lock(&mutex1);
  usleep(1);
  pthread_mutex_lock(&mutex2);

  printf("workA.... \n");
  pthread_mutex_unlock(&mutex2);
  pthread_mutex_unlock(&mutex1);
  return NULL;
}

void *workB(void *arg) {

  pthread_mutex_lock(&mutex2);
  pthread_mutex_lock(&mutex1);

  printf("workB.... \n");

  pthread_mutex_unlock(&mutex1);
  pthread_mutex_unlock(&mutex2);
  return NULL;
}

int main() {
  pthread_mutex_init(&mutex1, NULL);
  pthread_mutex_init(&mutex2, NULL);

  pthread_t tid1, tid2;

  pthread_create(&tid1, NULL, workA, NULL);
  pthread_create(&tid2, NULL, workB, NULL);
  
  pthread_join(tid1, NULL); // 回收子线程资源
  pthread_join(tid2, NULL);
  pthread_mutex_destroy(&mutex1);
  pthread_mutex_destroy(&mutex2);
  pthread_exit(NULL);
  return 0;
}
```

<a id="orgfc88959"></a>

## 读写锁

当一个线程已经持有互斥锁时，互斥锁将所有试图进入临界区的线程都阻塞住。但是考虑一种情形，当前持有互斥锁的线程只是要读访问共享资源，
而同时 有其他几个线程也想读取这个共享资源，但是由于互斥锁的排它性，所有其他线程都无法获取锁，也就无法读访问共享资源了，
但是实际上多个线程同时读 访问共享资源并不会导致问题。

在对数据的读写操作中，更多的是读操作，写操作比较少，例如对数据库的读写应用。为了满足当前能够允许多个读出，但只允许一个写入的需求，线程提供了`读写锁`来实现。

`读写锁`的特点：

-   如果有其它线程读数据，则允许其它线程执行读操作，但不允许写操作。
-   如果有其它线程写数据，则其它线程都不允许读写操作。
-   写是独占的，写的优先级高。

```c
//
// Created by larry on 23-4-3.
//

/**
   pthread_rwlock_t

   pthread_rwlock_init
   pthread_rwlock_destroy
   pthread_rwlock_rdlock
   pthread_rwlock_tryrdlock
   pthread_rwlock_wrlock
   pthread_rwlock_trywrlock
   pthread_rwlock_unlock

   案例：创建8个线程操作同一个全局变量
   3个线程不定时写这个全局变量
   其它线程不定时读这个全局变量
*/

#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

int num = 1;

pthread_rwlock_t rwlock; // 读时可多个并发读
//pthread_mutex_t mutex; // 多个线程读是需要阻塞

void *writeNum(void *arg) {
  while (1) {
    //        pthread_mutex_lock(&mutex);

    pthread_rwlock_wrlock(&rwlock);
    num++;
    printf("++write, tid: %ld, num = %d\n", pthread_self(), num);
    pthread_rwlock_unlock(&rwlock);

    //        pthread_mutex_unlock(&mutex);
    usleep(100);
  }
  return NULL;
}

void *readNum(void *arg) {
  while (1) {
    //        pthread_mutex_lock(&mutex);

    pthread_rwlock_rdlock(&rwlock);
    printf("===read, tid: %ld, num = %d\n", pthread_self(), num);
    pthread_rwlock_unlock(&rwlock);

    //        pthread_mutex_unlock(&mutex);

    usleep(100);
  }
  return NULL;
}

int main() {
  pthread_rwlock_init(&rwlock, NULL);

  //    pthread_mutex_init(&mutex, NULL);

  pthread_t w_t_ids[3], r_t_ids[5];
  for (int i = 0; i < 3; i++)
    pthread_create(&w_t_ids[i], NULL, writeNum, NULL);

  for (int i = 0; i < 5; i++)
    pthread_create(&r_t_ids[i], NULL, readNum, NULL);

  for (int i = 0; i < 3; i++)
    pthread_detach(w_t_ids[i]);

  for (int i = 0; i < 5; i++)
    pthread_detach(r_t_ids[i]);

  pthread_rwlock_destroy(&rwlock);

  //    pthread_mutex_destroy(&mutex);
  pthread_exit(NULL);
  return 0;
}
```

<a id="org7c54141"></a>

## 生产者和消费者模型

`生产者消费模型`中的对象：

-   生产者
-   消费者
-   容器

```c
//
// Created by larry on 23-4-3.
//

// 生产者消费者模型的简化版

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <malloc.h>
#include <stdlib.h>

struct Node {
  int num;
  struct Node *next;
};

// head node
struct Node *head = NULL;

void *producer(void *arg) {
  // 不断创建新的节点添加到链表中
  while (1) {
    struct Node *newNode = (struct Node *) malloc(sizeof(struct Node));

    // 头插法
    newNode->next = head;
    head = newNode;

    newNode->num = rand() % 1000;
    printf("add node, num: %d, tid: %ld\n", newNode->num, pthread_self());
    usleep(100);
  }
  return NULL;
}

void *customer(void *arg) {
  while (1) {
    struct Node *tmp = head;
    head = head->next;
    printf("delete node num: %d, tid: %ld\n", tmp->num, pthread_self());
    free(tmp);
    usleep(100);
  }
  return NULL;
}

int main() {

  // 创建5个生产者和5个消费者模型
  pthread_t producer_tid_s[5], customer_tid_s[5];

  for (int i = 0; i < 5; i++) {
    pthread_create(&producer_tid_s[i], NULL, producer, NULL);
    pthread_create(&customer_tid_s[i], NULL, customer, NULL);
  }

  for (int i = 0; i < 5; i++) {
    pthread_detach(producer_tid_s[i]);
    pthread_detach(customer_tid_s[i]);
  }

  while (1) {
    sleep(10);
  }
  pthread_exit(NULL);
  return 0;
}
```

<a id="org7344390"></a>

## 条件变量

```c
//
// Created by larry on 23-4-3.
//

/**
   条件变量的类型 pthread_cond_t, 它不是锁

   pthread_cond_init
   pthread_cond_destroy
   pthread_cond_wait
   - 等待，调用了该函数，线程会阻塞
   pthread_cond_timedwait
   - 等待了多长时间，调用了这个函数，线程会阻塞，知道指定的时间结束
   pthread_cond_signal
   - 唤醒一个或多个等待的线程
   pthread_cond_broadcast
   - 唤醒所有等待的线程
*/

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <malloc.h>
#include <stdlib.h>

// 创建一个互斥量
pthread_mutex_t mutex;

//
pthread_cond_t cond;

struct Node {
  int num;
  struct Node *next;
};

// head node
struct Node *head = NULL;

void *producer(void *arg) {
  // 不断创建新的节点添加到链表中
  while (1) {
    pthread_mutex_lock(&mutex);

    struct Node *newNode = (struct Node *) malloc(sizeof(struct Node));

    // 头插法
    newNode->next = head;
    head = newNode;

    newNode->num = rand() % 1000;
    printf("add node, num: %d, tid: %ld\n", newNode->num, pthread_self());

    // 只要生产了一个就通知消费者消费
    pthread_cond_signal(&cond);

    pthread_mutex_unlock(&mutex);
    usleep(100);
  }
  return NULL;
}

void *customer(void *arg) {
  while (1) {
    pthread_mutex_lock(&mutex);

    if (head != NULL) {
      // has data
      struct Node *tmp = head;
      head = head->next;
      printf("delete node num: %d, tid: %ld\n", tmp->num, pthread_self());
      free(tmp);
      pthread_mutex_unlock(&mutex);
      usleep(100);
    } else {
      // no data, need to wait.

      // 当着函数调用阻塞的时候，会对互斥锁进行解锁，当不阻塞的时候，会重新对互斥加锁。
      pthread_cond_wait(&cond, &mutex);
      pthread_mutex_unlock(&mutex);
    }
  }
  return NULL;
}

int main() {
  pthread_mutex_init(&mutex, NULL);
  pthread_cond_init(&cond, NULL);

  // 创建5个生产者和5个消费者模型
  pthread_t producer_tid_s[5], customer_tid_s[5];

  for (int i = 0; i < 5; i++) {
    pthread_create(&producer_tid_s[i], NULL, producer, NULL);
    pthread_create(&customer_tid_s[i], NULL, customer, NULL);
  }

  for (int i = 0; i < 5; i++) {
    pthread_detach(producer_tid_s[i]);
    pthread_detach(customer_tid_s[i]);
  }

  while (1) {
    sleep(10);

  }

  pthread_cond_destroy(&cond);
  pthread_mutex_destroy(&mutex);

  pthread_exit(NULL);
  return 0;
}
```

<a id="org647f85f"></a>

## 信号量

```c
//
// Created by larry on 23-4-3.
//

/**
   信号量的类型 sem_t
   NAME
   sem_init - initialize an unnamed semaphore

   SYNOPSIS
   #include <semaphore.h>

   int sem_init(sem_t *sem, int pshared, unsigned int value);
   - 参数： sem 信号变量的地址
   - pshared: 0 用在线程之间，非0用在进程之间
   - value: 信号量中的值

   sem_destroy
   - 释放资源

   NAME
   sem_wait, sem_timedwait, sem_trywait - lock a semaphore

   SYNOPSIS
   #include <semaphore.h>

   int sem_wait(sem_t *sem);
   - 对信号量的值减1，如果值为0，就阻塞

   int sem_trywait(sem_t *sem);

   int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);

   Link with -pthread.

   sem_post
   - 对信号量解锁，调用一次信号量的值加1.
   sem_getvalue

   sem_t psem;
   sem_t csem;
   init(psem,0, 8)
   init(csem,0, 0)

   producer(){
   sem_wait(&psem);
   sem_post(&csem);
   }

   customer(){
   sem_wait(&csem);
   sem_post(&psem);
   }

*/

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <malloc.h>
#include <stdlib.h>
#include <semaphore.h>

// 创建一个互斥量
pthread_mutex_t mutex;

// sem
sem_t psem, csem;

struct Node {
  int num;
  struct Node *next;
};

// head node
struct Node *head = NULL;

void *producer(void *arg) {
  // 不断创建新的节点添加到链表中
  while (1) {
    sem_wait(&psem);
    pthread_mutex_lock(&mutex);

    struct Node *newNode = (struct Node *) malloc(sizeof(struct Node));

    // 头插法
    newNode->next = head;
    head = newNode;

    newNode->num = rand() % 1000;
    printf("add node, num: %d, tid: %ld\n", newNode->num, pthread_self());

    pthread_mutex_unlock(&mutex);
    sem_post(&csem);
    usleep(100);
  }
  return NULL;
}

void *customer(void *arg) {
  while (1) {
    sem_wait(&csem);

    pthread_mutex_lock(&mutex);
    // has data
    struct Node *tmp = head;
    head = head->next;
    printf("delete node num: %d, tid: %ld\n", tmp->num, pthread_self());
    free(tmp);
    pthread_mutex_unlock(&mutex);

    sem_post(&psem);
    //        usleep(100);

  }
  return NULL;
}

int main() {
  pthread_mutex_init(&mutex, NULL);

  sem_init(&psem, 0, 8);
  sem_init(&csem, 0, 0);

  // 创建5个生产者和5个消费者模型
  pthread_t producer_tid_s[5], customer_tid_s[5];

  for (int i = 0; i < 5; i++) {
    pthread_create(&producer_tid_s[i], NULL, producer, NULL);
    pthread_create(&customer_tid_s[i], NULL, customer, NULL);
  }

  for (int i = 0; i < 5; i++) {
    pthread_detach(producer_tid_s[i]);
    pthread_detach(customer_tid_s[i]);
  }

  while (1) {
    sleep(10);
  }

  pthread_mutex_destroy(&mutex);
  pthread_exit(NULL);
  return 0;
}
```
