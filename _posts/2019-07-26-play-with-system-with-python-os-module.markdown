---
layout: post
title: "Play with system with python os module"
date: 2019-7-26
tag: python os time
---
# Table of Contents

1.  [OS 模块](#orgf7b3469)
2.  [文件](#org7619ce1)
    1.  [open](#orgf430668)
3.  [目录](#orgc90e9c7)
4.  [程序和进程](#org19d02fa)
    1.  [使用subprocess创建进程](#org4ee2a5b)
    2.  [使用 multiprocessing 创建进程](#orge04c5fa)
    3.  [使用 terminate() 终止程序](#org98a8ab4)
5.  [日期和时间](#org4b19846)
    1.  [datetime 模块](#orgfb47bf4)
    2.  [time 模块](#org1fbebb8)
    3.  [读写日期和时间](#org5643ca2)
    4.  [locale](#org5e8dbae)


<a id="orgf7b3469"></a>

# OS 模块

operating system


<a id="org7619ce1"></a>

# 文件


<a id="orgf430668"></a>

## open

    import os
    
    fout = open('oops.txt','wt')
    print('Oops, I create a file.', file = fout)
    fout.close()

用这个文件来进行一些测试

    os.path.exists('oops.txt')
    os.path.exists('./oops.txt')
    os.path.exists('waffles')
    os.path.exists('.')
    os.path.exists('..')

-   os.path.exists() 检查文件是否存在
-   os.path.isfile() 检查是否为文件
-   os.path.ifabs()  判断参数是否是绝对路径

    name = 'oops.txt'
    print(os.path.isfile(name))
    print(os.path.isdir(name))
    print(os.path.isabs('/name'))
    print(os.path.isabs('name'))

-   shutil.copy 复制文件
-   shutil.move 复制一个文件并删除源文件

    import shutil
    shutil.copy('oops.txt','ohno.txt')
    print(os.path.exists('ohno.txt'))
    os.rename('ohno.txt','ohwell.txt')
    print(os.path.exists('ohno.txt'),os.path.exists('ohwell.txt'))

-   os.link() 创建硬链接
-   os.symlink() 创建符号链接

    os.link('oops.txt','yikes.txt')
    os.symlink('oops.txt','jeepers.txt')
    print(os.path.isfile('yikes.txt'),os.path.islink('jeepers.txt'))

-   os.chmod() 修改权限
-   os.chown() 修改所有者
-   os.path.abspath() 获取路径名
-   os.path.realpath() 获取符号链接的路径名
-   os.remove() 删除文件

    os.chmod('oops.txt',0o400)
    uid = 5
    gid = 22 
    #os.chown('oops.txt',uid,gid)
    print("abspath: ", os.path.abspath('oops.txt'), "\nrealpath: ",\
          os.path.realpath('jeepers.txt'))
    os.remove('oops.txt')
    print(os.path.exists('oops.txt'))


<a id="orgc90e9c7"></a>

# 目录

-   os.mkdir() 创建目录
-   os.rmdir() 删除目录
-   os.listdir() 列出目录内容
-   os.chdir() 修改当前目录
-   glob.glob() 列出匹配的文件

    import os
    os.mkdir('poem')
    print(os.path.exists('poem'))
    os.rmdir('poem')
    print(os.path.exists('poem'))
    
    os.mkdir('poem')
    print(os.listdir('poem'))
    os.mkdir('poem/mcintyre')
    print(os.listdir('poem'))
    
    fout = open('poem/mcintyre/the_good_main','wt')
    fout.write('''Cheerful and heppy was his mood,
    He to the poor was kind and good.''')
    fout.close()
    
    print(os.listdir('poem/mcintyre'))
    os.chdir('poem')
    print(os.listdir('mcintyre'))
    
    import glob
    print(glob.glob('m*'),glob.glob('??'),glob.glob('m??????e'),glob.glob('[klm]*e'))


<a id="org19d02fa"></a>

# 程序和进程

当运行一个程序时，操作系统会创建一个进程。它会使用系统资源（CPU，内存，和磁盘空间）和操作系统内核中的数据结构（文件，
网络连接，用量统计等）。进程之间是相互隔离的即一个进程无法访问其他进程的内容，也无法操作其他进程。

-   os.getpid() 获取正在运行的python解释器的进程号
-   os.getcwd() 获取当前的工作目录
-   os.getuid() 获取我的用户ID
-   os.getgid() 获取我的用户组ID

    import os 
    print("pid: ", os.getpid(), " \ncwd: ",os.getcwd(), " \nuid ", os.getuid(),\
    "gid: ", os.getgid())


<a id="org4ee2a5b"></a>

## 使用subprocess创建进程

-   subprocess.getoutput() 参数是字符串，可以表示完整的shell命令
-   subprocess.check<sub>output</sub>() 可以接受一个命令和参数列表，默认情况下返回字节类型的标准输出，这个函数没有用shell
-   subprocess.getstatusoutput() 获取其他程序的退出状态，返回一个包含状态码和输出的元组。
-   subprocess.call() 获取推出状态

    import subprocess
    print('date: ',subprocess.getoutput('date'),\
          '\n使用管道： ',subprocess.getoutput('date -u | wc'),\
          "\ndate: ", subprocess.check_output(['date','-u']),\
          "\n获取退出状态： ",subprocess.getstatusoutput('date'),\
          "\n只想要退出状态： ", subprocess.call('date'),
          "\n运行带参数的程序：date -u ",subprocess.call('date -u',shell=True),\
          "\n运行带参数的程序：date -u ",subprocess.call(['date','-u']))


<a id="orge04c5fa"></a>

## 使用 multiprocessing 创建进程

-   multiprocessing.Process(target= ,args=( ,)) 创建一个新的进程

    import os
    import multiprocessing
    
    def whoami(what):
        print("Process %s says: %s" % (os.getpid(), what))
    
    whoami("I'm the main program")
    for n in range(4):
        p = multiprocessing.Process(target=whoami,args=("I'm function %s" % n,))
        p.start()


<a id="org98a8ab4"></a>

## 使用 terminate() 终止程序

    import os
    import multiprocessing
    import time
    
    def whoami(what):
        print("I'm %s, in process %s " % (what, os.getpid()))
    
    def loopy(name):
        whoami(name)
        start = 1
        stop = 1000000
        for num in range(stop):
    	print("\tNumber %s of %s. Honk!" % (num, stop))
    	time.sleep(1)
    
    whoami('main')
    p = multiprocessing.Process(target=loopy,args=('loopy',))
    p.start()
    time.sleep(6)
    p.terminate()


<a id="org4b19846"></a>

# 日期和时间

python 的标准库中有很多和日期和时间的模块： datetime, time, calendar, dateutil. 等等。

-   calendar.isleap() 检测是否是闰年


<a id="orgfb47bf4"></a>

## datetime 模块

定义了四个主要的对象：

-   date 处理年，月，日
-   time 处理时，分，妙，毫秒
-   datetime 处理日期和时间同时出现的情况
-   datedelta 处理日期和/或时间间隔
    
        from datetime import date
        halloween = date(2017,9,10)
        print('halloween: ',halloween,\
              "\nday: ",halloween.day,\
              "\nmonth: ",halloween.month,\
              "\nyear: ",halloween.year,\
              "\nisoformat: ",halloween.isoformat(),\
              "\n")
        
        now = date.today()
        
        from datetime import timedelta
        one_day = timedelta(days = 1)
        tomorrow = now + one_day
        yesterday = now - one_day
        
        print('now: ', now,\
              "\ntomorrow: ", tomorrow,\
              "\nnow + 17days: ", now + 17 * one_day,\
              "\nyesterday: ", yesterday,\
              "\ndate.min: ",date.min,\
              "\ndate.max: ",date.max,\
              "\n"
        )
        
        from datetime import time
        
        noon = time(12,0,0)
        
        print("noon: ",noon,\
              "\nnoon.hour: ",noon.hour,\
              "\nnoon.minute: ",noon.minute,\
              "\nnoon.second: ",noon.second,\
              "\nnoon.microsecond",noon.microsecond,\
              "\n")
        
        from datetime import datetime
        
        some_day = datetime(2017,5,7,2,45,5,8)
        now = datetime.now()
        
        print('some_day: ',some_day,\
              "\nnow: ",now,\
              '\nisoformat: ',some_day.isoformat(),\
              "\nnow.month: ",now.month,\
              "\nnow.day: ",now.day,\
              "\nnow.hour: ",now.hour,\
              "\nnow.minute: ",now.minute,\
              "\nnow.second: ",now.second,\
              "\nnow.microsecond: ",now.microsecond,\
              "\n"
        )
        
        noon = time(12)
        this_today = date.today()
        noon_today = datetime.combine(this_today,noon)
        print('combine time: ',noon_today,\
              "\ndate: ",noon_today.date(),\
              "\ntime: ",noon_today.time()
        )


<a id="org1fbebb8"></a>

## time 模块

python 有一个单独的 **time** 模块，纪元值是从1970年1月1日0点开始的秒数。

-   time.time() 返回当前时间的纪元值
-   time.ctime() 把一个纪元值转换成字符串
-   time.localtime() 返回当前系统时区下的时间
-   time.gmtime() 返回UTC时间
-   time.mktime() 将struct<sub>time</sub> 对象转换回纪元值，但struct<sub>time只能精确到妙</sub>。

    import time
    
    now = time.time()
    time_str = time.ctime(now)
    tm = time.localtime(now)
    
    print("now: ",now,\
          "\n字符串： ",time_str,\
          "\n当前系统时区下的时间： ",tm,\
          "\nUTC时间： ",time.gmtime(now),\
          "\n从struct_time返回纪元值： ",time.mktime(tm))


<a id="org5643ca2"></a>

## 读写日期和时间

-   strftime() 把时间和日期转换成字符串
-   strptime() 把字符串转换成日期或时间
    
        import time
        
        now = time.time()
        str_time = time.ctime(now)
        
        fmt = "It's %A, %B %d, %Y, local time %I:%M:%S%p"
        t = time.localtime(now)
        str_time_format = time.strftime(fmt,t)
        print("当前时间： ",str_time,
              "\ntime.strftime 转换: ",str_time_format,"\n")
        
        from datetime import date
        some_day = date(2014,7,9)
        fmt = "It's %B %d, %Y, local time %I:%M:%S%p"
        str_time_format1 = some_day.strftime(fmt)
        print("date 对象时间：",some_day,
              "\nstrftime 转换： ",str_time_format1,"\n")
        
        
        from datetime import time
        some_time = time(10,35)
        str_time_format2 = some_time.strftime(fmt)
        print("time 对象时间：",some_time,
              "\nstrftime 转换： ",str_time_format2,"\n")
        
        import time
        fmt = "%Y-%m-%d"
        print("字符串转换成日期或时间： \n",time.strptime("2012-01-29",fmt))


<a id="org5e8dbae"></a>

## locale

    import locale
    from datetime import date
    
    names = locale.locale_alias.keys()
    good_names = [ name for name in names if len(name) == 5 and name[2] == '_']
    print(good_names[:5])
    
    de = [ name for name in good_names if name.startswith('de')]
    print(de)
    
    halloween = date(2014,10,31)
    for lang_country in ['en_us','de_de']:
        # locale.setlocale(locale.LC_TIME, lang_country)  
        halloween.strftime('%A, %B, %d')


