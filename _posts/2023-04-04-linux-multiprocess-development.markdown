---
layout: post
title: "[学习笔记] Linux 多进程开发"
date: "2023-04-04 08: 21: 22"
tag: cpp linux system programming multiprocess
---
- [Linux 多进程开发](#org1c331bd)
  - [进程概述](#orgbc4c316)
    - [进程控制块](#orgddcacfb)
  - [进程的状态转换](#orgf925f24)
    - [查看进程](#org9d90058)
    - [实时显示进程动态](#orgedbc03f)
    - [杀死进程](#orgae721df)
    - [进程号和相关函数](#org16638a3)
  - [进程创建](#org198ce36)
  - [父子进程关系及GDB调试](#orgaec1b14)
    - [调试](#orgee9fbce)
  - [exec 函数族](#org37cdcd0)
  - [进程控制](#org1a4c685)
    - [进程退出](#orgd33563b)
    - [孤儿进程](#orgb232b6c)
    - [僵尸进程](#orgcf50b7e)
  - [wait 函数](#org3b434e7)
  - [waitpid()](#org15ef955)
  - [进程间通信](#org68db3f0)
    - [进程间通信概念](#orgd467f68)
  - [匿名管道概述](#orgcbf66d8)
    - [管道特点](#orge05c2a7)
  - [父子进程通过匿名管道通信](#orgc6d7bdc)
  - [匿名管道通信案例](#org8fcb818)
  - [管道的读写特点和管道设置为非阻塞](#orgfd892d4)
  - [有名管道](#org6e156c8)
    - [有名管道的注意事项](#org4f64069)
  - [有名管道完成聊天功能](#orgb63f9ac)


<a id="org1c331bd"></a>

# Linux 多进程开发


<a id="orgbc4c316"></a>

## 进程概述

程序是包含一系列信息的文件， 这些信息描述了如何在运行时创建一个进程：

进程： 是正在运行的程序的实例。

-   单道程序： 即在计算机内存中只允许一个的程序运行。
-   多道程序： 是在计算机内存中同时存放几道相互独立的程序，使它们在管理进程控制下，相互穿插运行，两个或两个以上程序在计算机系统中同时处于开始到结束之间的状态，这些程序共享计算机系统资源。

引入多道程序设计技术的根本目的是为了提高CPU的利用率。

-   时间片 timeslice

由操作系统内核的调度程序分配给每个进程。首先，内核会给每个进程分配相等的初始时间片，然后每个进程轮番地执行相应的时间，当所有的进程都处于时间片耗尽的状态时，内核会重新为每个进程计算并分配时间片，如此往复。

-   并行 （parallel） 指在同一时刻， 有多余指令在多个指令执行。如：两个队列在同时使用两台咖啡机。

-   并发 （concurrency） 指多个进程指令被快速的轮换执行，使得宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行，只是把时间分成若干段，使得多个进程快速交互的执行。

如：两个队列交替使用一台咖啡机。


<a id="orgddcacfb"></a>

### 进程控制块

为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。内核为每个进程分配一个 PCB(Processing Control Block)进程控制块，维护进程相关的信息， Linux内核的进程控制块是 task<sub>struct</sub> 结构体。


<a id="orgf925f24"></a>

## 进程的状态转换

-   三态模型：运行态，阻塞态，就绪态
-   五态模型：新建态，就绪态，运行态，阻塞态，终止态。

-   运行态：进程占有处理器正在运行
-   就绪态：进程具备运行条件，等待系统分配处理器以便运行。当进程已分配到除CPU以外的所有必要资源后，只要再获得CPU,便可立即执行。在一个系统中就绪状态的进程可能有多个，通常将他们排成一个队列，成为就绪队列。
-   阻塞态：又称等待(wait)态或者睡眠(sleep)态，指进程不具备运行条件，正在等待某个时间完成。
-   新建态：进程刚被创建时的状态，尚未进入就绪队列
-   终止态：进程完成任务到达正常结束点，或出现无法克制的错误而异常终止，或被操作系统以及有终止权的进程所终止时所处的状态。进入终止态的进程以后不在执行，但依然保留在操作系统中等待善后。一旦其他进程完成了对终止态进程的信息抽取之后，操作系统将删除该进程。


<a id="org9d90058"></a>

### 查看进程

-   ps

```shell
ps aux/ ajx
# a: 显示终端上的所有进程，包括其他用户的进程
# u: 显示进程的详细信息
# x: 显示没有控制终端的进程
# j: 列出与作业控制相关的信息

# STAT 参数的意义：
# D          不可中断，Uninterruptible (usually IO)
# R          正在运行，或在队列中的进程
# S(大写)    处于休眠状态
# T          停止或被追踪
# Z          僵尸进程
# W          进入内存交换（从内核2.6开始无效）
# X          死掉的进程
# <          高优先级
# N          低优先级
# s          包含子进程
# +          位于前台的进程组
```


<a id="orgedbc03f"></a>

### 实时显示进程动态

-   top

```shell
top # 可以在使用 top 命令时加上 -d 来指定显示信息更新的时间间隔， 在 top 命令执行后，可以按以下按键对显示的结果进行排序

# M    根据内存使用量排序
# P    根据CPU占有率排序
# T    根据进程运行时间长短排序
# U    更具用户名来筛选进程
# K    输入指定的PID杀死进程
```


<a id="orgae721df"></a>

### 杀死进程

-   kill

```shell
kill [-signal] pid
kill -l # 列出所有信号
kill -SIGKILL 进程ID
kill -9 进程ID

killall name 根据进程名杀死进程
```


<a id="org16638a3"></a>

### 进程号和相关函数

每个进程都是由进程号来标识，其类型为 pid<sub>t</sub> （整型）, 进程号的范围： 0~32767, 进程号总是唯一的，但可以重用。当一个进程终止后，其进程号就可以再次使用。

任何进程（除 init 进程）都是由另一个进程创建，该进程称为被创建进程的父进程，对应的进程号称为父进程号 (PPID);

进程组是一个或多个进程的集合，他们之间相互关联，进程组可以接收同一终端的各种信息，关联的进程有一个进程组号 （PGID）。默认情况下，当前的进程号会当做当前进程组号。

-   进程号和进程组相关函数。

```cpp
pid_t getpid(void);
pid_t getppid(void);
pid_t getpgid(pid_t pid);
```


<a id="org198ce36"></a>

## 进程创建

-   系统允许一个进程创建新的进程，新进程即为子进程。

```cpp
/**
   #include <sys/types.h>
   #include <unistd.h>

   pid_t fork(void);
   作用： 用于创建子进程
   返回值： 返回值会返回两次，一次是在父进程中，一次是在子进程中。
   子父进程中返回创建的子进程的ID, 在子进程中返回0,
   在父进程中返回-1， 创建进程失败，会设置errno。

*/

#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  int num = 10;

  pid_t pid = fork();

  if (pid > 0) {
    printf("pid: %d\n", pid);
    printf("parent process, pid : %d, ppid: %d\n", getpid(), getppid());

    printf("num: %d\n", num);
    num += 10;
    printf("num + 10: %d\n", num);

  } else if (pid == 0) {
    printf("child process, pid : %d, ppid: %d\n", getpid(), getppid());

    printf("num: %d\n", num);
    num += 100;
    printf("num + 100: %d\n", num);
  }

  for (int i = 0; i < 3; i++) {
    printf("i: %d, pid: %d\n", i, getpid());
    sleep(1);
  }

  return 0;
}
```

-   fork 以后， 子进程的用户区和父进程一样。内核区也会拷贝过来，但是 pid 不同。栈空间 pid 也不同。
    
    实际上， 更准确来说，Linux 的 fork() 是通过写时拷贝 （copy-on-write） 实现，写时拷贝是一种可以推迟甚至避免拷贝数据的技术。 内核此时并复制整个进程的地址空间，而是让父子进程共享一个地址空间。只用在需要写入的时候才会复制地址空间，从而使各个进程拥有各自的地址空间。 也就是说，资源的复制是在需要写入的时候才会进行，在此之前，只有以只读的方式共享。 **注意：fork 之后父子进程共享文件** fork 产生的子进程与父进程相同的文件描述符，指向相同的文件表，引用计数增加，共享文件偏移指针。


<a id="orgaec1b14"></a>

## 父子进程关系及GDB调试

-   区别：
    -   1. fork()函数的返回值不同， 父进程中，值大于0，返回子进程的ID.

子进程中，值 = 0

-   2. PCB 中的一些数据， 当前进程的 id pid, 当前进程的父进程的 id pid.
-   3. 信号集。

-   共同点：某些状态下，子进程刚被创建出来，还没有执行任何写数据的操作， 用户区的数据，文件描述符表。

-   父子进程对变量是不是共享的？ 刚开始的时候，是一样的，共享的。如果修改了数据，不共享了。 读时共享（子进程被创建，两个进程没有做任何写的操作），写时拷贝。


<a id="orgee9fbce"></a>

### 调试

使用 GDB 调试的时候，GDB 默认只能跟踪一个进程，可以在 fork 函数调用之前， 通过指令设置 GDB 调试工具跟踪父进程或者跟踪子进程，默认跟踪父进程。

```shell
# 设置调试父进程或者子进程
set follow-fork-mode [parent (default) | child ]

# 设置调试模式
set detach-on-fork [on | off]  # 默认为 on, 表示调试当前进程的时候，其它的进程继续运行， off, 调试当前进程的时候，其它的进程被挂起。

# 查看调试的进程
info inferiors

# 切换当前调试的进程
inferior id

# 使进程脱离 GDB 调试
detach inferiors id

```

-   所使用的调试代码
    
    ```c
    #include <stdio.h>
    #include <unistd.h>
    
    int main() {
      printf("begin: \n");
    
      if (fork() != 0) {
    
        printf("In parent process, pid = %d, ppid = %d\n", getpid(), getppid());
    
        int i;
        for (i = 0; i < 10; i++) {
          printf("i = %d\n", i);
          sleep(i);
        }
      } else {
        printf("In child process, pid = %d, ppid = %d\n", getpid(), getppid());
    
        int j;
        for (j = 0; j < 10; j++) {
          printf("j = %d\n", j);
          sleep(j);
        }
      }
    
      return 0;
    }
    ```


<a id="org37cdcd0"></a>

## exec 函数族

exec 函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容，换句话说，就是在调用进程内部执行一个可执行文件。

exec 函数族的函数执行成功后不会返回，因为调用进程的实体，包括代码段，数据段和堆栈等都已经被新的内容取代，只留下进程 ID 等一些表面上的信息任然保持原样， 看上去旧的驱壳，却已经注入了新的灵魂。只有调用失败了，它们才会返回 -1， 从原程序的调用点接着往下执行。

-   例子程序
    
    ```c
    /**
    
       #include <unistd.h>
    
    
       int execl(const char *pathname, const char *arg, ...);
       int execlp(const char *file, const char *arg, ...);
       int execle(const char *pathname, const char *arg, ...);
       int execv(const char *pathname, char *const argv[]);
       int execvp(const char *file, char *const argv[]);
       int execvpe(const char *file, char *const argv[],
       char *const envp[]);
    
       file, 主要执行的可执行文件的文件名。会到环境变量中查找可执行文件。
       arg：
       执行可执行的间的参数列表，第一个参数一般是可执行文件的名称，参数列表以空（NULL）结尾。
    
       返回值： 只有在出错的时候才有返回值，并设置 errno,
       如果调用成功，没有返回值。
    
    
    
    */
    
    #include <stdio.h>
    #include <unistd.h>
    
    int main() {
    
      // 创建一个子进程，在子进程中执行 exec 函数族中的函数。
    
      pid_t pid = fork();
      if (pid > 0) {
        printf("I am parent process, pid = %d, ppid = %d\n", getpid(), getppid());
        sleep(1);
      } else if (pid == 0) {
        // child
        //    execl("hello", "hello", NULL);
    
        /* execl("ps", "ps", "aux", NULL); */
        /* perror("execl"); */
    
        execlp("ps", "ps", "aux", NULL);
    
        printf("I am child process, pid = %d\n", getpid());
      }
    
      for (int i = 0; i < 4; i++) {
        printf("i = %d, pid = %d\n", i, getpid());
      }
      return 0;
    }
    ```

-   l(list) 参数地址列表， 以空指针结尾。
-   v(vector)存有各参数地址的指针数组的地址
-   p(path) 按 PATH 环境变量指定的目录搜索可执行文件
-   e(environment) 存有环境变量字符串地址的指针数组的地址。
    
    ```c
    #include <unistd.h>
    #include <stdio.h>
    
    int main(){
      char * argv []  = {"ps", "aux", NULL};
      execv("/bin/ps", argv);
    
    
      // int execve()
      char * envp[] = {"/home/aa", "/home/bb/"};
    
      return 0;
    }
    ```


<a id="org1a4c685"></a>

## 进程控制


<a id="orgd33563b"></a>

### 进程退出

```c
#include <stdlib.h>
void exit(int status);

#include <unistd.h>
void _exit(int status);
```

-   进程允许
    -   exit() 调用退出处理函数， 刷新I/O缓冲，关闭文件描述符
    -   \_exit() 调用 \_exit() 系统调用， 进程终止运行


<a id="orgb232b6c"></a>

### 孤儿进程

-   父进程运行结束，但子进程还在运行 （未运行结束）， 这样的子进程就称为孤儿进程 （Oraphan Process）。

-   每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init, 而 init 进程会循环地 wait() 它的已经退出的子进程。

-   孤儿进程并不会有什么危害。


<a id="orgcf50b7e"></a>

### 僵尸进程

每个进程结束之后，都会释放自己地址空间中的用户区数据，内核区的 PCB 没有办法自己释放掉，需要父进程去释放。 进程终止时，父进程尚未回收，子进程残留资源 PCB 存放于内核中，变成僵尸 Zombie 进程。 僵尸进程不能被 kill -9 杀死。

这样就会导致一个问题，如果父进程不调用 wait() 或 waitpid() 的话，那么保留的那段信息就不会释放，其进程号就会一直被占用， 但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，应当避免。

在每个进程退出的时候，内核释放该进程所有的资源，包括打开的文件，占用的内存等。但是任然为其保留一定的信息，这些信息主要指进程控制块 PCB 的信息 （包括进程号，退出状态，运行时间等）。

-   父进程可以通过调用 wait 或 wait pid 得到它的退出状态同时彻底清除掉这个进程
-   wait() 和 waitpid() 函数的功能不一样，区别在于 wait() 函数会阻塞，waitpid() 可以设置不阻塞，还可以指定等待那个子进程结束。

注意： 一次 wait 或 waitpid 调用只能清理一个子进程，清理多个子进程应用使用循环。


<a id="org3b434e7"></a>

## wait 函数

-   例子程序

```c
/**

   NAME
   wait, waitpid, waitid - wait for process to change state

   SYNOPSIS
   #include <sys/types.h>
   #include <sys/wait.h>

   pid_t wait(int *wstatus);
   功能：
   等待任意一个子进程结束，如果任意一个子进程结束了，此函数会回收子进程。 参数：
   进程退出时的状态信息，传入 int 类型的地址， 传出参数。 返回值：
   - 成功，返回被回收的子进程的 id.
   - 失败， -1， （所有的子进程都结束，调用函数失败）

   调用 wait
   函数的进程会被挂起（阻塞），知道它的一个子进程退出或者收到一个不能被忽略的信号时，
   如果没有子进程了，函数立刻返回，返回 -1，
   如果子进程都结束了，也会立刻返回

   pid_t waitpid(pid_t pid, int *wstatus, int options);


*/

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
  // 有一个父进程，产生5个子进程。
  pid_t pid;

  for (int i = 0; i < 5; i++) {
    pid = fork();
    if (pid == 0)
      break;
  }

  if (pid > 0) {
    // parent
    while (1) {
      sleep(2);
      /* int ret = wait(NULL); */

      int st;
      int ret = wait(&st);
      if (ret == -1)
	break;

      if (WIFEXITED(st)) {
	printf("exit with code: %d\n", WEXITSTATUS(st));
      }

      if (WIFSIGNALED(st)) {
	printf("exit with kill: %d\n", WTERMSIG(st));
      }

      printf("Chile process die, pid = %d\n", ret);

      printf("I am parent process, pid = %d\n", getpid());
    }

  } else if (pid == 0) {
    // child
    while (1) {
      printf("I am child process, pid = %d, ppid = %d\n", getpid(), getppid());
      sleep(1);
    }

    // exit(1);
    exit(0);
    //}
  }
  return 0;
}
```

退出信息相关的宏函数

-   WIFEXITED(status) 非0，进程正常退出
-   WEXITSTATUS(status) 如果上宏为真，获取进程退出的状态（exit 的参数）
-   WIFSIGNALED(status) 非0，进程异常终止
-   WTERMSIG(status) 如果上宏为真，获取使进程终止的信号编号
-   WIFSTOPPED(status) 非0， 进程处于暂停状态
-   WSTOPSIG(status) 如果上宏为真，获取使进程暂停的信号的编号
-   WIFCONTINUED(status) 非0，进程暂停后已经继续运行


<a id="org15ef955"></a>

## waitpid()

所用代码

```c
/**
   NAME
   wait, waitpid, waitid - wait for process to change state

   SYNOPSIS
   #include <sys/types.h>
   #include <sys/wait.h>

   pid_t wait(int *wstatus);

   pid_t waitpid(pid_t pid, int *wstatus, int options);

   功能： 回收指定进程号的子进程，可以设置是否阻塞。
   参数：pid: pid > 0, 表示某个子进程的pid. pid = 0,
   回收当前进程组的所有子进程。 pid = -1, 表示回收所有的子进程，相当于 wait(),
   (最常用)。 pid < -1, 回收某个进程组的组id的绝对值，回收指定进程组中的子进程。
   参数 options 设置阻塞和非阻塞， 0, 阻塞， WNOHANG 非阻塞。

   返回值: >0 返回子进程的id, =0 options = WNOHANG, 表示还有子进程活着，= -1
   表示错误，没有子进程了。

*/
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
  // 有一个父进程，产生5个子进程。
  pid_t pid;

  for (int i = 0; i < 5; i++) {
    pid = fork();
    if (pid == 0)
      break;
  }

  if (pid > 0) {
    // parent
    while (1) {
      printf("I am parent process, pid = %d\n", getpid());

      sleep(2);
      /* int ret = wait(NULL); */

      int st;
      //      int ret = waitpid(-1, &st, 0);

      int ret = waitpid(-1, &st, WNOHANG);
      if (ret == -1) {
	break;
      } else if (ret == 0) {
	// 还有子进程存在
	continue;
      } else if (ret > 0) {

	if (WIFEXITED(st)) {
	  printf("exit with code: %d\n", WEXITSTATUS(st));
	}

	if (WIFSIGNALED(st)) {
	  printf("exit with kill: %d\n", WTERMSIG(st));
	}

	printf("Chile process die, pid = %d\n", ret);
      }

      /* if (ret == -1) */
      /*   break; */

      /* if (WIFEXITED(st)) { */
      /*   printf("exit with code: %d\n", WEXITSTATUS(st)); */
      /* } */

      /* if (WIFSIGNALED(st)) { */
      /*   printf("exit with kill: %d\n", WTERMSIG(st)); */
      /* } */

      /* printf("Chile process die, pid = %d\n", ret); */

      /* printf("I am parent process, pid = %d\n", getpid()); */
    }

  } else if (pid == 0) {
    // child
    while (1) {
      printf("I am child process, pid = %d, ppid = %d\n", getpid(), getppid());
      sleep(1);
    }

    // exit(1);
    exit(0);
    //}
  }
  return 0;
}
```


<a id="org68db3f0"></a>

## 进程间通信


<a id="orgd467f68"></a>

### 进程间通信概念

-   进程是一个独立的资源分配单元，不同的进程（这里所说的进程通常是指用户进程）之间的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源，
-   但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要进行程间通信 （IPC: Inter Processes Communication) .
-   进程间通信的目的：
    -   数据传输： 一个进程需要将它的数据发送给另一个进程。
    -   通知事件： 一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要父进程）。
    -   资源共享： 多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同步机制。
    -   进程控制：有些进程希望完全控制另一个进程的执行（如 Debug 进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

-   同一主机进程间通信
    -   Unix 进程间通信方式
        -   匿名管道
        -   有名管道
        -   信号
    -   System V 进程间通信方式 & POSIX 进程间通信方式
        -   消息队列
        -   共享内存
        -   信号量

-   不同主机（网络）进程间通信
    -   Socket


<a id="orgcbf66d8"></a>

## 匿名管道概述

管道也叫无名（匿名）管道，它是 UNIX 系统 IPC 通信的最古老的形式，所有的 UNIX 系统都支持这种通信机制。

例如： 统计一个目录中文件的数目命令： ls | wc -l 为了执行这个命令， shell 创建了两个进程来分别执行 ls 和 wc。


<a id="orge05c2a7"></a>

### 管道特点

-   管道其实是一个在内核内存中维护的缓冲器，这个缓冲区的存储能力是有限的，不同的操作系统大小不一定相同。
-   管道拥有文件的特质：读操作，写操作，匿名管道没有实体，有名管道有文件实体，但不存储数据。可以按照操作文件的方式对管道进行操作。
-   一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多大。
-   通过管道传递的数据是顺序的，从管道中读取出来的字节顺序和它们被写入管道的顺序是完全一样的。
-   管道中的数据的传递方向是单向的，一端用于写入，一端用于读取，管道是半双工的。
-   从管道读取数据是一次性操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据，在管道中无法使用 lseek() 来随机访问数据。
-   匿名管道只能在具有公共祖先的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用。
-   进程缓冲区是在用户空间，管道缓冲区在内核区。

管道的数据结构 环形队列

-   创建匿名管道

```c
#include <unistd.h>
int pipe(int pipefd[2]);
```

-   查看管道缓冲区大小命令 ulimit -a

-   查看管道缓冲区大小函数

```c
#include <unistd.h>
long fpathconf(int fd, int name);
```


<a id="orgc6d7bdc"></a>

## 父子进程通过匿名管道通信

-   实例程序

```c
/**
   NAME
   pipe, pipe2 - create pipe

   SYNOPSIS
   #include <unistd.h>

   On Alpha, IA-64, MIPS, SuperH, and SPARC/SPARC64; see NOTES
   struct fd_pair {
   long fd[2];
   };
   struct fd_pair pipe();

   On all other architectures
   int pipe(int pipefd[2]);
   功能： 创建一个匿名管道，用来进程间通信。
   参数； int pipefd[2] 这个数组是一个传出参数。pipefd[0]
   对应的是管道的读端，pipefd[1] 对应的是管道的写端。 返回值，成功 0, 失败 -1。

   管道默认是阻塞的，如果管道中没有数据，read 阻塞，如果管道满了， write
   阻塞。 注意： 匿名管道只能用于具有关系的进程之间的通信（父子进程，兄弟进程）。
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  // 子进程发送数据，父进程读取数据并输出

  // 在 fork 之前创建管道 pipe
  int pipefd[2];
  int ret = pipe(pipefd);
  if (-1 == ret) {
    perror("pipe");
    exit(0);
  }

  pid_t pid = fork();
  if (pid > 0) {
    printf("I am parent process, pid = %d", getpid());
    char buf[1024] = {0};

    while (1) {
      int len = read(pipefd[0], buf, sizeof(buf));
      printf("Parent recv ： %s, pid = %d\n", buf, getpid());

      // write
      char *str = "Hello, I am parent process";
      write(pipefd[1], str, strlen(str));
      sleep(1);
    }

  } else if (pid == 0) {

    // sleep(2);
    // child process
    printf("I am child process, pid = %d", getpid());
    char buf[1024] = {0};
    while (1) {
      char *str = "Hello, I am child process";
      write(pipefd[1], str, strlen(str));
      sleep(1);

      int len = read(pipefd[0], buf, sizeof(buf));
      printf("Child recv ： %s, pid = %d\n", buf, getpid());
    }
  }

  return 0;
}
```

-   获取缓冲区大小

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {

  int pipefd[2];
  int ret = pipe(pipefd);

  // 获取管道的大小
  long sz = fpathconf(pipefd[0], _PC_PIPE_BUF);

  printf("pipe size = %ld\n", sz);

  return 0;
}


```


<a id="org8fcb818"></a>

## 匿名管道通信案例

-   without sleep

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <strings.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  // 子进程发送数据，父进程读取数据并输出

  // 在 fork 之前创建管道 pipe
  int pipefd[2];
  int ret = pipe(pipefd);
  if (-1 == ret) {
    perror("pipe");
    exit(0);
  }

  pid_t pid = fork();
  if (pid > 0) {
    printf("I am parent process, pid = %d\n", getpid());

    // 关闭写端
    close(pipefd[1]);

    char buf[1024] = {0};
    while (1) {
      int len = read(pipefd[0], buf, sizeof(buf));
      printf("Parent recv ： %s, pid = %d\n", buf, getpid());

      // write
      /* char *str = "Hello, I am parent process"; */
      /* write(pipefd[1], str, strlen(str)); */
      //
      // sleep(1);
    }

  } else if (pid == 0) {

    // sleep(2);
    // child process
    printf("I am child process, pid = %d\n", getpid());

    // 关闭读端
    close(pipefd[0]);

    char buf[1024] = {0};
    while (1) {
      char *str = "Hello, I am child process";
      write(pipefd[1], str, strlen(str));
      // sleep(1);

      /* int len = read(pipefd[0], buf, sizeof(buf)); */
      /* printf("Child recv ： %s, pid = %d\n", buf, getpid()); */

      /* bzero(buf, sizeof(buf)); */
    }
  }

  return 0;
}
```


<a id="orgfd892d4"></a>

## 管道的读写特点和管道设置为非阻塞

管道的读写特点： 使用管道时，需要注意以下几种特殊的情况（假设都是阻塞I/O操作）

1.  所有的指向管道写端的文件描述符都关闭了（管道写端引用计数为0）。由进程从管道的读端读数据，那么管道中剩余的数据被读取以后再次read会返回0， 就像读到文件末尾一样。

2.  如果有指向管道写端的文件描述符没有关闭（管道的写端引用计数大于0），而持有管道写端的进程也没有往里面写数据，这个时候有进程从管道中读取数据， 剩余的数据被读取，再次read 会阻塞，直到管道中有数据可以读了才读取数据。

3.  如果所有指向管道读端的文件描述符都关闭了（管道的读端引用计数大于0），这个时候有进程向管道中写数据，那么该进程会收到一个信号 SIGPIPE, 通常会导致进程异常终止。

4.  如果有指向管道读端的文件描述符没有关闭（管道的读端引用计数大于0），而持有管道读端的进程也没有从管道中读数据，这时，有进程往管道中写数据， 那么在管道中被写满的时候，再次调用 write 会阻塞，直到管道中有空位置了才能再次写入数据并返回。

总结：读管道，管道中有数据，read 返回实际读到的字节数。管道中无数据，写端全闭，read 返回0，相当于读到文件末尾。写端没有完全关闭，read 阻塞。 写管道，读端全部关闭，进程异常终止 （进程收到 SIGPIPE 信号），管道读端没有全部关闭，管道已满， write 阻塞，管道没有满， write 写入数据，并返回实际的写入字节数。

-   实例代码

```nillangnilswitchesnilflags
nilbody
```


<a id="org6e156c8"></a>

## 有名管道

匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了有名管道（FIFO），也就命名管道，FIFO文件。 有名管道（FIFO）不同于匿名管道之处在于它提供了一个路径名与之关联，以 FIFO 的文件形式存在于文件系统中，并且其打开方式与打开一个普通文件是一样的。 这样即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过FIFO通信，因此 FIFO 不相关的进程也能交换数据。

一旦打开了 FIFO， 就能在它上面使用与操作匿名管道和其他文件系统调用一样的 I/O 系统调用了 （如 read(), write() 和 close()）。与管道一样， FIFP 也有一个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。 FIFO 的名字也是由此而来，先入先出。

有名管道 （）FIFO 和 匿名管道 pipe 有一些特点是相同的，不一样的地方在于：

1.  FIFP 在文件系统中作为一个特殊文件存在，但 FIFO 中的内容却存在在内存中。

2.  当使用 FIFO 的进程退出后，FIFO 文件将继续保存在文件系统中以便以后使用。

3.  FIFO 有名字，不相关的进程可以通过打开有名管道进行通信。

-   通过命令创建有名管道

```shell
mkfifo 名字
```

-   一旦使用 mkfile 创建了一个 FIFO 就可以是使用 open 打开它， 常见的文件 I/O 函数都可用于 fifo。 如 close, read, write, unlink 等。

-   FIFO 严格遵循先进先出 (First in First out)， 对管道及 FIFO 的读总是从开始处返回数据，对它们的写则把数据添加到末尾。 它们不支持诸如 lseek() 等文件定位操作。


<a id="org4f64069"></a>

### 有名管道的注意事项

1.  一个为只读打开一个管道的进程会阻塞，直到另外一个进程为只写打开管道。
2.  一个为只写打开一个管道的进程会阻塞，直到另外一个进程为只读打开管道。

3.  读管道：
    -   管道中有数据： read 返回实际读到的字节数
    -   管道中无数据： 管道写端被全部关闭， read 返回 0， 相当于读到文件末尾。写端没有全部被关闭，read 会阻塞等待。

4.  写管道：
    -   管道读端被全部关闭，进程异常终止（收到一个 SIGPIPE）信号。
    -   管道读端没有全部关闭，管道已经满了， write 会阻塞，管道没有满， write 将数据写入，并返回实际写入的字节数。


<a id="orgb63f9ac"></a>

## 有名管道完成聊天功能

-   进程 A: 以只读的方式打开管道1， 以只写的方式打开管道2， 循环读写数据
-   进程 B: 以只读的方式打开管道2， 以只写的方式打开管道1， 循环写读数据

实例程序

-   chat<sub>a</sub>

```c

#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {

  int ret = access("chat_fifo_1", F_OK);
  if (-1 == ret) {
    printf("fifo do not exist, create fifo.\n");
    ret = mkfifo("chat_fifo_1", 0664);
    if (-1 == ret) {
      perror("mkfifo");
      exit(-1);
    }
  }

  ret = access("chat_fifo_2", F_OK);
  if (-1 == ret) {
    printf("fifo do not exist, create fifo.\n");
    ret = mkfifo("chat_fifo_2", 0664);
    if (-1 == ret) {
      perror("mkfifo");
      exit(-1);
    }
  }

  int fd_r = open("chat_fifo_1", O_RDONLY);
  if (-1 == fd_r) {
    perror("open");
    exit(-1);
  }
  printf("Open chat_fifo_1 success, waiting read...\n");

  int fd_w = open("chat_fifo_2", O_WRONLY);
  if (-1 == fd_w) {
    perror("open");
    exit(-1);
  }
  printf("Open chat_fifo_1 success, waiting write...\n");

  char buf[128];
  while (1) {
    memset(buf, 0, 128);
    fgets(buf, 128, stdin);

    ret = write(fd_w, buf, strlen(buf));
    if (-1 == ret) {
      perror("write");
      exit(-1);
    }

    // 5. read data.
    memset(buf, 0, 128);
    ret = read(fd_r, buf, 128);
    if (ret <= 0) {
      perror("read ");
      break;
    }

    printf("A read buf: %s\n", buf);
  }

  close(fd_r);
  close(fd_w);

  return 0;
}
```

-   chat<sub>b</sub>

```c

#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {

  int ret = access("chat_fifo_1", F_OK);
  if (-1 == ret) {
    printf("fifo do not exist, create fifo.\n");
    ret = mkfifo("chat_fifo_1", 0664);
    if (-1 == ret) {
      perror("mkfifo");
      exit(-1);
    }
  }

  ret = access("chat_fifo_2", F_OK);
  if (-1 == ret) {
    printf("fifo do not exist, create fifo.\n");
    ret = mkfifo("chat_fifo_2", 0664);
    if (-1 == ret) {
      perror("mkfifo");
      exit(-1);
    }
  }

  int fd_w = open("chat_fifo_1", O_WRONLY);
  if (-1 == fd_w) {
    perror("open");
    exit(-1);
  }
  printf("Open chat_fifo_1 success, waiting write...\n");

  int fd_r = open("chat_fifo_2", O_RDONLY);
  if (-1 == fd_r) {
    perror("open");
    exit(-1);
  }
  printf("Open chat_fifo_1 success, waiting read...\n");

  char buf[128];
  while (1) {

    // read data.
    memset(buf, 0, 128);
    ret = read(fd_r, buf, 128);
    if (ret <= 0) {
      perror("read ");
      break;
    }

    printf("B: read buf: %s\n", buf);

    // write data
    memset(buf, 0, 128);
    fgets(buf, 128, stdin);

    ret = write(fd_w, buf, strlen(buf));
    if (-1 == ret) {
      perror("write");
      exit(-1);
    }
  }

  close(fd_r);
  close(fd_w);

  return 0;
}
```
