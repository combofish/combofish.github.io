---
layout: post
title: "[学习笔记] Linux 多进程开发"
date: "2023-04-04 08: 21: 22"
tag: cpp linux system programming multiprocess
---
{::options parse_block_html="true" /}

<!-- TOC -->
* [Linux 多进程开发](#linux-多进程开发)
    * [进程概述](#进程概述)
        * [进程控制块](#进程控制块)
    * [进程的状态转换](#进程的状态转换)
        * [查看进程](#查看进程)
        * [实时显示进程动态](#实时显示进程动态)
        * [杀死进程](#杀死进程)
        * [进程号和相关函数](#进程号和相关函数)
    * [进程创建](#进程创建)
    * [父子进程关系及GDB调试](#父子进程关系及gdb调试)
        * [调试](#调试)
    * [exec 函数族](#exec-函数族)
    * [进程控制](#进程控制)
        * [进程退出](#进程退出)
        * [孤儿进程](#孤儿进程)
        * [僵尸进程](#僵尸进程)
    * [wait 函数](#wait-函数)
    * [waitpid()](#waitpid)
    * [进程间通信](#进程间通信)
        * [进程间通信概念](#进程间通信概念)
    * [匿名管道概述](#匿名管道概述)
        * [管道特点](#管道特点)
    * [父子进程通过匿名管道通信](#父子进程通过匿名管道通信)
    * [匿名管道通信案例](#匿名管道通信案例)
    * [管道的读写特点和管道设置为非阻塞](#管道的读写特点和管道设置为非阻塞)
    * [有名管道](#有名管道)
        * [有名管道的注意事项](#有名管道的注意事项)
    * [有名管道完成聊天功能](#有名管道完成聊天功能)
    * [内存映射](#内存映射)
    * [内存映射（2）](#内存映射2)
        * [使用内存映射实现文件拷贝的功能](#使用内存映射实现文件拷贝的功能)
    * [信号概述](#信号概述)
        * [Linux 信号一览表](#linux-信号一览表)
    * [kill, raise, abort 函数](#kill-raise-abort-函数)
    * [alarm 函数](#alarm-函数)
    * [setitimer 函数](#setitimer-函数)
    * [signal 信号捕捉函数](#signal-信号捕捉函数)
    * [信号集及相关函数](#信号集及相关函数)
        * [阻塞信号集和未决信号集](#阻塞信号集和未决信号集)
        * [示例程序](#示例程序)
    * [sigprocmask 函数使用](#sigprocmask-函数使用)
    * [sigaction 信号捕捉函数](#sigaction-信号捕捉函数)
    * [SIGCHLD 信号](#sigchld-信号)
    * [共享内存](#共享内存)
        * [共享内存的使用步骤](#共享内存的使用步骤)
        * [共享内存相关的函数](#共享内存相关的函数)
        * [共享内存和内存映射的区别](#共享内存和内存映射的区别)
    * [守护进程](#守护进程)
        * [进程组](#进程组)
        * [会话](#会话)
        * [守护进程](#守护进程-1)
        * [守护进程的创建步骤](#守护进程的创建步骤)
        * [示例代码](#示例代码)
<!-- TOC -->

# Linux 多进程开发

## 进程概述

`程序`是包含一系列信息的文件， 这些信息描述了如何在运行时创建一个`进程`：

`进程`： 是正在运行的程序的实例。

`单道程序`：
  即在计算机内存中只允许一个程序运行。

`多道程序`：
是在计算机内存中同时存放几道相互独立的程序，使它们在管理进程控制下，相互穿插运行，
两个或两个以上程序在计算机系统中同时处于开始到结束之间的状态，这些程序共享计算机系统资源。
引入多道程序设计技术的根本目的是为了提高CPU的利用率。

`时间片` timeslice: 
由操作系统内核的调度程序分配给每个进程。

首先，内核会给每个进程分配相等的初始时间片，然后每个进程轮番地执行相应的时间，当所有的进程都处于时间片耗尽的状态时，
内核会重新为每个进程计算并分配时间片，如此往复。

`并行` （parallel）: 

指在同一时刻， 有多余指令在多个指令执行。如：两个队列在同时使用两台咖啡机。

`并发` （concurrency）:

指多个进程指令被快速的轮换执行，使得宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行，只是把时间分成若干段，使得多个进程快速交互的执行。
如：两个队列交替使用一台咖啡机。


### 进程控制块

为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。
内核为每个进程分配一个 PCB(Processing Control Block) `进程控制块`，维护进程相关的信息， Linux内核的进程控制块是 task_struct 结构体。

## 进程的状态转换

+ 三态模型：`运行态`，`阻塞态`，`就绪态`
+ 五态模型：`新建态`，`就绪态`，`运行态`，`阻塞态`，`终止态`。

- `运行态`：进程占有处理器正在运行
- `就绪态`：进程具备运行条件，等待系统分配处理器以便运行。当进程已分配到除CPU以外的所有必要资源后，只要再获得CPU,便可立即执行。在一个系统中就绪状态的进程可能有多个，通常将他们排成一个队列，成为就绪队列。
- `阻塞态`：又称等待(wait)态或者睡眠(sleep)态，指进程不具备运行条件，正在等待某个时间完成。
- `新建态`：进程刚被创建时的状态，尚未进入就绪队列
- `终止态`：进程完成任务到达正常结束点，或出现无法克制的错误而异常终止，或被操作系统以及有终止权的进程所终止时所处的状态。进入终止态的进程以后不在执行，但依然保留在操作系统中等待善后。一旦其他进程完成了对终止态进程的信息抽取之后，操作系统将删除该进程。


### 查看进程

+ ps

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

### 实时显示进程动态

+ top

```shell
  top # 可以在使用 top 命令时加上 -d 来指定显示信息更新的时间间隔， 在 top 命令执行后，可以按以下按键对显示的结果进行排序

  # M    根据内存使用量排序
  # P    根据CPU占有率排序
  # T    根据进程运行时间长短排序
  # U    更具用户名来筛选进程
  # K    输入指定的PID杀死进程
```

### 杀死进程

+ kill

```shell
  kill [-signal] pid
  kill -l # 列出所有信号
  kill -SIGKILL 进程ID
  kill -9 进程ID

  killall name 根据进程名杀死进程
```

### 进程号和相关函数

每个进程都是由`进程号`来标识，其类型为 pid_t （整型）, 进程号的范围： 0~32767, 进程号总是唯一的，但可以重用。
当一个进程终止后，其进程号就可以再次使用。

任何进程（除 init 进程）都是由另一个进程创建，该进程称为被创建进程的父进程，对应的进程号称为`父进程号` (PPID);

进程组是一个或多个进程的集合，他们之间相互关联，进程组可以接收同一终端的各种信息，关联的进程有一个`进程组号` （PGID）。
默认情况下，当前的进程号会当做当前进程组号。

+ 进程号和进程组相关函数。

```c
  pid_t getpid(void);
  pid_t getppid(void);
  pid_t getpgid(pid_t pid);
```

## 进程创建

+ 系统允许一个进程创建新的进程，新进程即为子进程。


```c
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

+ fork 以后， 子进程的`用户区`和父进程一样。`内核区`也会拷贝过来，但是 pid 不同。栈空间、pid 也不同。

实际上， 更准确来说，Linux 的 fork() 是通过`写时拷贝` （copy-on-write） 实现，写时拷贝是一种可以推迟甚至避免拷贝数据的技术。
内核此时并复制整个进程的地址空间，而是让父子进程共享一个地址空间。只用在需要写入的时候才会复制地址空间，从而使各个进程拥有各自的地址空间。
也就是说，资源的复制是在需要写入的时候才会进行，在此之前，只有以只读的方式共享。 

**注意：fork 之后父子进程共享文件描述符。**
fork 产生的子进程与父进程相同的文件描述符，指向相同的文件表，引用计数增加，共享文件偏移指针。


## 父子进程关系及GDB调试

+ 区别：
1. fork()函数的返回值不同， 父进程中，值大于0，返回子进程的ID.
   子进程中，值 = 0
2. PCB 中的一些数据， 当前进程的 id pid, 当前进程的父进程的 id pid.
3. 信号集

+ 共同点：某些状态下，子进程刚被创建出来，还没有执行任何写数据的操作， 用户区的数据，文件描述符表。

+ 父子进程对变量是不是共享的？ 刚开始的时候，是一样的，共享的。如果修改了数据，不共享了。
  读时共享（子进程被创建，两个进程没有做任何写的操作），写时拷贝。

### 调试

使用 GDB 调试的时候，GDB 默认只能跟踪一个进程，可以在 fork 函数调用之前， 通过指令设置 GDB 调试工具跟踪父进程或者跟踪子进程，默认跟踪父进程。

```shell
  # 设置调试父进程或者子进程
  set follow-fork-mode [parent (default) | child ]

  # 设置调试模式
  set detach-on-fork [on | off]  
  # 默认为 on, 表示调试当前进程的时候，其它的进程继续运行， 
  # off, 调试当前进程的时候，其它的进程被挂起。

  # 查看调试的进程
  info inferiors

  # 切换当前调试的进程
  inferior id

  # 使进程脱离 GDB 调试
  detach inferiors id
```


+ 所使用的调试代码

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


## exec 函数族

`exec 函数族`的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容，换句话说，就是`在调用进程内部执行一个可执行文件`。

exec 函数族的函数执行成功后不会返回，因为调用进程的实体，包括代码段，数据段和堆栈等都已经被新的内容取代，
只留下进程 ID 等一些表面上的信息任然保持原样，
看上去旧的驱壳，却已经注入了新的灵魂。只有调用失败了，它们才会返回 -1， 从原程序的调用点接着往下执行。

+ 例子程序

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
       arg：执行可执行的间的参数列表，第一个参数一般是可执行文件的名称，参数列表以空（NULL）结尾。

       返回值： 只有在出错的时候才有返回值，并设置 errno,
       如果调用成功，没有返回值。
    */

    #include <stdio.h>
    #include <unistd.h>

    int main() {
      pid_t pid = fork();  // 创建一个子进程，在子进程中执行 exec 函数族中的函数。
      if (pid > 0) {
        printf("I am parent process, pid = %d, ppid = %d\n", getpid(), getppid());
        sleep(1);
      } else if (pid == 0) {  // child
        // execl("hello", "hello", NULL);
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


+ l(list) 参数地址列表， 以空指针结尾。
+ v(vector)存有各参数地址的指针数组的地址
+ p(path) 按 PATH 环境变量指定的目录搜索可执行文件
+ e(environment) 存有环境变量字符串地址的指针数组的地址。

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

## 进程控制

### 进程退出

```c
  #include <stdlib.h>
  void exit(int status);

  #include <unistd.h>
  void _exit(int status);
```

+ 进程允许
    - exit() 调用退出处理函数， 刷新I/O缓冲，关闭文件描述符
    - _exit() 调用 _exit() 系统调用， 进程终止运行

### 孤儿进程

+ 父进程运行结束，但子进程还在运行 （未运行结束）， 这样的子进程就称为`孤儿进程` （Orphan Process）。

+ 每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init,  而 init 进程会循环地 wait() 它的已经退出的子进程。

+ 孤儿进程并不会有什么危害。


### 僵尸进程

每个进程结束之后，都会释放自己地址空间中的用户区数据，内核区的 PCB 没有办法自己释放掉，需要父进程去释放。
进程终止时，父进程尚未回收，子进程残留资源 PCB 存放于内核中，变成`僵尸 (Zombie) 进程`。
`僵尸进程不能被 kill -9 杀死`。

这样就会导致一个问题，如果父进程不调用 wait() 或 waitpid() 的话，那么保留的那段信息就不会释放，其进程号就会一直被占用，
但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，应当避免。

在每个进程退出的时候，内核释放该进程所有的资源，包括打开的文件，占用的内存等。但是任然为其保留一定的信息，这些信息主要指进程控制块
PCB 的信息 （包括进程号，退出状态，运行时间等）。

+ 父进程可以通过调用 `wait()` 或 `waitpid()` 得到它的退出状态同时彻底清除掉这个进程
+ wait() 和 waitpid() 函数的功能不一样，区别在于 wait() 函数会阻塞，waitpid() 可以设置不阻塞，还可以指定等待那个子进程结束。

注意： 一次 wait 或 waitpid 调用只能清理一个子进程，清理多个子进程应用使用循环。

+ zombie.c
```c
  #include <stdio.h>
  #include <sys/types.h>
  #include <unistd.h>

  int main() {
    pid_t pid = fork();
    if (pid > 0) {
      while (1) {
        printf("I am parent process, pid = %d, ppid = %d\n", getpid(), getppid());
        sleep(1);
      }
    } else {
      printf("I am child process, pid = %d, ppid = %d\n", getpid(), getppid());
    }
    for (int i = 0; i < 5; i++) {
      printf("i = %d, pid = %d, ppid = %d\n", i, getpid(), getppid());
    }  
    return 0;
  }
```

## wait 函数

+ 例子程序

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
    pid_t pid;  // 有一个父进程，产生5个子进程。
    for (int i = 0; i < 5; i++) {
      pid = fork();
      if (pid == 0)
        break;
    }

    if (pid > 0) { // parent
      while (1) {
        sleep(2);
        /* int ret = wait(NULL); */
        int st;
        int ret = wait(&st);
        if (ret == -1) break;
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
+ WIFEXITED(status) 非0，进程正常退出
+ WEXITSTATUS(status) 如果上宏为真，获取进程退出的状态（exit 的参数）
+ WIFSIGNALED(status) 非0，进程异常终止
+ WTERMSIG(status) 如果上宏为真，获取使进程终止的信号编号
+ WIFSTOPPED(status) 非0， 进程处于暂停状态
+ WSTOPSIG(status)  如果上宏为真，获取使进程暂停的信号的编号
+ WIFCONTINUED(status) 非0，进程暂停后已经继续运行

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
    pid_t pid;  // 有一个父进程，产生5个子进程。

    for (int i = 0; i < 5; i++) {
      pid = fork();
      if (pid == 0) break;
    }

    if (pid > 0) {  // parent
      while (1) {
          printf("I am parent process, pid = %d\n", getpid());
          sleep(2);
          /* int ret = wait(NULL); */
          int st;
          //      int ret = waitpid(-1, &st, 0);
        
          int ret = waitpid(-1, &st, WNOHANG);
          if (ret == -1) {
            break;
          } else if (ret == 0) {  // 还有子进程存在
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
    } else if (pid == 0) {  // child
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

## 进程间通信

### 进程间通信概念

+ 进程是一个独立的资源分配单元，不同的进程（这里所说的进程通常是指用户进程）之间的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源，
+ 但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要进行`进程间通信` （`IPC`: Inter Processes Communication) .
+ 进程间通信的目的：
    - 数据传输： 一个进程需要将它的数据发送给另一个进程。
    - 通知事件： 一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要父进程）。
    - 资源共享： 多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同步机制。
    - 进程控制：有些进程希望完全控制另一个进程的执行（如 Debug 进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。


+ 同一主机进程间通信
    + Unix 进程间通信方式
        - `匿名管道`
        - `有名管道`
        - `信号`
    + System V 进程间通信方式 & POSIX 进程间通信方式
        - `消息队列`
        - `共享内存`
        - `信号量`

+ 不同主机（网络）进程间通信
    - `Socket`


## 匿名管道概述

`管道`也叫无名（匿名）管道，它是 UNIX 系统 IPC 通信的最古老的形式，所有的 UNIX 系统都支持这种通信机制。

例如： 统计一个目录中文件的数目命令： `ls | wc -l` 为了执行这个命令， shell 创建了两个进程来分别执行 ls 和 wc。

### 管道特点

+ 管道其实是一个在内核内存中维护的缓冲器，这个缓冲区的存储能力是有限的，不同的操作系统大小不一定相同。
+ 管道拥有文件的特质：读操作，写操作，匿名管道没有实体，有名管道有文件实体，但不存储数据。可以按照操作文件的方式对管道进行操作。
+ 一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多大。
+ 通过管道传递的数据是顺序的，从管道中读取出来的字节顺序和它们被写入管道的顺序是完全一样的。
+ 管道中的数据的传递方向是单向的，一端用于写入，一端用于读取，管道是半双工的。
+ 从管道读取数据是一次性操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据，在管道中无法使用 lseek() 来随机访问数据。
+ 匿名管道只能在具有公共祖先的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用。
+ 进程缓冲区是在用户空间，管道缓冲区在内核区。

管道的数据结构 `环形队列`

+ 创建匿名管道
```c
  #include <unistd.h>
  int pipe(int pipefd[2]);
```

+ 查看管道缓冲区大小命令
```shell
ulimit -a
```  

+ 查看管道缓冲区大小函数
```c
  #include <unistd.h>
  long fpathconf(int fd, int name);
```

## 父子进程通过匿名管道通信

+ 实例程序

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
  ,*/

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

+ 获取缓冲区大小

```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>

  int main() {
    int pipefd[2];
    int ret = pipe(pipefd);
    
    long sz = fpathconf(pipefd[0], _PC_PIPE_BUF);  // 获取管道的大小
    printf("pipe size = %ld\n", sz);
    return 0;
  }
```

## 匿名管道通信案例

+ without sleep

```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <strings.h>
  #include <sys/types.h>
  #include <unistd.h>

  int main() {  // 子进程发送数据，父进程读取数据并输出
    int pipefd[2];  // 在 fork 之前创建管道 pipe
    int ret = pipe(pipefd);
    if (-1 == ret) {
      perror("pipe");
      exit(0);
    }

    pid_t pid = fork();
    if (pid > 0) {
      printf("I am parent process, pid = %d\n", getpid());
      close(pipefd[1]); // 关闭写端
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
      close(pipefd[0]);  // 关闭读端
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

## 管道的读写特点和管道设置为非阻塞

管道的读写特点：
使用管道时，需要注意以下几种特殊的情况（假设都是阻塞I/O操作）
1. 所有的指向管道写端的文件描述符都关闭了（管道写端引用计数为0）。由进程从管道的读端读数据，那么管道中剩余的数据被读取以后再次read会返回0，
   就像读到文件末尾一样。
2. 如果有指向管道写端的文件描述符没有关闭（管道的写端引用计数大于0），而持有管道写端的进程也没有往里面写数据，这个时候有进程从管道中读取数据，
   剩余的数据被读取，再次read 会`阻塞`，直到管道中有数据可以读了才读取数据。
3. 如果所有指向管道读端的文件描述符都关闭了（管道的读端引用计数大于0），这个时候有进程向管道中写数据，那么该进程会收到一个信号 `SIGPIPE`,
   通常会导致进程异常终止。
4. 如果有指向管道读端的文件描述符没有关闭（管道的读端引用计数大于0），而持有管道读端的进程也没有从管道中读数据，这时，有进程往管道中写数据，
   那么在管道中被写满的时候，再次调用 write 会`阻塞`，直到管道中有空位置了才能再次写入数据并返回。

总结：
- `读管道`，管道中有数据，read 返回实际读到的字节数。管道中无数据，写端全闭，read 返回0，相当于读到文件末尾。写端没有完全关闭，read 阻塞。
- `写管道`，读端全部关闭，进程异常终止 （进程收到 SIGPIPE 信号），管道读端没有全部关闭，管道已满， write 阻塞，管道没有满，
write 写入数据，并返回实际的写入字节数。

+ 实例代码

```c
  #include <fcntl.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <sys/types.h>
  #include <unistd.h>

  /**
  设置管道非阻塞
  fcntl(fd[0], F_GETFL);  // 获取原来的 flag,;
  flags |= O_NONBLOCK;  // 修改 flag 的值
  fcntl(fd[0], F_SETFL, flags); // 设置新的 flag.
  */
  int main() {  // 子进程发送数据,父进程读取数据并输出
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
		close(pipefd[1]);
		int flages = fcntl(pipefd[0], F_GETFL);
		flages |= O_NONBLOCK;
		fcntl(pipefd[0], F_SETFL, flages);
		char buf[1024] = {0};
		while (1) {
			  int len = read(pipefd[0], buf, sizeof(buf));
			  printf("len = %d ", len);
			  printf("Parent recv ： %s, pid = %d\n", buf, getpid());
			  memset(buf, 0, 1024);
			  sleep(1);
			  // write
			  /* char *str = "Hello, I am parent process"; */
			  /* write(pipefd[1], str, strlen(str)); */
			  /* sleep(1); */
		      }
	    } else if (pid == 0) {
		// sleep(2);
		// child process
		printf("I am child process, pid = %d", getpid());
		close(pipefd[0]);
		char buf[1024] = {0};
		while (1) {
			  char *str = "Hello, I am child process";
			  write(pipefd[1], str, strlen(str));
			  sleep(3);

			  /* int len = read(pipefd[0], buf, sizeof(buf)); */
			  /* printf("Child recv ： %s, pid = %d\n", buf, getpid()); */
		      }
	    }
	    return 0;
  }
```

## 有名管道

匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了`有名管道`（FIFO），也就命名管道，FIFO文件。
有名管道（FIFO）不同于匿名管道之处在于它提供了一个路径名与之关联，以 FIFO 的文件形式存在于文件系统中，并且其打开方式与打开一个普通文件是一样的。
这样即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过FIFO通信，因此 FIFO 不相关的进程也能交换数据。

一旦打开了 FIFO， 就能在它上面使用与操作匿名管道和其他文件系统调用一样的 I/O 系统调用了 （如 `read()`, `write()` 和 `close()`）。
与管道一样， FIFO 也有一个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。 FIFO 的名字也是由此而来，先入先出。

有名管道 (FIFO) 和 匿名管道 pipe 有一些特点是相同的，不一样的地方在于：
1. FIFO 在文件系统中作为一个特殊文件存在，但 FIFO 中的内容却存在在`内存`中。
2. 当使用 FIFO 的进程退出后，FIFO 文件将继续保存在文件系统中以便以后使用。
3. FIFO `有名字`，不相关的进程可以通过打开有名管道进行通信。

- 通过命令创建有名管道

```shell
  mkfifo 名字
```

- 一旦使用 mkfile 创建了一个 FIFO 就可以是使用 open 打开它， 常见的文件 I/O 函数都可用于 fifo。 如 close, read, write, unlink 等。
- FIFO 严格遵循先进先出 (First in First out)， 对管道及 FIFO 的读总是从开始处返回数据，对它们的写则把数据添加到末尾。
  它们不支持诸如 lseek() 等文件定位操作。

### 有名管道的注意事项

1. 一个为只读打开一个管道的进程会阻塞，直到另外一个进程为只写打开管道。
2. 一个为只写打开一个管道的进程会阻塞，直到另外一个进程为只读打开管道。

+ `读管道`：
    - 管道中有数据： read 返回实际读到的字节数
    - 管道中无数据： 管道写端被全部关闭， read 返回 0， 相当于读到文件末尾。写端没有全部被关闭，read 会阻塞等待。

+ `写管道`：
    - 管道读端被全部关闭，进程异常终止（收到一个 SIGPIPE）信号。
    - 管道读端没有全部关闭，管道已经满了， write 会阻塞，管道没有满， write 将数据写入，并返回实际写入的字节数。

## 有名管道完成聊天功能

+ 进程 A: 以只读的方式打开管道1， 以只写的方式打开管道2， 循环读写数据
+ 进程 B: 以只读的方式打开管道2， 以只写的方式打开管道1， 循环写读数据

实例程序
+ chat_a

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
    printf("Open chat_fifo_2 success, waiting write...\n");

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

+ chat_b

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

## 内存映射

`内存映射`： (Memory-mapped I/O) 是将磁盘文件的数据映射到内存，用户修改内存就能修改磁盘文件。

内存映射相关的系统调用：

```c
// mmap, munmap - map or unmap files or devices into memory
#include <sys/mman.h>
     
void *mmap(void *addr, size_t length, int prot, int flags,
int fd, off_t offset);
int munmap(void *addr, size_t length);
```

+ 实例代码

```c
  /**
     NAME
     mmap, munmap - map or unmap files or devices into memory

     SYNOPSIS
     #include <sys/mman.h>

     void *mmap(void *addr, size_t length, int prot, int flags,
     int fd, off_t offset);

     -  功能： 将一个文件或者设备的数据映射到内存中。
     -  参数：
     - void *addr: NULL, 由内核指定
     - length: 要映射的数据的长度，
     这个值不能为0，建议使用文件的长度；获取文件的长度可以用 stat 或 lseek 函数。
     - prot： 对我们申请的内存映射区的操作权限
     - PROT_EXEC  Pages may be executed.
     - PROT_READ  Pages may be read.
     - PROT_WRITE Pages may be written.
     - PROT_NONE  Pages may not be accessed.
     - 要操作映射内存，必须要有读的权限。
     - flags:
     - MAP_SHARED ：
     映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项。
     - MAP_SHARED_VALIDATE (since Linux 4.15)
     - MAP_PRIVATE：
     不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件。（copy
     on write）
     - fd: 需要映射的那个文件的文件描述符
     - 通过 open 得到，open 的是一个磁盘文件，注意文件的大小要大于0，open
     执行的权限不能和 prot 的参数权限有冲突。
     - offset: 偏移量，必须指定4k 的整数倍，0表示不偏移。
     - 返回值： 返回创建内存的首地址， 失败返回 MAP_FAILED, (void *) -1,
     并设置 errno。

     int munmap(void *addr, size_t length);
     - 功能： 释放内存映射
     - 参数：
     - addr : 要释放的内存的首地址。
     - length ： 要释放的内存的大小，要和 mmap 函数中的 Length
     参数一致。
  */
  /**
     使用内存映射实现进程间通信：
     1. 有关系的进程（父子进程）
     -
     还没有子进程的时候，通过唯一的父进程，先创建内存映射区，之后创建子进程，父子进程共享创建的内存映射区。

     2. 没有关系的进程间通信
     - 准备一个大小不是 0 的磁盘文件，
     - 进程1 通过磁盘文件创建内存映射区，得到一个操作这块内存的指针
     - 进程2 通过磁盘文件创建内存映射区，得到一个操作这块内存的指针
     - 使用内存映射区通信

     注意： 内存映射区通信，是非阻塞的。
  */
  #include <fcntl.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <sys/mman.h>
  #include <sys/types.h>
  #include <sys/wait.h>
  #include <unistd.h>

  int main() {
    int fd = open("test.txt", O_RDWR);  // 1. 打开一个文件
    int size = lseek(fd, 0, SEEK_END);  // 获取文件的大小

    // 创建内存映射区
    void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (ptr == MAP_FAILED) {
      perror("mmap");
      exit(0);
    }
    
    pid_t pid = fork();  // 创建子进程
    if (pid > 0) {  // parent process
      wait(NULL);
      char buf[64];
      strcpy(buf, (char *)ptr);
      printf("read data: %s\n", buf);
    } else {  // child process
      strcpy((char *)ptr, "Nihao, a son!!");
    }
    munmap(ptr, size);
    close(fd);
    return 0;
  }
```

## 内存映射（2）

+ 如果对 mmap 的返回值 （ptr）做++操作 （ptr++）, munmap 是否能够成功？
 
```c
void *ptr = mmap(...); ptr++;   // 可以对齐进行 ++ 操作， munmap(ptr,len); 不能够正确释放。
```

+ 如果 open 时 O_RDONLY, mmap 时 prot 参数指定 `PROT_READ | PROT_WRITE` 会怎么样？
  
错误， 返回 MAP_FAILED。 open() 函数中的权限建议和 prot 参数的权限保持一致。

+ 如果文件偏移量为 1000 会怎么样？

偏移量必须是 4 * 1024 = 4k 的整数倍，返回错误 MAP_FAILED。

+ mmap 什么情况下会调用失败？
    - 第二个参数，length = 0,
    - 第三个参数, prot,
        - 只指定了写权限；
        - prot `PROT_READ | PROT_WRITE`, 第5个参数 fd 通过 open 函数指定的 O_RDONLY / O_WRONLY

+ 可以 open 的时候 O_CREAT 一个新文件来创建映射区吗？
    - 可以的， 但是创建的文件的大小如果为0 的话，肯定不行
    - 可以对新的文件进行扩展。
        - lseek()
        - truncate()

+ mmap 后关闭文件描述符， 对 mmap 文件有没有影响?

```c
int fd = open("XX");
mmap(,,,,fd, 0);
close(fd);
```

映射区还是存在，创建映射区的 fd 被关闭，没有任何影响。

+ 对 ptr 越界操作会怎样？

```c
void *ptr = mmap(NULL, 100, ...);  // 越界操作，操作的是非法内存， -> 段错误。
```  

### 使用内存映射实现文件拷贝的功能

```c
  // 使用内存映射实现文件拷贝的功能
  /**
     1. 对原始的文件进行内存映射
     2. 创建一个新的文件，（扩展该文件）
     3. 把新文件的数据映射到内存中，
     4. 通过内存拷贝，将第一个文件的内存数据拷贝到新的文件中
     5. 释放资源。
  */

  #include <fcntl.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <sys/mman.h>
  #include <unistd.h>
  #include <sys/types.h>
  #include <sys/stat.h>
  
  int main() {
    int fd_ori = open("test.txt", O_RDWR);
    if (fd_ori == -1) {
      perror("open");
      exit(0);
    }

    int size_ori = lseek(fd_ori, 0, SEEK_END);
    // 2. 创建新的文件
    int fd1 = open("cpy_from_test.txt", O_RDWR | O_CREAT, 0664);
    if (fd1 == -1) {
      perror("open");
      exit(0);
    }

    // 对新文件进行扩展
    truncate("cpy_from_test.txt", size_ori);
    write(fd1, " ", 1);

    // 3. 分别内存映射
    void *ptr_ori =
      mmap(NULL, size_ori, PROT_READ | PROT_WRITE, MAP_SHARED, fd_ori, 0);
    if (ptr_ori == MAP_FAILED) {
      perror("mmap");
      exit(0);
    }

    void *ptr1 = mmap(NULL, size_ori, PROT_READ | PROT_WRITE, MAP_SHARED, fd1, 0);
    if (ptr1 == MAP_FAILED) {
      perror("mmap");
      exit(0);
    }

    memcpy(ptr1, ptr_ori, size_ori);
    munmap(ptr1, size_ori);
    munmap(ptr_ori, size_ori);
    
    close(fd1);
    close(fd_ori);
    return 0;
  }
```

+ 匿名映射

```c
  /**
     匿名映射， 不需要文件实体实现进程的内存映射
     用于父子进程。
  */

  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <sys/mman.h>
  #include <sys/stat.h>
  #include <sys/types.h>
  #include <sys/wait.h>
  #include <unistd.h>

  int main() {
    int length = 4;
    void *ptr = mmap(NULL, length, PROT_READ | PROT_WRITE,
		     MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (ptr == MAP_FAILED) {
      perror("mmap");
      exit(0);
    }

    // 父子进程通信
    pid_t pid = fork();
    if (pid > 0) {
      strcpy((char *)ptr, "Hello World");
      wait(NULL);
    } else if (pid == 0) {
      sleep(1);
      printf("%s\n", (char *)ptr);
    }

    int ret = munmap(ptr, length);
    if (-1 == ret) {
      perror("munmap");
      exit(0);
    }
    return 0;
  }
```

## 信号概述

`信号`是 Linux 进程间通信的最古老的方式之一，是事件发生时对进程的通知机制，有时也称之为`软件中断`，
它是在软件层次上对中断机制的一种模拟，是一种`异步通信`的方式。
信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某一个突发事件。

发往进程的诸多信号，通常都是源于内核。引发内核为进程产生信号的各类事件如下：
  - 对于前台进程，用户可以通过输入特殊的终端字符来给它发送信号。比如输入 Ctrl+C 通常会给进程发送一个中断信号。
  - 硬件发生异常，即硬件检测到一条错误条件并通知内核，随机再由内核发送相应信号给相关进程。
    比如执行一条异常的机器语言指令，诸如被 0 除，或者引用了无法访问的内存区域。
  - 系统状态变化，比如 alarm 定时器到期引起 SIGLARM 信号， 进程执行 CPU 时间超限，或者该进程的某个子进程退出。
  - 运行 kill 命令或者调用 kill 函数。

使用信号的两个`目的`：
  - 让进程知道已经发生了一个特定的事情。
  - 强迫进程执行它自己的代码中的信号处理程序。

信号的`特点`：
  - 简单
  - 不能携带大量信息
  - 满足某个特定条件才能发送
  - 优先级比较高

查看系统定义的信号列表: `kill -l` ,前 31 个信号为常规信号，其余为实时信号。

### Linux 信号一览表

| 编号 | 信号名称    | 对应事件                                                | 默认动作            |
|----|---------|-----------------------------------------------------|-----------------|
| 2  | SIGINT  | 当用户按下了 <Ctrl+C> 组合键时，用户终端正在运行中的由该终端启动的程序发出此信号       | 终止进程            |
| 3  | SIGQUIR | 当用户按下 <Ctrl+\> 组合键时产生该信号，用户终端向正在运行中的由该终端启动的程序发出那些信号 | 终止进程            |
| 9  | SIGKILL | 无条件终止进程。该信号不能被忽略，处理和阻塞                              | 终止进程，可以杀死任何进程   |
| 11 | SIGSEGV | 指示进程进行了无效内存访问（段错误）                                  | 终止进程并产生 core 文件 |
| 13 | SIGPIPE | Broken pipe 向一个没有读端的管道写数据                           | 终止进程            |
| 17 | SIGCHLD | 子进程结束时，父进程会收到这个信号                                   | 忽略这个信号          |
| 18 | SIGCONT | 如果进程已经停止，则使其继续运行                                    | 继续/ 忽略          |
| 19 | SIGSTOP | 停止进程的执行。信号不能被忽略，处理阻塞                                | 为终止进程           |

+ 查看信号的详细信息 `man 7 signal`
+ 信号的 5 种 默认处理动作
    - Term 终止进程
    - Ign  当前进程忽略掉这个信号
    - Core 终止进程，并生成一个 Core 文件
    - Stop 暂停当前进程
    - Cont 继续执行当前被暂停的进程
+ 信号的几种状态：`产生`，`未决`，`抵达`
+ `SIGKILL` 和 `SIGSTOP` 信号不能被捕捉，阻塞或者忽略，只能执行默认动作。

## kill, raise, abort 函数

+ 代码示例

```c
  /**

     NAME
     kill - send signal to a process

     SYNOPSIS
     #include <signal.h>
     #include <sys/types.h>

     int kill(pid_t pid, int sig);
     - 功能： 给某个进程 pid 或者进程组, 发送某个信号 sig。
     - 参数：
     - pid ： 发送给进程的 pid,
     - > 0 : 将信号发送给指定的进程
     - = 0 : 将信号发送给当前的进程组
     - = -1 ： 将信号发送给每一个有权限接收这个信号的进程
     - < -1 : 这个 pid = 某个进程组的 ID 取反。
     - sig : 发送给进程的信号的编号或者宏值。0 表示不发送任何信号。


     int raise(int sig);
     - 功能： 给当前进程发送信号
     - 参数： sig 要发送的信号
     - 返回值： 成功 0， 失败 非0。


     void abort(void);
     - 功能： 发送 SIGABRT 信号给当前的进程，杀死当前的进程。
  */

  #include <signal.h>
  #include <stdio.h>
  #include <sys/types.h>
  #include <unistd.h>

  int main() {
    pid_t pid = fork();
    if (pid > 0) { // parent
      printf("parent process: \n");
      sleep(2);
      printf("kill child process now! \n");
      kill(pid, SIGINT);
    } else if (pid == 0) {  // child
      int i = 0;
      for (i = 0; i < 5; i++) {
        printf("child process: i = %d\n", i);
        sleep(1);
      }
    }
    return 0;
  }
```

## alarm 函数

+ 代码实例

```c
  /**
     NAME
     alarm - set an alarm clock for delivery of a signal

     SYNOPSIS
     #include <unistd.h>

     unsigned int alarm(unsigned int seconds);
     - 功能 ： 设置定时器（闹钟）， 函数调用开始倒计时，倒计时为 0
     时，函数会给当前的进程发送一个信号 SIGALRM.
     - 参数： seconds: 倒计时的时长， 单位 ： 秒， 如果参数为0，
     定时器无效，不进行倒计时。取消一个定时器，通过 alarm(0);
     - 返回值：
     - 之前没有定时器： 返回0，
     - 之前有定时器，返回之前的定时器剩余时间。

     - SIGALARM：默认终止当前的进程，每一个进程都有且只有唯一的一个定时器。
     alarm(10); -> 0;
     sleep(1);
     alarm(5); -> 9;
  
     alarm(100); 该函数是不阻塞的。
  */
  #include <stdio.h>
  #include <unistd.h>

  int main() {
    int seconds = alarm(5);
    printf("seconds = %d\n", seconds); // 0

    sleep(2);
    seconds = alarm(2);
    printf("second = %d\n", seconds); // 3
    while(1){ }
    return 0;
  }
```

+ 计算电脑1秒可以计算多少个数

```c
  // 1 秒中电脑能数多少个数
  #include <stdio.h>
  #include <unistd.h>

  int main() {
    alarm(1);
    int i = 0;
    while (1) {
      printf("%d\n", i++);
    }
    return 0;
  }
```

```shell
./a.out >> a.txt
```

+ 实际的时间 = 内核时间 + 用户时间 + 消耗的时间
+ 进行文件 IO 操作的时候比较浪费时间

`定时器`，与进程的状态无关（自然定时法），无论进程处于什么状态，alarm 都会计时。

## setitimer 函数

```c
  /**
     NAME
     getitimer, setitimer - get or set value of an interval timer

     SYNOPSIS
     #include <sys/time.h>

     int getitimer(int which, struct itimerval *curr_value);
     int setitimer(int which, const struct itimerval *new_value,
     struct itimerval *old_value);

     - 功能： 设置定时器（闹钟）， 可以替代 alarm 函数， 精度在 us,
     可以实现周期性的定时。
     - 参数：
     - which : 定时器以什么时间计时
     ITIMER_REAL: 真实时间， 时间到达，发送 SIGALARM （常用）
     ITIMER_VIRTUAL: 用户时间， 时间到达，发送 SIGVTALRM
     ITIMER_PROF:
     以该进程在用户态和内核态下所消耗的时间来计算，时间到了发送 SIGPROP
     - new_value： 设置定时器的属性。
     Timer values are defined by the following structures:

     struct itimerval {       // 定时器的结构体
     struct timeval it_interval; // 每个阶段的时间，间隔时间
     struct timeval it_value; // 延迟多长时间执行定时器
     };

     struct timeval { // 时间的结构体
     time_t      tv_sec; // 秒数
     suseconds_t tv_usec; // 微秒
     };

     - old_value ： 记录上一次的定时的时间参数。一般不使用。

     - 返回值
     RETURN VALUE
     On success, zero is returned.  On error, -1 is returned, and errno is set
     appropriately.
  */

  #include <stdio.h>
  #include <stdlib.h>
  #include <sys/time.h>

  // 过 3 秒钟以后， 每隔 2 秒钟定时一次。
  int main() {
    struct itimerval new_value;
    // set
    // 设置间隔时间
    new_value.it_interval.tv_sec = 2;
    new_value.it_interval.tv_usec = 0;

    // 设置延迟的时间
    new_value.it_value.tv_usec = 0;
    new_value.it_value.tv_sec = 3;

    int ret = setitimer(ITIMER_REAL, &new_value, NULL);
    printf("定时器开始了\n");
    if (ret == -1) {
      perror("setitimer");
      exit(0);
    }
    getchar();
    return 0;
  }
```

## signal 信号捕捉函数

```c
  /**
     NAME
     signal - ANSI C signal handling

     SYNOPSIS
     #include <signal.h>

     typedef void (*sighandler_t)(int);

     sighandler_t signal(int signum, sighandler_t handler);
     - 功能： 设置某个信号的捕捉行为
     - 参数：
     - signum: 要捕捉的信号
     - handler: 捕捉到信号要如何处理
     - SIG_IGN: 忽略信号
     - SIG_DEF: 使用信号默认的行为。
     - 回调函数：
     这个函数是内核调用，程序员只负责去写，捕捉到信号后如何去处理信号。
     - 返回值：RETURN VALUE
     signal() returns the previous value of the signal handler, or SIG_ERR on
     error.  In the event of an error, errno is set to indicate the cause.
  
     回调函数：
     需要程序员去实现，提前准备好的，函数的类型根据实际需求，看函数指针的定义
     不是程序员调用，而是当信号产生，由内核调用
     函数指针是实现回调的手段，函数实现之后，将函数名放到函数指针的位置就可以了。
  */

  #include <signal.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <sys/time.h>

  void my_alarm(int num) {
    printf("cat signal = %d\n", num);
    printf("xxxxxxx\n");
  }
  
  int main() {  // 过 3 秒钟以后， 每隔 2 秒钟定时一次。

    // 注册信号捕捉
    //  signal(SIGALRM, SIG_IGN);
    //  signal(SIGALRM, SIG_DFL);

    __sighandler_t res= signal(SIGALRM, my_alarm);
    if (res == SIG_ERR) {
      perror("signal");
      exit(0);
    }

    struct itimerval new_value;

    // set
    // 设置间隔时间
    new_value.it_interval.tv_sec = 2;
    new_value.it_interval.tv_usec = 0;

    // 设置延迟的时间
    new_value.it_value.tv_usec = 0;
    new_value.it_value.tv_sec = 3;

    int ret = setitimer(ITIMER_REAL, &new_value, NULL);
    printf("定时器开始了\n");

    if (ret == -1) {
      perror("setitimer");
      exit(0);
    }

    getchar();
    return 0;
  }
```

+ sigaction()

## 信号集及相关函数

许多信号相关的系统调用都需要能表示一组不同的信号，多个信号可使用一个称之为`信号集`的数据结构来表示，其系统数据类型为 sigset_t。

在 PCB 中有两个非常重要的信号集。一个称之为`阻塞信号集`, 另一个称之为`未决信号集`。 
这两个信号集都是使用位图机制来实现的。但操作系统不允许我们直接对者两个信号进行位操作。
而需自定义另外一个集合，借助信号集操作函数来对 PCB 中的这两个信号集进行修改。

- 信号的`未决`是一种状态，指的是从信号的产生到信号被处理前的这一段时间。
- 信号的`阻塞`是一个开关动作，指的是阻止信号被处理，但不是阻止信号产生。

信号的阻塞就是让系统暂时保留信号留待以后发送。由于另外有办法让系统忽略信号，所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作。


### 阻塞信号集和未决信号集

1. 用户通过键盘 Ctrl-C，产生 2 号信号 SIGINT (信号被创建）

2. 信号产生但是没有被处理（`未决`）
- 在内核中，将所有的没有被处理的信号存储在一个集合中（未决信号集）
- SIGINT 信号状态被存储在第二个标志位上
    - 这个标志位为 0， 说明信号不是未决状态，1 ，说明该信号处于未决状态。

3. 这个未决状态的信号，需要被处理，处理之前需要和另一个信号集（阻塞信号集），进行一个比较。
- 阻塞信号集默认不阻塞任何信号。
- 如果想要阻塞某些信号需要用户调用系统的 API

4. 在处理的时候和阻塞信号集中的标志位进行查询，看是不是对该信号设置阻塞了
- 如果没有阻塞，这个信号就被处理
- 如果阻塞了，这个信号就继续处于未决状态，直到阻塞解除，这个信号就被处理

### 示例程序

```c
  /**

     int sigemptyset(sigset_t *set);
     - 功能： 清空信号集中的数据， 将信号集中的所有的标志位置0
     - 参数：
     - set 传出参数： 需要操作的信号集
     - signum: 需要设置阻塞的那个信号
     - 返回值：


     以下信号集相关的函数都是对自定义的信号集进行操作。
     NAME
     sigemptyset, sigfillset, sigaddset, sigdelset, sigismember - POSIX signal
     set operations

     SYNOPSIS
     #include <signal.h>

     int sigemptyset(sigset_t *set);

     int sigfillset(sigset_t *set);
     - 功能： 将信号集中的所有标志位为 1

     int sigaddset(sigset_t *set, int signum);
     - 功能： 设置信号集中的某一信号对应的标志位为 1，表示阻塞这个信号

     int sigdelset(sigset_t *set, int signum);
     - 功能： 设置信号集中的某一信号对应的标志位为 0，表示不阻塞这个信号

     int sigismember(const sigset_t *set, int signum);
     - 功能： 判断某个信号是否阻塞。

     RETURN VALUE
     sigemptyset(), sigfillset(), sigaddset(), and sigdelset() return 0 on
     success and -1 on error.

     sigismember() returns 1 if signum is a member of set, 0 if signum is not
     a member, and -1 on error.

     On error, these functions set errno to indicate the cause of the error.
  */

  #include <signal.h>
  #include <stdio.h>

  int main() {
    sigset_t set;  // 创建一个信号集
    sigemptyset(&set);  // 清空信号集的内容

    // 判断 SIGINT 是否在信号集 set 里
    int ret = sigismember(&set, SIGINT);
    if (ret == 0) {
      printf("SIGINT 不阻塞。\n");
    } else if (ret == 1) {
      printf("SIGINT 阻塞。\n");
    }
    
    sigaddset(&set, SIGINT);  // 添加几个信号到信号集中
    sigaddset(&set, SIGQUIT);

    // 判断 SIGINT 是否在信号集 set 里
    ret = sigismember(&set, SIGINT);
    if (ret == 0) {
      printf("SIGINT 不阻塞。\n");
    } else if (ret == 1) {
      printf("SIGINT 阻塞。\n");
    }

    // 判断 SIGINT 是否在信号集 set 里
    ret = sigismember(&set, SIGQUIT);
    if (ret == 0) {
      printf("SIGQUIT 不阻塞。\n");
    } else if (ret == 1) {
      printf("SIGQUIT 阻塞。\n");
    }

    // 从信号集中删除一个信号
    sigdelset(&set, SIGQUIT);

    // 判断 SIGINT 是否在信号集 set 里
    ret = sigismember(&set, SIGQUIT);
    if (ret == 0) {
      printf("SIGQUIT 不阻塞。\n");
    } else if (ret == 1) {
      printf("SIGQUIT 阻塞。\n");
    }

    return 0;
  }
```

## sigprocmask 函数使用

+ 实例程序

```c
  /**
     NAME
     sigprocmask, rt_sigprocmask - examine and change blocked signals

     SYNOPSIS
     #include <signal.h>

     int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
     - 功能：
     将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
     - 参数：
     - how 如果对内核阻塞信号集进行处理
     - SIG_BLOCK:
     将用户设置的阻塞信号集添加到内核中，内核中原来的数据不变，
     - 假设内核中默认的阻塞信号集是 mask | set。
     - SIG_UNBLOCK:
     根据用户设置的数据，对内核中的数据进行解除阻塞。mask &= ~ set;
     - SIG_SETMASK: 覆盖内核中原来的值

     - set: 已经初始化号的用户自定义的信号集
     - oldset: 保存设置之前的内核中的阻塞信号集，一般设置为 NULL。

     RETURN VALUE
     sigprocmask() returns 0 on success and -1 on error.  In the event of an
     error, errno is set to indicate the cause. ERRORS EFAULT The set or oldset
     argument points outside the process's allocated address space.

     EINVAL Either the value specified in how was invalid or the kernel does
     not support the size passed in sigsetsize.


     NAME
     sigpending, rt_sigpending - examine pending signals

     SYNOPSIS
     #include <signal.h>

     int sigpending(sigset_t *set);
     - 功能： 获取内核中的未决信号集
     - 参数： set 传出参数， 保存的是内核中的未决信号集中的信息。
  */
  // 编写一个程序，把常规信号 1-31 的未决状态打印到屏幕
  // 设置某些信号是阻塞的，通过键盘产生这些信号
  #include <bits/types/sigset_t.h>
  #include <signal.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>

  int main() {
    // 设置 2 号， 3 号信号阻塞
    sigset_t set;
    sigemptyset(&set);

    // 将2号， 3号添加到信号集中
    sigaddset(&set, SIGINT);
    sigaddset(&set, SIGQUIT);

    // 修改内核中的阻塞信号集
    sigprocmask(SIG_BLOCK, &set, NULL);

    int num = 0;
    while (1) {
      num++;
      sigset_t pendingset;
      sigemptyset(&pendingset);
      sigpending(&pendingset);

      // 遍历 32 位
      for (int i = 1; i <= 32; i++) {
        if (sigismember(&pendingset, i) == 1) {
          printf("1");
        } else if (sigismember(&pendingset, i) == 0) {
          printf("0");
        } else {
          perror("sigismember");
          exit(0);
        }
      }

      printf("\n");
      sleep(1);
      if (num == 10) {
        // 解除阻塞
        sigprocmask(SIG_UNBLOCK, &set, NULL);
      }
    }

    return 0;
  }
```

## sigaction 信号捕捉函数

```c
  /**
     NAME
     sigaction, rt_sigaction - examine and change a signal action

     SYNOPSIS
     #include <signal.h>

     int sigaction(int signum, const struct sigaction *act,
     struct sigaction *oldact);
     - 功能： 检查或者改变信号的处理。信号捕捉
     - 参数：
     - signum: 需要捕捉的信号的编号或者宏值，
     - act : 捕捉到信号之后的处理动作
     - oldact: 上一次对信号捕捉相关的设置，一般不使用， 传递 NULL

     RETURN VALUE
     sigaction() returns 0 on success; on error, -1 is returned, and errno is
     set to indicate the error.

     ERRORS
     EFAULT act or oldact points to memory which is not a valid part of the
     process address space.

     EINVAL An  invalid  signal  was  specified.   This will also be generated
     if an attempt is made to change the action for SIGKILL or SIGSTOP, which cannot
     be caught or ignored.


     struct sigaction {
     // 函数指针，指向的函数就是信号捕捉到之后的处理函数
     void     (*sa_handler)(int);

     // 不常用
     void     (*sa_sigaction)(int, siginfo_t *, void *);

     // 临时阻塞信号集，信号捕捉函数执行过程中，临时阻塞某些信号
     sigset_t   sa_mask;

     // 使用哪一个信号处理函数对捕捉到的信号进行处理，
     这个值可以是0，表示使用 sa_handler, 也可以是 SA_SIGINFO , 使用 sa_sigaction。
     int        sa_flags;

     // 被废弃掉了
     void     (*sa_restorer)(void);
     };
  */
  #include <signal.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <sys/time.h>

  void my_alarm(int num) {
    printf("捕捉到信号的编号是： %d\n", num);
    printf("xxxx\n");
  }

  // 过 3 秒钟以后， 每隔 2 秒钟定时一次。
  int main() {
    struct sigaction act;
    act.sa_flags = 0;
    act.sa_handler = my_alarm;
    sigemptyset(&act.sa_mask); // 清空临时阻塞信号集
    
    sigaction(SIGALRM, &act, NULL);  // 注册信号

    struct itimerval new_value;

    // set
    // 设置间隔时间
    new_value.it_interval.tv_sec = 2;
    new_value.it_interval.tv_usec = 0;

    // 设置延迟的时间
    new_value.it_value.tv_usec = 0;
    new_value.it_value.tv_sec = 3;

    int ret = setitimer(ITIMER_REAL, &new_value, NULL);
    printf("定时器开始了\n");

    if (ret == -1) {
      perror("setitimer");
      exit(0);
    }

    // getchar();
    while (1) {   }
    return 0;
  }
```

尽量使用 `sigaction`. 而不是 signal.

1. 回调函数中用的是临时阻塞信号集， 当回调完，又使用内核中的阻塞信号集。
2. 在处理 `SIGALRM` 信号的时候会默认屏蔽掉 SIGALRM 信号（系统行为），等处理完才会再触发。
3. 不支持排队。只能最多记录一个信号是否是未决，不能记录个数。

+ 实时信号支持排队。

## SIGCHLD 信号

+ `SIGCHLD` 信号产生的条件
    - 子进程终止时
    - 子进程 `SIGSTOP` 信号停止时
    - 子进程 处在停止态，接收到 `SIGCONT` 后唤醒时

+ 以上三种条件都会给父进程发送 SIGCHLD 信号，父进程默认会忽略该信号

+ 回收子进程时可能会回收不全。

```c
  //  使用 SIGCHLD 信号解决僵尸进程的问题

  #include <bits/types/sigset_t.h>
  #include <signal.h>
  #include <stdio.h>
  #include <string.h>
  #include <sys/stat.h>
  #include <sys/types.h>
  #include <sys/wait.h>
  #include <unistd.h>

  void myfun(int num) {
    printf("捕捉到的信号： %d,\n", num);
    // 回收子进程的 PCB 资源
    // 1
    /**
       回收不全，会留下僵尸进程
    ,*/
    // wait(NULL);

    // 2
    /**
       会阻塞
    ,*/
    /* while (1) { */
    /*   wait(NULL); */
    /* } */

    // 3
    /**
       有一种可能，子进程已经结束，SIGCHLD 信号还未注册。
       需要提前设置好阻塞信号集。
    ,*/
    while (1) {
      int ret = waitpid(-1, NULL, WNOHANG);
      if (ret > 0) {
        printf("child die, pid = %d\n", ret);
      } else if (ret == 0) {  // 说明还有子进程活着
        break;
      } else if (ret == -1) {
        break;
      }
    }
  }

  int main() {
    // 提前设置好阻塞信号集，阻塞 SIGCHLD,
    // 因为有可能子进程很快结束，父进程还没有注册完信号捕捉

    sigset_t set;
    sigemptyset(&set);
    sigaddset(&set, SIGCHLD);
    sigprocmask(SIG_BLOCK, &set, NULL);
    
    pid_t pid;
    for (int i = 0; i < 20; i++) {
      pid = fork();
      if (pid == 0) break;
    }

    if (pid > 0) { // parent child
      // 捕捉子进程死亡时发送的 SIGCHLD 信号
      struct sigaction act;
      act.sa_flags = 0;
      act.sa_handler = myfun;
      sigemptyset(&act.sa_mask);
      sigaction(SIGCHLD, &act, NULL);

      // 注册完信号捕捉以后解除阻塞
      sigprocmask(SIG_UNBLOCK, &set, NULL);

      while (1) {
        printf("Parent process: pid = %d\n", getpid());
        sleep(2);
      }
    } else if (pid == 0) {
      printf("Child process: pid = %d\n", getpid());
    }

    return 0;
  }
```

## 共享内存

`共享内存`允许两个或多个进程共享物理内存的同一块区域（通常被称为`段`）。由于一个共享内存段会称之为一个进程用户空间的一部分，因此
这种`IPC`机制无需内核介入。所有需要做的就是让一个进程将数据复制进共享内存中， 并且这部分数据会对其他所有共享同一个段的进程可用。

与管道等要求发送进程将数据从用户空间的缓冲区复制进内核内存和接收进程将数据从内存复制进用户空间的缓冲区的做法相比，这个 IPC 技术的速度更快。

### 共享内存的使用步骤

+ 调用 shmget() 创建一个新的共享内存段或取得一个既有共享内存段的标识符（即由其他进程创建的共享内存段）。
这个调用将返回后续调用中需要用到的共享内存标识符。

+ 使用 shmat() 来附上共享内存段，即使该段称为调用进程的虚拟内存的一部分

+ 此刻程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存，程序需要使用由 shmat()
调用返回的 addr 值，它是一个指向进程的虚拟地址空间中该共享内存段的起点的指针。

+ 调用 shmdt() 来分离共享内存段。在这个调用之后，进程就无法再引用这块共享内存了。
这一步是可选的，并且进程终止时会自动完成这一步。

+ 调用 shmctl() 来删除共享内存段。只有当当前所有附加内存段的进程都与之分离之后内存段才会销毁。只有一个进程需要执行这一步。


### 共享内存相关的函数

+ write_shm.c

```c
  /**
     NAME
     shmget - allocates a System V shared memory segment

     SYNOPSIS
     #include <sys/ipc.h>
     #include <sys/shm.h>

     int shmget(key_t key, size_t size, int shmflg);
     - 功能： 创建一个新的共享内存段或取得一个既有共享内存段的标识符
     新创建的内存段中的数据都会被初始化为0，
     - 参数：
     - key, key_t
     类型是一个整型，通过这个可以找到或者创建一个共享内存，一般使用16进制表示，非0值。
     - size, 共享内存大小，以分页为单位
     - shmflg: 属性：如访问权限，附加属性，创建或者判断是否存在，
     - 创建： IPC_CREAT
     - 判断共享内存是否存在 IPC_EXCL, 需要和 IPC_CREAT 一起使用。IPC_CREAT |
     IPC_EXCL | 0664
     - 返回值：xsRETURN VALUE
     On success, a valid shared memory identifier is returned.  On error, -1 is
     returned, and errno is set to indicate the error.

     void *shmat(int shmid, const void *shmaddr, int shmflg);
     功能 ： 和当前的进程进行关联
     参数： shmid: 共享内存的标识（id）, 由 shmget 返回值获取。
     shmaddr: 申请的共享内存的起始地址，执行 NULL，由内核指定。
     shmflg: 对共享内存的操作，读权限 SHM_RDONLY 必须要有读权限； 读写 指定0.
     返回值： 成功： 返回共享内存的起始地址，失败，返回 (void*) -1; 会设置错误号

     int shmdt(const void *shmaddr);
     - 功能 解除当前进程和共享内存的关联
     参数： shmaddr 共享内存的首地址
     返回值： 成功0， 失败 -1；


     int shmctl(int shmid, int cmd, struct shmid_ds *buf);
     - 功能： 删除共享内存，
     共享内存要删除才会消失，创建共享内存的进程被销毁对共享内存没有影响。
     - 参数：： shmid : 共享内存的 ID
     -- cmd: 要做的操作，：IPC_STAT: 获取共享内存的当前的状态，
     --- IPC_SET: 设置共享内存的状态，
     --- IPC_RMID： 标记共享内存被销毁。
     -- buf : 需要设置或获取的共享内存的属性信息
     - IPC_STAT: buf 存储数据
     - IPC_SET： buf 中需要初始化数据，设置到内核中
     - IPC_RMID: 没有用，传递NULL。

     key_t ftok(const char *pathname, int proj_id);
  */
  #include <stdio.h>
  #include <string.h>
  #include <sys/ipc.h>
  #include <sys/shm.h>

  int main() {
    // 创建一个共享内存
    int shmid = shmget(100, 4096, IPC_CREAT | 0664);
    printf("shmid = %d\n", shmid);
    
    void *ptr = shmat(shmid, NULL, 0);  // 和当前进程进行关联
    char *str = "hello world";
    
    memcpy(ptr, str, strlen(str) + 1);  // 3. 写数据
    printf("按任意键继续...\n");
    getchar();
    
    shmdt(ptr);  // 4， 解除关联
    shmctl(shmid, IPC_RMID, NULL);  //  5. 删除共享内存
    return 0;
  }
```

+ read_shm.c

```c
  #include <stdio.h>
  #include <string.h>
  #include <sys/ipc.h>
  #include <sys/shm.h>

  int main() {
    // 1. 获取一个共享内存
    int shmid = shmget(100, 0, IPC_CREAT);
    printf("shmid = %d\n", shmid);

    // 2. 和当前进程进行关联
    void *ptr = shmat(shmid, NULL, 0);

    printf("%s\n", (char *)ptr);
    shmdt(ptr);
    shmctl(shmid, IPC_RMID, NULL);
    return 0;
  }
```


1. 操作系统如何知道一块共享内存被多少个进程关联？
- 共享内存中维护了一个结构体。struct shmid_ds 这个结构体中有一个成员， shm_nattach,
- shm_nattach 记录了关联的进程的个数

2. 可不可以对共享内存进行多次删除 shmctl
- 可以的，
- 因为 shmctl 标记删除共享内存，不是直接删除
- 什么时候真正删除呢？
    - 当和共享内存关联的进程数为0的时候，就真正删除了。
- 当共享内存的 key 为0 的时候表示共享内存被标记删除了。
    - 如果一个进程和共享内存取消关联，那么这个进程就不能继续操作这个共享内存。也不能进行关联。

+ 共享内存操作命令 ipcs, ipcrm

```shell
  ipcs -a # 打印当前系统中所有的进程间通信方式的信息
  ipcs -m # 打印出使用共享内存进行进程间通信的信息

  ipcrm -M shmkey # 移除用 shmkey 创建的共享内存段
  ipcrm -m shmid # 移除用 shmid 标识的共享内存段
```

### 共享内存和内存映射的区别

1. 共享内存可以直接创建， 内存映射需要磁盘文件（匿名映射除外）
2. 共享内存效率更高，
3. 所有进程操作的是同一块共享内存，内存映射是每个进程在自己的虚拟地址空间中有一个独立的内存。
4. 数据安全问题，
- 进程突然退出，共享内存还在；内存映射区会消失
- 运行进程的电脑死机了，宕机了，数据在共享内存中，没有了，内存映射区，由于磁盘文件中的数据还在，所以内存映射区的数据还在

5. 生命周期：
- 内存映射区： 进程退出，内存映射区销毁。
- 共享内存： 进程退出，共享内存还在，标记删除（所有的关联进程数为0），或者关机
    - 如果一个进程退出，会自动和共享内存取消关联

## 守护进程

在 UNIX 系统中，用户通过终端登录系统后得到一个 shell 进程，这个终端称为 shell 进程的`控制终端` （Controling Terminal）, 进程中，
控制终端是保存在 PCB 中的信息， 而 fork() 会复制 PCB 中的信息， 因此 shell 进程启动的其他进程的控制终端也是这个终端。

```shell
tty
echo $$ # 查看当前终端的进程ID
```

默认情况下（没有重定向），每个进程的`标准输入`，`标准输出`和`标准错误输出`都能指向控制终端，进程从标准输入读也就是读用户的键盘输入，
进程往标准输出或标准错误输出写也就是输出到显示器上。

在控制终端输入一些特殊的控制键可以给当前进程发送信号，例如 `Ctrl + C` 会产生 `SIGINT` 信号， `Ctrl + \` 会产生 `SIGQUIT` 信号。

### 进程组

`进程组`和`会话`在进程之间形成了一种两级层次关系： 进程组是一组相关进程的集合，会话是一组相关进程组的集合。
进程组和会话是为支持 shell 作业控制而定义的抽象概念，用户通过 shell 能够交互式地在前台或后台运行命令。

`进程组`由一个或多个共享同一个进程组标识符 （PGID） 的进程组成。一个进程组拥有一个进程组首进程，该进程是创建该组的进程，
其进程 ID 为该进程组的ID，新进程会继承其父进程所属的进程组ID。

`进程组`拥有一个生命周期，其开始时间为首进程创建组的时刻，结束时间为最后一个成员退出组的时刻。
一个进程可能会因为终止而退出进程组，也可能会因为加入了另外一个进程组而退出进程组。进程组首进程无需是最后一个离开进程组的成员。

### 会话

`会话`是一组进程组的集合。会话首进程是创建该新会话的进程，其进程 ID 会称为 会话 ID. 新进程会继承父进程的会话 ID。

一个会话中的所有进程共享单个控制终端。控制终端会在会话首进程首次打开一个终端设备时被建立。一个终端最多可能会称为一个会话的控制终端。

在任意时刻，会话中的其中一个进程组会称为终端的前台进程组，其他进程组会称为后台进程组。只有前台进程组中的进程才能控制终端中读取输入。
当用户在控制终端中输入终端字符生成信号后，该信号会被发送到前台进程组中的所有成员。

当控制终端的连接建立起来之后，会话首进程会称为该终端的控制进程。

+ PID， PPID， PGID， SID

+ 进程组、会话操作函数

```c
  /**
     NAME
     setpgid, getpgid, setpgrp, getpgrp - set/get process group

     SYNOPSIS
     #include <sys/types.h>
     #include <unistd.h>

     int setpgid(pid_t pid, pid_t pgid);
     pid_t getpgid(pid_t pid);

     pid_t getpgrp(void);                 
     pid_t getpgrp(pid_t pid);            

     int setpgrp(void);       
     int setpgrp(pid_t pid, pid_t pgid);

     NAME
     getsid - get session ID

     SYNOPSIS
     #include <sys/types.h>
     #include <unistd.h>

     pid_t getsid(pid_t pid);
     pid_t getsid(pid_t pid);
  */
```

### 守护进程

`守护进程` （Daemon Process）, 也就是通常说的 Daemon 进程（精灵进程），是 Linux 中的后台服务进程。
它是一个生存期较长的进程，通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。
一般采用以 d 结尾的名字。

守护进程具备以下特征：
+ 生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭。
+ 它在后台运行并且不拥有控制终端。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如 SIGINT，SIGQUIT）  。

Linux 的大多数服务器就是用守护进程实现的。比如 Internet 服务器， inetd, web 服务器 httpd 等。

### 守护进程的创建步骤

+ [-] 执行一个 fork() , 之后父进程退出，子进程继续执行。(保证不是进程组的首进程)
+ [-] 子进程调用 setsid() 开启一个新会话。（无控制终端）避免组和会话ID冲突
+ 清楚进程的 umask 以确保当前守护进程创建文件和目录时拥有所需的权限。
+ 修改进程的当前工作目录，通常会改为根目录(/);
+ 关闭守护进程从其父进程继承而来的所有打开着的文件描述符。
+ 在关闭了文件描述符0、1、2之后，守护进程通常会打开 /dev/null 并使用 dup2() 使所有这些描述符指向这个设备。
+ [-] 核心业务逻辑。

### 示例代码

```c
  //  写一个守护进程，每个两秒 获取一下系统时间，将这个时间写入到磁盘文件中。
  #include <fcntl.h>
  #include <signal.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <sys/stat.h>
  #include <sys/time.h>
  #include <sys/types.h>
  #include <time.h>
  #include <unistd.h>

  void work(int num) {
    // printf("catch signal num = %d,\n",num);

    // 捕捉到信号之后，获取系统时间，写入磁盘文件。
    time_t tm = time(NULL);
    struct tm *loc = localtime(&tm);

    /* char buf[1024]; */
    /* sprintf(buf, "%d-%d-%d %d:%d:%d\n", loc->tm_year, loc->tm_mon,
     ,* loc->tm_mday, */
    /*         loc->tm_hour, loc->tm_min, loc->tm_sec); */

    /* printf("%s", buf); */

    char *str = asctime(loc);
    int fd = open("/home/larry/GitProjects/chips-get-cpp/Tiny-Web-Server-CPP/"
		  "Preparation/Code_02/time.txt",
		  O_RDWR | O_CREAT | O_APPEND, 0644);
    write(fd, str, strlen(str));
    close(fd);
  }

  int main() {
    // 1. 创建子进程
    pid_t pid = fork();

    if (pid > 0) {
      exit(0);
    }

    // 2, 将子进程创建一个会话
    setsid();

    // 3. 设置掩码
    umask(022);

    // 4. 更改工作目录
    chdir("/");

    // 5. 关闭和重定向文件描述符
    int fd = open("/dev/null", O_RDWR);
    dup2(fd, STDIN_FILENO);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);

    // 6. 业务逻辑

    // 捕捉定时信号
    struct sigaction act;
    act.sa_flags = 0;
    act.sa_handler = work;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);

    // 创建定时器
    struct itimerval val;
    val.it_value.tv_sec = 2;
    val.it_value.tv_usec = 0;

    val.it_interval.tv_usec = 0;
    val.it_interval.tv_sec = 2;
    setitimer(ITIMER_REAL, &val, NULL);
    
    while (1) {  // 不让进程结束
      sleep(10);
    }

    return 0;
  }
```