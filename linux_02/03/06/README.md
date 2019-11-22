[TOC]



## 1、超线程技术 (Hyper-Threading)

- 1、就是利用特殊的 **硬件指令**，把 **1个物理芯片** 模拟为 **2个逻辑内核(CPU core)** 
- 2、让 **单个CPU处理器** 都能使用 **线程级** 的并行计算
- 3、进而兼容多线程操作系统和软件，减少了CPU的闲置时间，提高的CPU的运行效率
- 4、我们常听到的 **双核四线程/四核八线程** 指的就是支持 **超线程技术** 的CPU



## 2、物理CPU

- 1、机器上安装的 **实际 CPU 硬件**
- 2、比如说你的主板上安装了一个 **8核 CPU** => 只有有 **1个 物理 CPU**



## 3、逻辑CPU

- 1、一颗CPU硬件，可以有多个 **物理核**
- 2、通过 intel 超线程技术(Hyper-Threading)，可以在 **逻辑上** 虚拟出  `物理核 * 2` 个数的核处理处理数据




## 4、逻辑 CPU 核, 个数计算

```
逻辑CPU数量 = 物理CPU硬件数量 X `CPU cores` X `2(如果支持并开启HT)`
```

注意：

- 1、前提是 **所有的CPU硬件型号** 必须一致
- 2、如果不一致只能一个一个的加起来，不用直接乘以物理CPU数量
- 3、比如你的电脑安装了 **一块4核CPU**，并且支持且开启了 **超线程（HT）技术**，那么 **逻辑CPU数量 = 1 × 4 × 2 = 8**



## 5、linux下查看 CPU 相关信息

> CPU 信息主要都在 /proc/cupinfo 虚拟文件系统内.

### 1. 查看物理 ==CPU 个数==

```
cat /proc/cpuinfo | grep "physical id" | sort -u | wc -l
```

eg:

```
xzh@xzh-VirtualBox:~/share$ cat /proc/cpuinfo | grep "physical id" | sort -u | wc -l
1
```

### 2. 查看 每个物理CPU 中的  ==core 个数 (物理核数)==

```
cat /proc/cpuinfo | grep "cpu cores" | uniq
```

eg:

```
xzh@xzh-VirtualBox:~/share$ cat /proc/cpuinfo | grep "cpu cores" | uniq
cpu cores	: 1
```

### 3. 查看 ==逻辑 CPU== 个数

```
cat /proc/cpuinfo | grep "processor" | wc -l
```

eg:

```
xzh@xzh-VirtualBox:~/share$ cat /proc/cpuinfo | grep "processor" | wc -l
1
```

### 4. 查看CPU的名称型号

```
cat /proc/cpuinfo | grep "name" | cut -f2 -d: | uniq
```

eg:

```
xzh@xzh-VirtualBox:~/share$ cat /proc/cpuinfo | grep "name" | cut -f2 -d: | uniq
 Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
```



## 6、linux 查看 ==某个进程== 运行在哪个 ==逻辑 CPU==

### 1. 命令格式

```
ps -eo pid,args,psr
```

参数的含义

```
pid  - 进程ID
args - 该进程执行时传入的命令行参数
psr  - 分配给进程的逻辑CPU
```

### 2. 查看 nginx 绑定进程到指定的逻辑核

```
[~]# ps -eo pid,args,psr | grep nginx
nginx: master process /usr/   1
nginx: worker process         0
nginx: worker process         1
nginx: worker process         2
nginx: worker process         3
grep nginx                   3
```



## 7、查看 线程id（TID）

```
linux_thread_线程与进程.md
```



## 8、CPU affinity（亲和力）

- 1、CPU affinity 是一种 **调度属性** (scheduler property)
- 2、可以指定 **一个进程** 绑定到 **一个或一组逻辑CPU** 上运行
- 3、Linux 调度器(scheduler)会根据 **CPU affinity** 设置，让该进程运行在"绑定"的CPU上，而不会在别的CPU上运行

- 4、Linux调度器同样支持自然CPU亲和性(natural CPU affinity)
  - 1) 调度器会试图保持进程在相同的CPU上运行
  - 2) 进程不会在处理器之间频繁迁移
  - 3) 进程迁移的频率小就意味着产生的负载小

- 5、指定程序运行时的 CPU 亲和力 的作用：
  - 1) 因为程序的作者比调度器更了解程序
  - 2) 所以我们可以手动地为其分配CPU核
  - 3) 而不会过多地占用CPU0（某一个CPU核）
  - 4) 或是让我们关键进程和一堆别的进程挤在一起
  - 5) 设置CPU亲和性可以使某些程序提高性能



## 9、CPU affinity 位掩码(bitmask)

- 1、CPU affinity 使用位掩码(bitmask)表示
- 2、**每一个二进制位** 都表示 **一个CPU**, 置 **1** 表示 **绑定**
- 3、最低位 表示 第一个 逻辑CPU
- 4、最高位 表示 最后一个 逻辑CPU
- 5、CPU affinity 典型的表示方法是使用 **16进制（0x...）**

```
0x00000001
  is processor #cpu0

0x00000003（二进制 11）
  is processors #cpu0 and #cpu1

0xFFFFFFFF（二进制 1111.........1111（32个1））
  is all processors (#cpu0 through #cpu31)
```



## 10、 taskset 命名用于获取或者设定CPU亲和性

### 1. 使用指定的CPU亲和性运行一个新程序

使用 **CPU0** 运行 **ls 可执行文件** 显示/etc/init.d下的所有内容:

```shell
taskset -c 0 ls -al /etc/init.d/
```

-c, --cpu-list: 声明CPU的亲和力使用数字表示而不是用位掩码表示. 例如 0,5,7,9-11.

### 2. 查看init进程(PID=1)的CPU亲和性

```shell
xzh@xzh-VirtualBox:~/share$ taskset -p 1
pid 1's current affinity mask: 1
```

### 3. 改变已经运行进程的CPU亲和力

#### 1. 在第一个终端: 运行top命令

```shell
xzh@xzh-VirtualBox:~$ top
.....................
```

#### 2. 在第二个终端: 获取top命令的pid和其所运行的CPU号

```shell
xzh@xzh-VirtualBox:~$ ps -eo pid,args,psr | grep top
 1197 nautilus-desktop              0
 3378 top                           0
 3382 grep --color=auto top         0
xzh@xzh-VirtualBox:~$
```

#### 3. 在第二个终端: 更改top命令运行的CPU号

格式：

```shell
taskset -cp 新的CPU号 pid
```

示例：

```shell
taskset -cp 1 3378
```

#### 4. 在第二个终端: 查看是否更改成功

```shell
ps -eo pid,args,psr | grep top
```



## 11、获取 CPU 亲和力 API

### 1. 不带 `_S` 宏定义

```c
#define _GNU_SOURCE
#include <sched.h>
#include <unistd.h> /* sysconf */
#include <stdlib.h> /* exit */
#include <stdio.h>

int main(void)
{
  int i, nrcpus;
  cpu_set_t mask;
  unsigned long bitmask = 0;

  // 1、init mask set
  CPU_ZERO(&mask);

  // 2、Get the CPU affinity for a pid，写入到 mask 对应的二进制位
  if (sched_getaffinity(0, sizeof(cpu_set_t), &mask) == -1) 
  {   
    perror("sched_getaffinity");
    exit(EXIT_FAILURE);
  }

  // 3、get logical cpu number
  nrcpus = sysconf(_SC_NPROCESSORS_CONF);
  printf("nrcpus = %d\n", nrcpus);

  // 4、打印 mask 每一个二进制位是否 `置1`
  // 重点: 如果 `置1`, 则表示 `对应CPU` 被绑定运行 `当前程序进程` （设置了CPU亲和力）
  for (i = 0; i < nrcpus; i++)
  {
    if (CPU_ISSET(i, &mask))
    {
      bitmask |= (unsigned long)0x01 << i; // 将获取到对应二进制位，保存到 bitmask 
      printf("processor #%d is set\n", i);
    }
  }

  // 5、以 `十六进制` 打印 mask
  printf("bitmask = %#lx\n", bitmask);

  exit(EXIT_SUCCESS);
}
```

```
xzh@xzh-VirtualBox:~/share$ gcc main.c
xzh@xzh-VirtualBox:~/share$ ./a.out
nrcpus = 1
processor #0 is set
bitmask = 0x1
```

### 2. 带 `_S` 宏定义

```c
#define _GNU_SOURCE
#include <sched.h>
#include <unistd.h> /* sysconf */
#include <stdlib.h> /* exit */
#include <stdio.h>

int main(void)
{
  int i, nrcpus;
  cpu_set_t *pmask;
  size_t cpusize;
  unsigned long bitmask = 0;

  /* get logical cpu number */
  nrcpus = sysconf(_SC_NPROCESSORS_CONF);

  pmask = CPU_ALLOC(nrcpus);
  cpusize = CPU_ALLOC_SIZE(nrcpus);
  CPU_ZERO_S(cpusize, pmask);

  /* Get the CPU affinity for a pid */
  if (sched_getaffinity(0, cpusize, pmask) == -1) 
  {
    perror("sched_getaffinity");
    CPU_FREE(pmask);
    exit(EXIT_FAILURE);
  }
  for (i = 0; i < nrcpus; i++)
  {
    if (CPU_ISSET_S(i, cpusize, pmask))
    {
      bitmask |= (unsigned long)0x01 << i;
      printf("processor #%d is set\n", i); 
    }
  }
  printf("bitmask = %#lx\n", bitmask);

  CPU_FREE(pmask);
  exit(EXIT_SUCCESS);
}
```


在 **4核** CPU 下可以如下测试结果：

```
[cpu_affinity #2]$ taskset 1 ./a.out 
processor #0 is set
bitmask = 0x1
```

```
[cpu_affinity #4]$ taskset 2 ./a.out 
processor #1 is set
bitmask = 0x2
```

- 2 => 10 => 第一个 CPU 被置位

```
[cpu_affinity #5]$ taskset 3 ./a.out 
processor #0 is set
processor #1 is set
bitmask = 0x3
```

- 3 => 11 => 第一个 和 第二个 总共 2个 CPU 被置位

```
[cpu_affinity #6]$ taskset 4 ./a.out 
processor #2 is set
bitmask = 0x4
```

- 4 => 100 => 第二个 CPU 被置位

```
[cpu_affinity #7]$ taskset 5 ./a.out 
processor #0 is set
processor #2 is set
bitmask = 0x5
```

- 5 => 101 => 第一个 和 第三个 总共 2个 CPU 被置位

```
[cpu_affinity #8]$ taskset 6 ./a.out 
processor #1 is set
processor #2 is set
bitmask = 0x6
```

- 6 => 110 => 第二个 和 第三个 总共 2个 CPU 被置位

```
[cpu_affinity #9]$ taskset 7 ./a.out 
processor #0 is set
processor #1 is set
processor #2 is set
bitmask = 0x7
```

- 7 => 111 => 第一个、第二个、第三个 总共 3个 CPU 被置位

```
[cpu_affinity #10]$ taskset 8 ./a.out 
processor #3 is set
bitmask = 0x8
```

```
[cpu_affinity #11]$ taskset 9 ./a.out 
processor #0 is set
processor #3 is set
bitmask = 0x9
```

```
[cpu_affinity #12]$ taskset A ./a.out 
processor #1 is set
processor #3 is set
bitmask = 0xa
```

```
[cpu_affinity #13]$ taskset B ./a.out 
processor #0 is set
processor #1 is set
processor #3 is set
bitmask = 0xb
```

```
[cpu_affinity #14]$ taskset C ./a.out 
processor #2 is set
processor #3 is set
bitmask = 0xc
```

```
[cpu_affinity #15]$ taskset D ./a.out 
processor #0 is set
processor #2 is set
processor #3 is set
bitmask = 0xd
```

```
[cpu_affinity #16]$ taskset E ./a.out 
processor #1 is set
processor #2 is set
processor #3 is set
bitmask = 0xe
```

```
[cpu_affinity #17]$ taskset F ./a.out 
processor #0 is set
processor #1 is set
processor #2 is set
processor #3 is set
bitmask = 0xf
```

```
[cpu_affinity #18]$ taskset 0 ./a.out 
sched_setaffinity: Invalid argument
failed to set pid 0's affinity.
```



## 12、设置 ==进程== 的CPU亲和性, 再获取显示CPU亲和性

```c
#define _GNU_SOURCE
#include <sched.h>
#include <unistd.h> /* sysconf */
#include <stdlib.h> /* exit */
#include <stdio.h>

int main(void)
{
  int i, nrcpus;
  cpu_set_t mask;
  unsigned long bitmask = 0;

  // 1、
  CPU_ZERO(&mask);

  // 2、设置 `当前进程` 可以在 `CPU0 与 CPU2` 上运行
  CPU_SET(0, &mask); /* add CPU0 to cpu set */
  CPU_SET(2, &mask); /* add CPU2 to cpu set */

  // 3、sched_setaffinity() 设置 `当前进程` 的 CPU affinity 
  if (sched_setaffinity(0, sizeof(cpu_set_t), &mask) == -1) 
  {   
    perror("sched_setaffinity");
    exit(EXIT_FAILURE);
  }

  // 4、
  CPU_ZERO(&mask);

  // 5、Get the CPU affinity for a pid
  if (sched_getaffinity(0, sizeof(cpu_set_t), &mask) == -1) 
  {   
    perror("sched_getaffinity");
    exit(EXIT_FAILURE);
  }

  // 6、get logical cpu number
  nrcpus = sysconf(_SC_NPROCESSORS_CONF);

  // 7、
  for (i = 0; i < nrcpus; i++)
  {
    if (CPU_ISSET(i, &mask))
    {
      bitmask |= (unsigned long)0x01 << i;
      printf("processor #%d is set\n", i); 
    }
  }

  // 8、
  printf("bitmask = %#lx\n", bitmask);

  exit(EXIT_SUCCESS);
}
```



## 13、设置 ==线程== 的CPU属性, 再获取显示CPU亲和性

### 1. pthread api

```c
#define _GNU_SOURCE
#include <pthread.h> // 不用再包含 <sched.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

#define handle_error_en(en, msg) \
  do { \
    errno = en; 
    perror(msg); \
    exit(EXIT_FAILURE); \
  } while (0)

int
main(int argc, char *argv[])
{
  int s, j;
  cpu_set_t cpuset;
  pthread_t thread;

  // 1、
  thread = pthread_self();

  // 2、Set affinity mask to include CPUs 0 to 7
  CPU_ZERO(&cpuset);
  for (j = 0; j < 8; j++)
    CPU_SET(j, &cpuset);

  // 3、pthread_setaffinity_np() 设置 `当前线程` 的CPU亲和力
  s = pthread_setaffinity_np(thread, sizeof(cpu_set_t), &cpuset);
  if (s != 0)
  {
    handle_error_en(s, "pthread_setaffinity_np");
  }

  // 4、Check the actual affinity mask assigned to the thread 
  s = pthread_getaffinity_np(thread, sizeof(cpu_set_t), &cpuset);
  if (s != 0)
  {
    handle_error_en(s, "pthread_getaffinity_np");
  }

  // 5、
  printf("Set returned by pthread_getaffinity_np() contained:\n");
  for (j = 0; j < CPU_SETSIZE; j++) //CPU_SETSIZE 是定义在<sched.h>中的宏，通常是1024
  {
    if (CPU_ISSET(j, &cpuset))
    {
      printf("    CPU %d\n", j);
    }
  }
  exit(EXIT_SUCCESS);
}
```

### 2. linux api

```c
#define _GNU_SOURCE
#include <sched.h>
#include <stdlib.h>
#include <sys/syscall.h> // syscall

int main(void)
{
  pid_t tid;
  int i, nrcpus;
  cpu_set_t mask;
  unsigned long bitmask = 0;

  CPU_ZERO(&mask);
  CPU_SET(0, &mask); /* add CPU0 to cpu set */
  CPU_SET(2, &mask); /* add CPU2 to cpu set */

  /**
   * get tid(线程的PID，线程是轻量级进程，所以其本质是一个进程)
   */
  tid = syscall(__NR_gettid); // or syscall(SYS_gettid);

  /* Set the CPU affinity for a pid */
  if (sched_setaffinity(tid, sizeof(cpu_set_t), &mask) == -1) 
  {
    perror("sched_setaffinity");
    exit(EXIT_FAILURE);
  }

  exit(EXIT_SUCCESS);
}
```