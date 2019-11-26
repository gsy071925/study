# 常用命令/工具介绍
jps：查看本机java进程信息。  
jstack：打印线程的栈信息，制作线程dump文件。  
jmap：打印内存映射，制作堆dump文件  
jstat：性能监控工具  
jhat：内存分析工具  
jconsole：简易的可视化控制台  
jvisualvm：功能强大的控制台  

## JAVA Dump
JAVA Dump：就是虚拟机运行时的快照，将虚拟机运行时的状态和信息保存到文件中  
线程dump：包含所有线程的运行状态，纯文本格式  
堆dump：包含所有堆对象的状态，二进制格式  

# jps
显示当前所有java进程pid的命令，我们可以通过这个命令来查看到底启动了几个java进程（因为每一个java程序都会独占一个java虚拟机实例），不过jps有个缺点是只能显示当前用户的进程id，要显示其他用户的还只能用linux的ps命令。

## 示例
jps -help

jps -l 输出应用程序main.class的完整package名或者应用程序jar文件完整路径名

jps -v 输出传递给JVM的参数

## jps失效
我们在定位问题过程会遇到这样一种情况，用jps查看不到进程id，用ps -ef | grep java却能看到启动的java进程。  
要解释这种现象，先来了解下jps的实现机制：  
java程序启动后，会在目录/tmp/hsperfdata_{userName}/下生成几个文件，文件名就是java进程的pid，因此jps列出进程id就是把这个目录下的文件名列一下而已，至于系统参数，则是读取文件中的内容。  
我们来思考下：如果由于磁盘满了，无法创建这些文件，或者用户对这些文件没有读的权限。又或者因为某种原因这些文件或者目录被清除，出现以上这些情况，就会导致jps命令失效。

如果jps命令失效，而我们又要获取pid，还可以使用以下两种方法：  
1、top | grep java  
2、ps -ef |grep java

# jstack
主要用于生成指定进程当前时刻的线程快照，线程快照是当前java虚拟机每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致长时间等待。

## 示例
jstack {pid}
jstack {pid} > {pid}.log


# jmap
主要用于打印指定java进程的共享对象内存映射或堆内存细节。
堆Dump是反映堆使用情况的内存镜像，其中主要包括系统信息、虚拟机属性、完整的线程Dump、所有类和对象的状态等。一般在内存不足，GC异常等情况下，我们会去怀疑内存泄漏，这个时候就会去打印堆Dump。
## 用法
jmap -help
## 示例
jmap {pid}
jmap -heap {pid}   查看堆使用情况
jmap -histo {pid}  查看堆中对象数量和大小
jmap -dump:format=b,file={pid}.bin {pid}  将内存使用的详细情况输出到文件


# jstat
主要是对java应用程序的资源和性能进行实时的命令行监控，包括了对heap size和垃圾回收状况的监控。
## 用法
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
option：  我们经常使用的选项有gc、gcutil
vmid：    java进程id
interval：间隔时间，单位为毫秒
count：   打印次数

## 示例
jstat -gc PID 5000 20
### 主要字段说明
S0C： 年轻代第一个survivor的容量（字节）  
S1C： 年轻代第二个survivor的容量（字节）  
S0U： 年轻代第一个survivor已使用的容量（字节）  
S1U： 年轻代第二个survivor已使用的容量（字节）  
EC：  年轻代中Eden的空间（字节）  
EU：  年代代中Eden已使用的空间（字节）  
OC：  老年代的容量（字节）  
OU：  老年代中已使用的空间（字节）  
YGC： 从应用程序启动到采样时年轻代中GC的次数  
YGCT：从应用程序启动到采样时年轻代中GC所使用的时间（单位：S）  
FGC： 从应用程序启动到采样时老年代中GC（FULL GC）的次数  
FGCT：从应用程序启动到采样时老年代中GC所使用的时间（单位：S）  


# jhat

# jinfo
jinfo可以用来查看正在运行的java运用程序的扩展参数，甚至支持在运行时动态地更改部分参数。
## 用法
基本使用语法如下： jinfo -<option> <pid> ，其中option可以为以下信息：
-flag <name>: 打印指定java虚拟机的参数值  
-flag [+|-]<name>：设置或取消指定java虚拟机参数的布尔值  
-flag <name>=<value>：设置指定java虚拟机的参数的值
## 示例
/data/program/jdk/bin/jinfo -flag UseConcMarkSweepGC {pid}  
/data/program/jdk/bin/jinfo -flag -UseConcMarkSweepGC {pid}  
/data/program/jdk/bin/jinfo -flag SurvivorRatio=8 {pid}
/data/program/jdk/bin/jinfo -flag CMSInitiatingOccupancyFraction=65 25770

# jcmd
