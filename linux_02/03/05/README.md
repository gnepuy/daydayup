[TOC]



## 1、linux 作业

- 1、一个作业（job）来包含 n 个进程（ ls -l | wc -l )
- 2、前台：同一时刻只有 `1个` 作业执行
- 3、后台：同一时刻只有 `n个` 作业执行



## 2、进程 - ==组==

### 1. 定义

- 1、一个或多个进程的集合
- 2、进程组ID: 正整数

### 2. api

#### 1. getpgrp() 获取`当前进程`的进程组id

```c
pid_t getpgrp(void);
```

#### 2. getpgid(pid) 获取`入参 pid 进程`的进程组id

```c
pid_t getpgid(pid_t pid);
```

### 3. getpgid(0) 或 getpgid(子进程id) 等价于 getpgrp()

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
	printf("getpid() = %d\n", getpid());      // 当前进程id
	printf("getpgrp() = %d\n", getpgrp());    // 进程组id
	printf("getpgid(0) = %d\n", getpgid(0));  // 进程组id
	printf("getpgid(getpid()) = %d\n", getpgid(getpid())); // 进程组id

	return 0;
}
```

```
xzh@xzh-VirtualBox:~/share$ gcc main.c
xzh@xzh-VirtualBox:~/share$ ./a.out
getpid() = 3095
getpgrp() = 3095
getpgid(0) = 3095
getpgid(getpid()) = 3095
xzh@xzh-VirtualBox:~/share$
```

### 4. 进程组id = 父进程id（父进程 => 组长进程）

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
  pid_t pid;

  if ((pid=fork())<0)
  {
    printf("fork error!");
  }
  else if (pid==0)
  {
    /**
     * 子进程回调
     */
    printf("-------------- 1 --------------\n");
    printf("getpid() = %d\n", getpid());
    printf("getpgrp() = %d\n", getpgrp());
    printf("getpgid(0) = %d\n", getpgid(0));
    printf("getpgid(getpid()) = %d\n", getpgid(getpid()));
    exit(0);
  }

  /**
   * 父进程回调
   */

  sleep(3);
  printf("-------------- 2 --------------\n");
  printf("getpid() = %d\n", getpid());
  printf("getpgrp() = %d\n", getpgrp());

  return 0;
}
```

```
xzh@xzh-VirtualBox:~/share$ gcc main.c
xzh@xzh-VirtualBox:~/share$ ./a.out
-------------- 1 --------------
getpid() = 3058
getpgrp() = 3057
getpgid(0) = 3057
getpgid(getpid()) = 3057
-------------- 2 --------------
getpid() = 3057
getpgrp() = 3057
xzh@xzh-VirtualBox:~/share$
```

### 5. 组长进程 与 进程组

- 1、组长进程标识: 其进程组ID==其进程ID
- 2、组长进程可以创建一个进程组，创建该进程组中的进程，然后终止
- 3、只要进程组中有一个进程存在，进程组就存在
- 4、与 **组长进程** 是否终止 **无关**
- 5、进程组生存期: 进程组创建到最后一个进程离开(终止或转移到另一个进程组)

### 6. 一个进程可以为自己或子进程设置进程组ID

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() 
{
  pid_t pid;

  if ((pid=fork())<0) 
  {
    printf("fork error!");
    exit(1);
  }
  else if (pid==0) 
  {
    printf("The child process PID is %d.\n", getpid());
    printf("The Group ID of child is %d.\n", getpgid(0));
    sleep(5);

    printf("The Group ID of child is changed to %d.\n", getpgid(0));
    exit(0);
  }

  sleep(1);

  // 改变 `子进程` 的 `组id` 为 `子进程本身`
  setpgid(pid, pid); 
  
  sleep(5);
  printf("The parent process PID is %d.\n", getpid());
  printf("The parent of parent process PID is %d.\n", getppid());
  printf("The Group ID of parent is %d.\n", getpgid(0));

  // 改变 `父进程` 的 `组id` 为 `父进程的父进程`
  setpgid(getpid(), getppid()); 
  printf("The Group ID of parent is changed to %d.\n", getpgid(0));

  return 0;
}
```



## 3、进程 - ==会话==

### 1. 会话

- 1、一个进程组：包含n个 **进程**
- 2、一个进程会话：包含n个 **进程组**

### 2. 创建会话的注意点

- 1、进程组的 **组长（父亲）进程**，不能够创建会话
- 2、创建会话的进程，会变为会话的 **组长**
- 3、创建会话的进程，需要 **root权限**（linux需要，ubuntu不需要）
- 4、创建会话的进程所在的 **终端** 会被 **丢弃**

### 3. 创建进程会话的步骤

- 1、fork() 创建 **子进程**
- 2、**结束 父进程** 执行
- 3、**子进程** 调用 **setsid()**

### 4. 示例

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main()
{   
  pid_t pid;

  printf("getpid() = %d\n", getpid());
  printf("getpgid() = %d\n", getpgid(0));
  printf("getsid() = %d\n", getsid(0));
  printf("-------------------------------\n");

  // 1、创建子进程
  pid = fork();

  2、结束父进程
  if (pid > 0)
    exit(0);

  // 3、子进程回调
  if (0 == pid)
  {
    // 子进程创建会话
    setsid();

    // 打印信息
    printf("getpid() = %d\n", getpid());      // 当前进程id
    printf("getpgid() = %d\n", getpgid(0));   // 进程组id
    printf("getsid() = %d\n", getsid(0));     // 进程会话id

    // 子进程创建会话并且休眠20秒后才会return结束
    sleep(20);
  }
  
  exit(0);
}
```

```
➜  signal gcc main.c
➜  signal ./a.out
getpid() = 8459
getpgid() = 8459
getsid() = 3427
-------------------------------
getpid() = 8460
getpgid() = 8460
getsid() = 8460
➜  signal
```

最终创建的进程会话，**pid == pgid == sid**。



## 4、守护进程：一直存活在 ==系统后台== 的进程

### 1. linux 版本

只有在 **注册信号捕捉** 的函数API不一样。

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <signal.h>

void do_sig_capture(int signal, siginfo_t *info, void *arg)
{
  //1. 关闭释放所有占用的内存、文件....
  //2. 退出进程
  exit(0);
}

int main()
{
  // 1. 创建子进程
  pid_t pid = fork();

  // 2. 父进程直接退出执行，子进程继续往下执行创建会话
  switch(pid)
  {
    case -1:// fork error
      return -1;
    case 0:// child process 跳出循环
      break;
    defualt:// main process 直接退出
      _exit(EXIT_SUCCESS);
  }

  // 3. 子进程创建新的会话
  if (-1 == setsid())
    return -1;

  /** 
   * 4. 改变子进程的工作目录
   * 1=> 目录必须存在
   * 2=> 目录不能被卸载
   */
  chdir("/home/daemon");

  // 5. 设置umask, 保证权限【最大】
  umask(0);

  // 6. 关闭不必要的文件fd
#if 0
  // 方法一：关闭所有打开的文件（非必须）
  int maxfd = sysconf(_SC_OPEN_MAX);
  int fd;
  for (fd=0; fd<maxfd; fd++)
    close(fd);
#else
  // 方法二：让0、1、2三个文件描述符，指向一个/dev/null黑洞描述符
  int fd; 
  fd = open("dev/null", O_RDWR);  // 先打开/dev/null文件，得到他的文件fd
  dup2(fd, STDIN_FILENO);         // STDIN_FILENO => /dev/null
  dup2(fd, STDOUT_FILENO);        // STDOUT_FILENO => /dev/null
  dup2(fd, STDERR_FILENO);        // STDERR_FILENO => /dev/null
#endif

  // 7. 注册捕捉信号（非必须），用于接收到信号时退出守护进程
  struct sigaction action;
  action.sa_sigaction = do_sig_capture;
  sigemptyset(&(action.sa_mask));
  action.sa_flags = 0;
  sigaction(SIGUSR2, &action, NULL);

  // 8. 永久运行，如果需要退出，可以注册捕捉信号
  while(1)
  {
    /**
     *  守护进程要做的事情
     */
    //........
  }
}
```

```
➜  signal gcc main.c
➜  signal ./a.out
➜  signal
```

- 可以看到程序运行之后，就结束返回了。

打开另外一个新的终端，使用`ps aux`查看当前系统进程

```
➜  signal ps -ajx | grep a.out
    1  8535  8535  8535 ?           -1 Rs    1000   0:20 ./a.out
 3427  8549  8548  3427 pts/2     8548 S+    1000   0:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn a.out
➜  signal
```

可以看到进程`a.out`处于`Rs`运行模式，表示运行在`后台`。

### 2. mac os 版本

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <signal.h>

void do_sig_capture(int signal, siginfo_t *info, void *arg)
{
  //1. 关闭释放所有占用的内存、文件....
  //2. 退出进程
  exit(0);
}

int main(int argv, char* argr[])
{   
  //1. 创建子进程
  pid_t pid = fork();

  //2. 父进程直接return结束
  if (pid > 0)
  {
    return 0;
  }

  //3. 子进程回调
  if (0 == pid) 
  {
    //3.1 创建会话，接收创建的会话id
    pid_t new_session_id = setsid();

    //3.2 改变子进程的工作目录。1)目录必须存在 2)目录不能被卸载
    chdir("/home/daemon");

    //3.3 设置umask
    umask(0022);

    //3.4 让默认的三个文件描述符，指向一个黑洞描述符
    int fd = open("dev/null", O_RDWR);
    dup2(fd, STDIN_FILENO);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);

    //3.5 注册捕捉SIGKILL信号
    union __sigaction_u clk;
    clk.__sa_sigaction = do_sig_capture;

    struct sigaction action;
    action.__sigaction_u = clk;
    sigemptyset(&(action.sa_mask));
    action.sa_flags = 0;

    sigaction(SIGUSR2, &action, NULL);

    //3.5 在后台永久运行，在注册的信号捕捉函数中完成
    // => 守护进程完成的业务
    // => 结束进程
    while(1)
    {
      /**
       *  守护进程要做的事情
       */
    }
  }

  return 0;
}
```

### 3. 关闭守护进程

```
ps -aux 找到守护进程id
kill -9 pid
```



## 5、unistd.h 库函数, 直接创建 ==守护进程==

```c
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
  // 1. daemon()创建守护进程
  if(daemon(0, 0) == -1)
    exit(EXIT_FAILURE);

  // 2. 守护进程的逻辑
  while(1);
}
```

```
➜  signal ps -ajx | grep a.out
    1  8609  8609  8609 ?           -1 Rs    1000   0:17 ./a.out
 3628  8630  8629  3628 pts/17    8629 S+    1000   0:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn a.out
➜  signal
```