---
layout: post
title: "[学习笔记] Linux 系统编程入门"
date: "2022-12-08 08: 21: 22"
tag: cpp linux system programming
---
- [Linux 系统编程入门](#org514c87e)
  - [静态库与动态库](#orgcf2ef70)
    - [静态库命名规则](#org8407937)
    - [静态库的制作](#org181b542)
    - [静态库使用](#org6180f59)
    - [动态库制作](#org40f12ef)
    - [动态库使用](#org5da20c1)
    - [加载动态库](#org811d511)
    - [静态库的优缺点](#orgaff67b9)
    - [动态库的优缺点](#orgea46012)
  - [Makefile](#orge2e86da)
    - [文件命名](#org480a533)
    - [工作原理](#org914578b)
    - [变量](#org53da277)
    - [模式匹配](#orgc7cc95a)
    - [函数](#orgce51452)
  - [GDB 调试](#org7b111c9)
    - [GDB 是什么？](#org464318e)
    - [开始调试](#orga483410)
    - [gdb 指令](#org81eeef9)
  - [Linux系统IO函数](#org3a4ed91)
    - [标准C库IO函数](#orga55ff6d)
    - [虚拟地址空间(真实不存在)](#orgd14d836)
    - [文件描述符](#org15fedb4)
    - [open 打开文件](#org879e4a6)
    - [create 文件](#org9f7c97e)
    - [read 和 write](#orga684c7e)
    - [lseek 函数](#orgb57140a)
    - [stat 和 lstat 函数](#orge66b557)
    - [模拟实现 ls -l](#org1742a02)
    - [文件属性操作函数](#org7dbe7c8)
    - [目录操作函数](#org341aa7a)
    - [目录遍历函数](#orgb0494e9)
    - [dup 和 dup2 函数](#org954074a)
    - [fcntl 函数](#org606f868)


<a id="org514c87e"></a>

# Linux 系统编程入门

<a id="orgcf2ef70"></a>

## 静态库与动态库

<a id="org8407937"></a>

### 静态库命名规则

-   linux: libxxx.a 保持 lib 前缀 和 .a 后缀
-   Windows: libxxx.lib

<a id="org181b542"></a>

### 静态库的制作

gcc 获得 .o 文件 将 .o 文件用 ar 工具打包

```shell
ar rcs libxxx.a xxx.o xxx.o
# -r 将文件插入备份文件中
# -c 建立备份文件  
# -s 索引
```

<a id="org6180f59"></a>

### 静态库使用

```shell
g++ main.cpp -I ./include  -l calc -L./lib -o test && ./test
# -I 包含路径
# -l 库名字
# -L 库路径
```

<a id="org40f12ef"></a>

### 动态库制作

-   命名规则
    -   Linux： libxxx.so 保持 lib 前缀和 .so 后缀, 在linux 下是一个可执行文件
    -   Windows: libxxx.dll

```shell
gcc -c -fpic *.c  # 得到和位置无关的代码
gcc -shared *.o -o libxxx.so  # 生成动态库
```

```shell
g++ main.cpp -I ./include -L lib/ -l calc -o main && ./main
# ./main: error while loading shared libraries: libcalc.so: cannot open shared object file: No such file or directory
```


<a id="org5da20c1"></a>

### 动态库使用

程序启动之后，动态库会被动态加载到内存，通过 `ldd` (list dynamic dependencies) 命令检查动态库依赖关系。

```shell
ldd main
# linux-vdso.so.1 (0x00007ffd771f6000)
# libcalc.so => not found		
# libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f47d0be9000)
# libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f47d09f7000)
# libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f47d08a8000)
# /lib64/ld-linux-x86-64.so.2 (0x00007f47d0e28000)
# libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f47d088d000)
```

- 如何定位共享库文件呢？

    当系统加载可执行代码时，能够知道其所依赖的`库的名字`，但还是需要知道`绝对路径`。
    
    -   搜索路径
        -   elf 文件的 DT_RPATH 段
        -   LD_LIBRARY_PATH
        -   /etc/ld.so.cache 文件列表
        -   /lib, /usr/lib


<a id="org811d511"></a>

### 加载动态库

```shell
# 方法一， 临时用
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:`pwd`/bin
./main
ldd main

# 方法二, 用户级别
# 写到 .shrc 中

# 方法三， 系统级别
# 写到 /etc/profile
```

-   添加到文件列表

```shell
sudo nano /etc/ld.so.conf
sudo ldconfig
```


<a id="orgaff67b9"></a>

### 静态库的优缺点

-   优点：
    -   静态库被打包到应用程序中加载速度快
    -   发布程序无需提供静态库，移植方便

-   缺点
    -   消耗系统资源，浪费内存
    -   更新、部署、发布麻烦


<a id="orgea46012"></a>

### 动态库的优缺点

-   优点：
    -   可以实现进程间资源共享（共享库）
    -   更新、部署、发布简单
    -   可以控制何时加载动态库

-   缺点
    -   加载速度相对静态库慢
    -   发布程序时需要提供依赖的动态库


<a id="orge2e86da"></a>

## Makefile

-   `Makefile` 文件定义了一系列规则来编译指定文件。
-   `make` 一个命令行工具： 解释 Makefile 文件中指令的命令工具。


<a id="org480a533"></a>

### 文件命名

-   makefile 或者 Makefile
-   一个 Makefile 文件中可以有一个或多个规则
    -   目标 &#x2026; : 依赖 &#x2026;
        -   命令 （shell 命令）
        -   &#x2026;

-   `目标`： 最终要生成的文件（伪目标除外）
-   `依赖`： 生成目标所需要的文件或是目标
-   `命令`： 通过执行命令对依赖操作生成目标（命令前必须 Tab 缩进）

-   Makefile 中的其他规则一般都是为第一条规则服务的


<a id="org914578b"></a>

### 工作原理

-   命令在执行之前， 需要先检查规则中的依赖是否存在
    -   如果存在， 执行命令
    -   如果不存在，向下检查其他规则，检查有没有一个规则是用来生成这个依赖的，如果找到了，则执行该规则中的命令。

-   检测更新，在执行规则中的命令是，会比较目标文件和依赖文件的时间
    -   如果依赖的时间比目标的时间晚，需要重新生成目标
    -   如果依赖的时间比目标时间早， 目标不需要更新，对应规则中的命令不需要被执行


<a id="org53da277"></a>

### 变量

-   自定义变量： 变量名=变量值 var=hello
-   预定义变量：
    -   `AR`: 归档维护程序的名称，默认值 ar
    -   `CC`: C 编译器的名称， 默认值 cc
    -   `CXX`: C++ 编译器的名称, 默认值 g++
    -   `$@`: 目标的完整名称

-   获取变量的值: $(变量名)


<a id="orgc7cc95a"></a>

### 模式匹配

-   %.o:%.c
    -   %: 通配符， 匹配一个字符串
    -   两个%匹配的是同一个字符串


<a id="orgce51452"></a>

### 函数

-   $(wildcard PATTERN)
    -   功能： 获取指定目录下指定类型的文件列表
    -   参数： PATTERN 指的是某个或多个目录下对应的某种类型的文件，如果有多个目录，一般使用空格间隔
    -   返回： 得到的若干个文件列表，文件名之间使用空格间隔
    -   示例： $(wildcard \*\*.cpp ./sub/\*\*.cpp)
        -   返回值： a.cpp b.cpp c.cpp

-   $(patsubst [patten],[replacement],[text])
    -   功能： 查找 text, 符合 patten, 用 replacement 替换
    -   pattern 可以包含通配符 %, 表示任意长度的字串。如果 replacement 中也包含 %， 那么replacement 中的 % 与 patten 代表字串相同，可以用 \\ 来转义。
    -   返回： 函数返回被替换过后的字符串
    -   示例： $(patsubst %.c, %.o, x.c bar.c)
        -   返回值格式: x.o bar.o


<a id="org7b111c9"></a>

## GDB 调试


<a id="org464318e"></a>

### GDB 是什么？

-   `GDB` 是由 GUN 软件系统社区提供的调试工具， 同 GCC 配套组成了一套完整的开发环境， GDB 是 Linux 和许多类 Unix 系统中的标准开发环境。


<a id="orga483410"></a>

### 开始调试

-   关掉编译器优化选项 （-O）
-   打开调试选项 (-g) 在可执行文件中加入源代码的信息，比如可执行文件中的第几条机器指令对应源代码中的第几行，但不是嵌入源文件，调试时需要保证 gdb 能找到源文件。
-   (-Wall) 在不影响程序行为的情况下，打开所有 warning

```shell
g++ main.cpp -g -Wall -o test
gdb test
```


<a id="org81eeef9"></a>

### gdb 指令

-   查看当前文件代码
    -   list : 列出10行代码
    -   list\l 行号 : 从指定行显示, 指定行在显示的中心位置
    -   list\l 函数名 : 从指定函数显示

-   查看非当前文件代码
    -   list filename:line_number
    -   list filename:function_name

-   设置显示的行数
    -   show list/listsize
    -   set list/listsize

-   回车 : 上一次命令

-   设置断点
    -   b/break 行号/函数名/文件名：行号/文件名：函数

-   查看断点
    -   i/info b/break

-   删除断点
    -   d/del/delete 断点编号

-   设置断点无效
    -   dis/disable 断点编号

-   设置断点生效
    -   ena/enable 断点编号

-   设置断点条件
    -   b/break 10 if i == 5

-   运行gdb 程序
    -   start 程序停在第一行
    -   run 遇到断点才停

-   继续运行, 到下一个断点才停
    -   c/continue

-   向下执行一行代码,不会进入函数体
    -   n/next

-   变量操作
    -   p/print 变量名: 打印变量值
    -   ptype 变量名： 打印变量类型

-   向下单步调试，遇到函数进入函数体
    -   s/step
    -   finish 跳出函数体

-   自动变量操作
    -   display num， 自动打印指定变量的值
    -   i/info display
    -   undisplay 编号

-   其他操作
    -   set var 变量名=变量值
    -   until， 跳出循环


<a id="org3a4ed91"></a>

## Linux系统IO函数


<a id="orga55ff6d"></a>

### 标准C库IO函数

-   fopen, fclose, fread, fwrite, fgets, fputs, fscanf, fprintf, fseek, fgetc, fputc, ftell, feof, fflush&#x2026;

-   结构体： 文件描述符(整型值), 文件读写指针位置，IO缓冲区（内存地址）

用户程序 <&#x2013;> C标准I/O库 <&#x2013;> 内核(Read/Write) <&#x2013;> 磁盘


<a id="orgd14d836"></a>

### 虚拟地址空间(真实不存在)

Linux 下的可执行文件格式： ELF

| 用户区  | 内核区   |
|------|-------|
| 0-3G | 3G-4G |

从左到右：

-   `用户区`
    -   受保护的地址
    -   .text （代码段，二进制机器指令）
    -   .data (已初始化全局变量)
    -   .bss (未初始化全局变量)
    -   堆空间 (大)
    -   共享库
    -   栈空间 (小)
    -   命令行参数
    -   环境变量

-   `内核区` 内核空间是受保护的，用户不能对该空间进行读写操作，否则会出现段错误。
    -   内存管理
    -   进程管理
    -   设备驱动管理
    -   VFS虚拟文件系统


<a id="org15fedb4"></a>

### 文件描述符

-   `PCB` `进程控制块`

-   文件描述符
    -   0 -> stdin_fileno， 标准输入，默认打开
    -   1 -> stdout_fileno，标准输出，默认打开
    -   2 -> stderr_fileno, 标准错误， 默认打开
    -   3-1023 每打开一个新文件，占用一个文件描述符，而且是空闲的最小的一个文件描述符。


<a id="org879e4a6"></a>

### open 打开文件

```c
#include <fcntl.h>
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main(int argc, char **argv) {

  // 打开一个已经存在的文件
  // int open(const char* pathname, int flags);
  // int open(const char* pathname, int flags， mode_t mode );
  // 参数： pathname: 要创建的文件路径
  //       flags: 对文件的操作权限和其他设置
  //       mode: 八进制数，表示用户对新创建的文件的操作权限， 最终权限是 (mode & ~umask)
  // 返回一个新的文件描述符， 如果失败，返回 -1。
  // errno: 属于Linux 系统函数库, 库里面的一个全局变量，记录的是最近的错误号。

  /**
     #include<stdio.h>
     void perror(const char *s);
     作用： 打印 errno 对应的错误描述
     s 参数： 用户描述，比如 hello， 最终输出的内容的是 hello: xxx
     (实际的错误输出)
  */

  int fd = open("./a.txt", O_RDONLY);
  if (-1 == fd) {
    perror("open");
  }

  close(fd);
  return 0;
}
```


<a id="org9f7c97e"></a>

### create 文件

```c
#include <stdio.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {

  // 创建一个新的文件
  int fd = open("create.txt", O_RDWR |O_CREAT, 0775);
  if(-1 == fd) {
    perror("open create err");
  }

  close(fd);
  return 0;
}
```


<a id="orga684c7e"></a>

### read 和 write

```c
#include <fcntl.h>
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  /**
     #include <unistd.h>
     ssize_t read(int fd, void *buf, size_t count);
     参数: fd: 文件描述符， open得到，通过这个文件描述符操作某个文件
     buf: 需要读取数据存放的地方，数组的地址
     count: 指定数组的大小
     返回值: 成功， 大于0， 返回实际读取到的字节数， 0 文件读取完，
     失败-1 文件读取失败

     #include <unistd.h>	     
     ssize_t write(int fd, const void *buf, size_t count);
  */
  
  int fd = open("read_write.c", O_RDONLY);  // 1. open 打开文件
  if(-1 == fd){
    perror("open err");
    return -1;
  }

  // 2. 创建一个新的文件（copy 文件）
  int dest_fd = open("cpy.txt", O_WRONLY| O_CREAT, 0777);
  if(-1 == dest_fd){
    perror("create err");
    return -1;
  }
  
  char buffer[1024] = {0};  // 3. 频繁的读写操作
  int len = 0;
  while((len =   read(fd, buffer, sizeof(buffer))) > 0){
    write(dest_fd, buffer, len);
  }
  
  close(dest_fd);  // 4. 关闭文件
  close(fd);
  return 0;
}
```


<a id="orgb57140a"></a>

### lseek 函数

-   文件扩展功能

```c
/**
 // 标准C库函数
 #include <stdio.h>
 int fseek(FILE *stream, long offset, int whence);


 // Linux 系统函数库
 #include <sys/types.h>
 #include <unistd.h>

 off_t lseek(int fd, off_t offset, int whence);
 参数:  fd: 通过 open 得到，文件描述符
 offset: 偏移量
 whence:  SEEK_SET 设置文件指针的偏移量
 SEEK_CUR 设置偏移量，当前位置 + 第二个参数的 offset 的值
 SEEK_END 设置偏移量，文件大小 + 第二个参数的 offset 的值
 返回值： 返回文件指针的位置

 作用： 1. 移动文件指针到文件头
 lseek(fd, 0, SEEK_SET);
 2. 获取当前文件指针的位置
 lseek(fd, 0，SEEK_SET);
 3. 获取文件长度
 lseek(fd, 0, SEEK_END);
 4. 扩展文件的长度， 当前文件 10b, 110b, 增加了100个字节
 lseek(fd, 100, SEEK_END)
*/

#include <fcntl.h>
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {

  int fd = open("cpy.txt", O_RDWR);
  if (-1 == fd) {
    perror("read err");
    return -1;
  }

  // 扩展文件的长度
  int ret = lseek(fd, 100, SEEK_END);
  if (-1 == ret) {
    perror("lseek");
    return -1;
  }

  write(fd , " ", 1);
  close(fd);
  return 0;
}
```

**注： 需要写一次数据才能改变大小**


<a id="orge66b557"></a>

### stat 和 lstat 函数

-   st_mode 变量： 一个16位的变量， 包含（文件类型， 特殊权限位，User, Group, Others）
-   (st_mode & S_IFMT) == S_IFREG

```c
/**
   #include <sys/stat.h>
   #include <sys/types.h>
   #include <unistd.h>

   int stat(const char *pathname, struct stat *statbuf);
   作用：获取一个文件相关的一些信息
   参数：
   - pathname： 操作文件的路径
   - statbuf: 结构体变量，传出参数，用于保存获取到的文件信息。

   返回值：
   - 成功： 返回0，
   - 失败： 返回-1， 设置 errno

   int lstat(const char *pathname, struct stat *statbuf);
   作用： 获取软连接文件的信息
*/

#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {

  struct stat statbuf;
  //  int ret = stat("cpy_s.txt", &statbuf);
  int ret = lstat("cpy_s.txt", &statbuf);

  if (-1 == ret) {
    perror("stat err");
    return -1;
  }

  printf("size: %1d\n", (int)statbuf.st_size);
  return 0;
}
```


<a id="org1742a02"></a>

### 模拟实现 ls -l

```c
#include <pwd.h>
#include <stdio.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <time.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
  if (argc < 2) {  // 判断输入的参数是否正确
    printf("%s filename\n", argv[0]);
    return -1;
  }
  
  struct stat buffstat;  // 通过 stat 获取文件的信息
  int ret = stat(argv[1], &buffstat);
  if (-1 == ret) {
    perror("stat err");
    return -1;
  }
  
  char permit[11] = {0};  // 获取文件类型和文件权限
  switch (buffstat.st_mode & S_IFMT) {
  case S_IFLNK:
    permit[0] = '1';
    break;
  case S_IFDIR:
    permit[0] = 'd';
    break;
  case S_IFREG:
    permit[0] = '-';
    break;
  case S_IFBLK:
    permit[0] = 'b';
    break;
  case S_IFSOCK:
    permit[0] = 's';
    break;
  case S_IFCHR:
    permit[0] = 'c';
    break;
  case S_IFIFO:
    permit[0] = 'p';
    break;
  default:
    permit[0] = '?';
    break;
  }

  // 判断文件的访问权限
  permit[1] = (buffstat.st_mode & S_IRUSR) ? 'r' : '-';
  permit[2] = (buffstat.st_mode & S_IWUSR) ? 'w' : '-';
  permit[3] = (buffstat.st_mode & S_IXUSR) ? 'x' : '-';

  permit[4] = (buffstat.st_mode & S_IRGRP) ? 'r' : '-';
  permit[5] = (buffstat.st_mode & S_IWGRP) ? 'w' : '-';
  permit[6] = (buffstat.st_mode & S_IXGRP) ? 'x' : '-';

  permit[7] = (buffstat.st_mode & S_IROTH) ? 'r' : '-';
  permit[8] = (buffstat.st_mode & S_IWOTH) ? 'w' : '-';
  permit[9] = (buffstat.st_mode & S_IXOTH) ? 'x' : '-';

  // 硬连接数
  int linkNum = buffstat.st_nlink;
  char *fileUser = getpwuid(buffstat.st_uid)->pw_name;
  char *fileGroup = getpwuid(buffstat.st_gid)->pw_name;

  // 文件大小
  long int fileSize = buffstat.st_size;

  // 获取修改时间
  char *changeTime = ctime(&buffstat.st_mtime);
  char mtime[512] = {0};
  strncpy(mtime, changeTime, strlen(changeTime) - 1);

  char buf[1024];
  sprintf(buf, "%s %d %s %s %ld %s %s", permit, linkNum, fileUser, fileGroup,
	  fileSize, mtime, argv[1]);

  printf("%s\n", buf);
  return 0;
}
```


<a id="org7dbe7c8"></a>

### 文件属性操作函数

-   access
-   判断文件存在

```c
/**
   #include <unistd.h>

   int access(const char *pathname, int mode);
   作用： 判断某个文件是否有某个权限， 或者判断文件是否存在
   参数： pathname: 判断文件路径
   mode: R_OK, W_OK, X_OK： 判断是否有相应权限
   返回值： 0 成功，-1 失败
*/

#include <stdio.h>
#include <unistd.h>

int main() {

  int ret = access("cpy.txt", R_OK);
  if (-1 == ret) {
    perror("access err");
    return -1;
  }

  printf("file exist!!! \n");
  return 0;
}
```

-   chmod

```c
/**
   #include <sys/stat.h>

   int chmod(const char *pathname, mode_t mode);
   作用： 修改文件的权限
   参数： pathname: 要修改文件的路径
   mode:  需要修改的权限值，八进制的数

   返回值：成功 0， 失败 -1
*/
#include <stdio.h>
#include <sys/stat.h>

int main() {
  int ret = chmod("cpy.txt", 0775);
  if (-1 == ret) {
    perror("chmod err");
    return -1;
  }
  return 0;
}
```

-   chown

```c
#include <unistd.h>
int chown(const char *pathname, uid_t owner, gid_t group);
```

-   truncate

```c
/**
   #include <sys/types.h>
   #include <unistd.h>

   int truncate(const char *path, off_t length);
   作用： 缩减或者扩展文件的尺寸到指定大小
   参数： path: 需要修改的文件的路径
   length: 需要最终文件变成的大小

   返回值： 成功 0， 失败 -1.
*/

#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  int ret = truncate("cpy.txt", 20);
  if (-1 == ret) {
    perror("truncate err");
    return -1;
  }
  return 0;
}
```


<a id="org341aa7a"></a>

### 目录操作函数

-   mkdir
-   rmdir
-   rename
-   chdir
-   getcwd

```c
/**
 // mkdir
 #include<sys/stat.h>
 #include <sys/types.h>

 int mkdir(const char *pathname, mode_t mode);
 参数： mode: 八进制的数

 // rmdir
 #include <unistd.h>

 int rmdir(const char *pathname);
 作用： 删除空目录

 // rename
 #include <stdio.h>

 int rename(const char *oldpath, const char *newpath);

 // chdir
 #include <unistd.h>

 int chdir(const char *path);
 作用：修改进程的工作目录

 // getcwd
 #include <unistd.h>

 char *getcwd(char *buf, size_t size);
 作用： 获取当前的工作目录
 参数： buf:存储的路径，指向的是一个数组（传出参数）
 size: 数组大小
 返回值： 返回的指向的一块内存， 这个数据就是第一个参数
*/

#include <sys/unistd.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main(){
  char buff[128];  // 获取当前的工作目录
  getcwd(buff, sizeof(buff));
  printf("current work path:%s \n", buff);
  
  int ret = chdir("../");  // 修改工作目录
  int fd = open("chdir.txt", O_CREAT | O_RDWR, 0664);
  if(-1 == fd){
    perror("open err");
    return -1;
  }
  close(fd);
   
  char buffCur[128];  // 获取当前的工作目录
  getcwd(buffCur, sizeof(buffCur));
  printf("now current work path: %s\n", buffCur);
  return 0;
}
```


<a id="orgb0494e9"></a>

### 目录遍历函数

-   opendir
-   readdir
-   closedir

```c
/**
   #include <dirent.h>
   #include <sys/types.h>

   DIR *opendir(const char *name);
   参数：name : 需要打开的目录的名称
   返回值： DIR* 类型，理解为名录流，错误返回 NULL;

   #include <dirent.h>

   struct dirent *readdir(DIR *dirp);
   作用： 读取目录中的数据
   参数： dirp 通过 opendir 返回的结果
   返回值：struct dirent :
   代表读取到的文件信息，读取到末尾或失败，返回NULL；

   #include <dirent.h>
   #include <sys/types.h>

   int closedir(DIR *dirp);
   作用： 关闭目录
*/

#include "string.h"
#include <dirent.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>

// 获取目录下所有普通文件的个数
int getFIleNumber(const char *path) {
  DIR *dir = opendir(path);
  if (dir == NULL) {
    perror("opendir");
    exit(0);
  }

  struct dirent *prt;
  int total_number = 0;

  while ((prt = readdir(dir))) {
    char *dname = prt->d_name;  // 获取名称

    // 忽略 ./ 和 ../ 两个目录
    if (strcmp(dname, ".") == 0 || strcmp(dname, "..") == 0) {
      continue;
    }
    
    if (prt->d_type == DT_DIR) {  // 判断是否是普通文件还是目录
      // 目录， 需要继续读取这个目录
      char newPath[256];
      sprintf(newPath, "%s/%s", path, dname);
      total_number += getFIleNumber(newPath);
    }

    if (prt->d_type == DT_REG) {
      ++total_number;  // 普通文件
    }
  }
  
  closedir(dir);  // 关闭目录
  return total_number;
}

// 读取某个目录下文件的个数
int main(int argc, char *argv[]) {
  if (argc < 2) {
    printf("%s path\n", argv[0]);
    return -1;
  }

  int file_numbers = getFIleNumber(argv[1]);
  printf("file numbers: %d\n", file_numbers);
  return 0;
}
```


<a id="org954074a"></a>

### dup 和 dup2 函数

-   dup 复制文件描述符

```c
/**
   #include <unistd.h>

   int dup(int oldfd);
   作用： 复制一个新的文件描述符，和原来的文件描述符指向同一个文件，
   从空闲文件描述符表中找一个最小的，作为新的拷贝的文件描述符

   int dup2(int oldfd, int newfd);
*/

#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  int fd1 = open("a.txt", O_RDWR | O_CREAT, 0664);
  int fd2 = dup(fd1);
  if (fd2 == -1) {
    perror("dup");
    return -1;
  }

  printf("fd1: %d, fd2: %d\n", fd1, fd2);
  close(fd1);

  char *str = "helloworld";
  int ret = write(fd2, str, strlen(str));
  if (-1 == ret) {
    perror("write");
    return -1;
  }

  close(fd2);
  return 0;
}
```

-   dup2 重定向文件描述符

```c
/**

   #include <unistd.h>

   int dup(int oldfd);
   作用： 复制一个新的文件描述符，和原来的文件描述符指向同一个文件，
   从空闲文件描述符表中找一个最小的，作为新的拷贝的文件描述符

   int dup2(int oldfd, int newfd);
   作用： 重定向文件描述符，
   oldfd 指向 a.txt, newfd 指向 b.txt,
   调用函数成功后， newfd 和 b.txt close, newfd 指向了a.txt。 
   oldfd 必须是一个有效的文件描述符, oldfd 和 newfd
   相同，相当于什么都没做
*/

#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  //
  int fd = open("1.txt", O_RDWR | O_CREAT, 0664);
  if (fd == -1) {
    perror("open");
    return -1;
  }

  int fd1 = open("2.txt", O_RDWR | O_CREAT, 0664);
  if (-1 == fd1) {
    perror("open");
    return -1;
  }

  printf("fd: %d, fd1 %d \n", fd, fd1);
  int fd2 = dup2(fd, fd1);
  if (fd2 == -1) {
    perror("dup2");
    return -1;
  }

  // 通过 fd1 去写数据， 实际操作的是 1.txt, 而不是 2.txt
  char str[] = "hello dup2";
  int len = write(fd1, str, strlen(str));
  if (len == -1) {
    perror("write");
    return -1;
  }

  printf("fd: %d, fd1: %d, fd2: %d\n", fd, fd1, fd2);

  close(fd1);
  //  close(fd2);
  close(fd);
  return 0;
}
```


<a id="org606f868"></a>

### fcntl 函数

-   fcntl 复制文件描述符，设置、获取文件的状态标志

```c
/**
   #include <fcntl.h>
   #include <unistd.h>

   int fcntl(int fd, int cmd, ... args);
   参数： fd 需要操作的文件描述符
   cmd : 表示对文件描述符进行如何操作
   1. 复制文件描述符， F_DUPFD
   2. F_GETFL 获取指定文件描述符状态 flag, 获取的flag 和通过 open
   传递的flag 是一个东西。
   3. F_SETFL 设置文件描述符状态 flag, 必选项 O_RDONLY, O_WRONLY, O_RDWR,
   可选项： O_APPEND, NONBLOCK, O_APPEND 表示追加数据， NONBLOK 设置能非阻塞。

   阻塞和非阻塞： 描述的是函数调用的行为。
*/

#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  // 1. 复制文件描述符
  // int fd = open("1.txt", O_RDONLY);
  // int ret = fcntl(fd , F_DUPFD);
  // 2. 修改或者获取文件状态 flag
  int fd = open("1.txt", O_RDWR);
  if (-1 == fd) {
    perror("open");
    return -1;
  }

  int flag = fcntl(fd, F_GETFL);  // 获取文件描述符的状态
  if (-1 == flag) {
    perror("fcntl");
    return -1;
  }

  flag |= O_APPEND;

  // 修改文件描述符状态的 flag, 给 flag 加入 O_APPEND 这个标记
  int ret = fcntl(fd, F_SETFL, flag);
  if (-1 == ret) {
    perror("fcntl");
    return -1;
  }
  char str[] = "\nhello world from fcntl\n";
  write(fd, str, strlen(str));
  close(fd);
  return 0;
}
```
