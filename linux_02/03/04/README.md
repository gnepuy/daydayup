[TOC]



## 1、wait() ==同步阻塞==, 每次回收 ==一个== 子进程

### 1. wait() 函数原型

#### 1. wait() 返回值

- **返回值 > 0** => 回收成功，返回被回收的子进程id
- **返回值 == -1** => 如果调用进程**没有子进程**进行回收，同时**errno**被置为**ECHILD**

#### 2. status 传出参数，判断进程结束状态

- status 参数最主要的两个8bit位：
- 高8位 => 记录进程调用exit退出的状态（正常退出）
  - 如果**正常退出**(exit) ---高8位是退出状态号，低8位是0
  - 如果**非正常退出**(signal)----高八位是0，低8位是siganl id
- 低8位 => 记录进程接受到的信号 （非正常退出）
  - **WIFEXITED(status)**  这时可以用**WEXITSTATUS(status)** 得到**退出码**（exit的参数）
  - **WIFSIGNALED(status)** 如果**异常退出** （子进程接受到退出信号） 返回**非零值**；使用**WTERMSIG (status)** 得到使子进程退出得**信号编号**
  - **WIFSTOPPED(status)** 如果是**暂停进程**返回的状态，返回**非零值**；使用**WSTOPSIG(status)** 得到使子进程暂停得**信号编号**
  - **WIFCONTINUED(status)** 则是**恢复进程**返回的状态，用法同**WIFSTOPPED(status)** 

| 宏定义               | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| WIFEXITED(status)    | 如果子进程**正常退出**（exit）返回**真（非零值）**；否则返回假。 |
| WEXITSTATUS(status)  | 如果**WIFEXITED**(status)为**真（非零值）**，则可以用该宏取得**子进程exit()的返回值**。 |
| WIFSIGNALED(status)  | 如果子进程因为一个**未捕获的信号而终止**，它就返回**真（非零值）**；否则返回假。 |
| WTERMSIG(status)     | 如果**WIFSIGNALED**(status)为**真（非零值）**，则可以用该宏获得导致子进程终止的**信号编号**。 |
| WIFSTOPPED(status)   | 如果当前子进程被**暂停**了，则返回**真（非零值）**；否则返回假。 |
| WSTOPSIG(status)     | 如果**WIFSTOPPED**(status)为**真（非零值）**，则可以使用该宏获得导致子进程暂停的**信号编号**。 |
| WIFCONTINUED(status) | 如果当前子进程被**恢复执行**了，则返回**真（非零值）**；否则返回假。 |

主要的三个入口宏

| Macro                   | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| WIF**EXITED**(status)   | 如果子进程**正常退出**（exit）返回**真（非零值）**；否则返回假。 |
| WIF**SIG**NALED(status) | 如果子进程因为一个**未捕获的信号而终止**，它就返回**真（非零值）**；否则返回假。 |
| WIF**STOP**PED(status)  | 如果当前子进程被**暂停**了，则返回**真（非零值）**；否则返回假。 |

#### 3. wait() 作用

- 阻塞等待子进程退出
- 清理子进程残留在内核的 pcb 资源
- 通过传出参数，得到子进程结束状态

### 2. wait() 同步阻塞 回收一个子进程 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argv, char* argr[])
{
  //1.
  pid_t pid = -1;
  int wid = -1;
  int status = -1;

  //2.
  pid = fork();

  //3.
  if (pid == 0)
  {// 子进程
    for(;;);// 子进程处于死循环，一直不会执行结束
    return 99;
  }
  else if (pid > 0)
  {// 主进程
    
    // 等待子进程执行完毕，将其回收
    wid = wait(&status);
    printf("wid = %d, status = %d\n", wid, status);

    // 判断是否回收了子进程
    if (wid > 0)
      fprintf(stdout, "【子进程回收成功】子进程id = %d\n", wid);
    else
      fprintf(stdout, "没有子进程被回收\n");

    return 0;
  }
  else
  {
    perror("fork error: ");
  }
}
```

![](../../../../../collect_record/xzhblogs/ipc_04.gif)

- 子进程执行**for(;;);**死循环，一直不会退出
- 而父进程调用**wait()**一直同步阻塞等待子进程执行完毕，然后对其进行回收
- 所以导致父进程一直处于阻塞状态



## 2、wait(&status) 回收子进程, 并获取 ==子进程 返回值==

### 1. 子进程执行 ==exit(返回值)== 成功退出 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <signal.h>

int main(int argv, char* argr[])
{
  //1.
  pid_t pid = -1;
  int wid = -1;
  int status = -1;
  int ret = -1;

  //2.
  pid = fork();

  //3.
  if (pid == 0) 
  {// 子进程
    exit(99); //注意：exit值不要写超过100
  }
  else if (pid > 0)
  {//主进程
    
    // 等待子进程执行完毕，将其回收
    wid = wait(&status);
    printf("wid = %d, status = %d\n", wid, status);

    // status的低16位的高8位的值 => 子进程exit()退出值
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), status >> 8 );
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), (status & 0xff00) >> 8 );

    // 获取status的低4位的值 => 信号id
    fprintf(stdout, "parent [%d] status低8位 =  %d\n", getpid(), status & 0xf);

    // 使用提供的宏定义获取exit()返回值
    if (WIFEXITED(status)) 
    { // 子进程正常退出
      fprintf(stdout, "parent [%d] WEXITSTATUS(status) = %d\n", getpid(), WEXITSTATUS(status));
    }
    else 
    {// 子进程非正常退出
      fprintf(stderr, "子进程非正常退出");
    }

    // 结束主进程
    return 0;
  }
  else
  {
    perror("fork error: ");
  }

  return 0;
}
```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
wid = 13354, status = 25344
parent [13353] status高8位 =  99
parent [13353] status高8位 =  99
parent [13353] status低8位 =  0
parent [13353] WEXITSTATUS(status) = 99
➜  ipc
```

### 2. 子进程因被 ==其他信号== 退出 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <signal.h>

int main(int argv, char* argr[])
{
  //1.
  pid_t pid = -1;
  int wid = -1;
  int status = -1;
  int ret = -1;

  //2.
  pid = fork();

  //3.
  if (pid == 0) 
  {// 子进程
    /** 
     * 触发 SIGSEGV 信号，导致子进程异常退出
     */
    char* p = "hello";
    p[0] = 'H';
  }
  else if (pid > 0)
  {//主进程
    
    // 等待子进程执行完毕，将其回收
    wid = wait(&status);
    printf("wid = %d, status = %d\n", wid, status);

    // status的低16位的高8位的值 => 子进程exit()退出值
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), status >> 8 );
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), (status & 0xff00) >> 8 );

    // 获取status的低8位的值 => 信号id
    fprintf(stdout, "parent [%d] status低8位 =  %d\n", getpid(), status & 0xff);

    // 获取status的低4位的值 => 信号id
    fprintf(stdout, "parent [%d] status低8位 =  %d\n", getpid(), status & 0xf);

    // 使用提供的宏定义获取因为哪个信号而停止
    if (WIFSIGNALED(status))
    {
      fprintf(stdout, "parent [%d] WTERMSIG(status) =  %d\n", getpid(), WTERMSIG(status));   
    }
    else 
    {
      // 其他的信号：信号暂停、信号恢复
    }

    // 结束主进程
    return 0;
  }
  else
  {
    perror("fork error: ");
  }

  return 0;
}
```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
wid = 13421, status = 139
parent [13420] status高8位 =  0
parent [13420] status高8位 =  0
parent [13420] status低8位 =  139
parent [13420] status低8位 =  11
parent [13420] WTERMSIG(status) =  11
➜  ipc
```

### 3. 综合判断 status 所有情况 

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
#include <signal.h>

int main(int argv, char* argr[])
{
  //1.
  pid_t pid = -1;
  int wid = -1;
  int status = -1;
  int ret = -1;

  //2.
  pid = fork();

  //3.
  if (pid == 0) 
  {// 子进程
#if defined(bad)
   /** 
    * 测试触发 SIGSEGV 信号，导致子进程异常退出
    */
    char* p = "hello";
    p[0] = 'H';
#elif defined(success)
  // 测试正常退出
  exit(99);
#else
  // 测试信号暂停、信号恢复
  pause();
#endif
  }
  else if (pid > 0)
  {//主进程
    printf("fork()子进程id = %d\n", pid);
    
    // 等待子进程执行完毕，将其回收
    wid = wait(&status);
    printf("wid = %d, status = %d\n", wid, status);

    // status的低16位的值
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), status >> 8 );
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), (status & 0xff00) >> 8 );
    fprintf(stdout, "parent [%d] status低8位 =  %d\n", getpid(), status & 0xff);
    fprintf(stdout, "parent [%d] status低4位 =  %d\n", getpid(), status & 0xf);

    // 综合子进程的退出情况：
    // => 1. 子进程执行 exit()
    // => 2. 未捕捉的信号而退出
    // => 3. 暂停信号
    // => 4. 恢复信号
    // => 5. 不可能的状态
    if (WIFEXITED(status)) 
    { // 1. 子进程【正常】退出 
      fprintf(stdout, "子进程【正常】退出 => parent [%d] WEXITSTATUS(status) = %d\n", getpid(), WEXITSTATUS(status));
    } 
    else if (WIFSIGNALED(status)) 
    { // 2. 子进程接收到【未捕捉信号】而导致【非正常】退出 
      fprintf(stderr, "子进程【非正常】退出 => 未捕捉信号导致子进程退出 = %d(%s)%s\n", WTERMSIG(status), strsignal(WTERMSIG(status)),  
#ifdef  WCOREDUMP  
              WCOREDUMP(status) ? " (core file generated)" : "");  
#else  
              "");  
#endif 
    } 
    else if (WIFSTOPPED(status)) 
    { // 3. 子进程被【暂停】
      printf("子进程【暂停】执行 => %d(%s)\n", WSTOPSIG(status), strsignal(WSTOPSIG(status)));
    }  
#ifdef WIFCONTINUED  
    else if (WIFCONTINUED(status)) 
    { // 4. 子进程被【恢复】
      printf("子进程【恢复】执行\n");  
    }  
#endif  
    else 
    { //5. 不可能被执行的情况      
      /* Should never happen */  
      printf("what happened to this child? (status=%x)\n", (unsigned int) status);  
    }  
  }
  else
  {
    perror("fork error: ");
  }

  return 0;
}
```

子进程执行exit()退出

```
➜  ipc gcc waitpid.c -Dsuccess
➜  ipc ./a.out
fork()子进程id = 14300
wid = 14300, status = 25344
parent [14299] status高8位 =  99
parent [14299] status高8位 =  99
parent [14299] status低8位 =  0
parent [14299] status低4位 =  0
子进程【正常】退出 => parent [14299] WEXITSTATUS(status) = 99
➜  ipc
```

子进程因为被未捕捉的信号退出

```
➜  ipc gcc waitpid.c -Dbad
➜  ipc ./a.out
fork()子进程id = 14319
wid = 14319, status = 139
parent [14318] status高8位 =  0
parent [14318] status高8位 =  0
parent [14318] status低8位 =  139
parent [14318] status低4位 =  11
子进程【非正常】退出 => 未捕捉信号导致子进程退出 = 11(Segmentation fault) (core file generated)
➜  ipc

```

子进程被暂停

```

```

子进程被恢复

```

```



## 3、wait() 如果当前进程, ==没有子进程== 回收，则返回值为 -1

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argv, char* argr[])
{
  //1.
  pid_t pid = -1;
  int wid = -1;
  int status = -1;

  // 2. 等待子进程执行完毕，将其回收
  wid = wait(&status);
  printf("wid = %d, status = %d\n", wid, status);

  // 3. 判断是否回收了子进程
  if (wid > 0)
    fprintf(stdout, "【子进程回收成功】子进程id = %d\n", wid);
  else
    fprintf(stdout, "没有子进程被回收\n");
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
wid = -1, status = -1
没有子进程被回收
➜  ipc

```



## 4、wait() 一次, 只能回收 一个

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i, status;
  pid_t pid, wait_pid;

  // 1. 主进程，循环创建子进程
  for (i=0; i<5; i++)
  {
    pid = fork(); // 创建子进程
    switch(pid)
    {
      case 0: { // 子进程回调
        /** 
         * 跳出for(;;)指向子进程的代码
         */
        goto child;
      }
      break;
      case -1:
        perror("fork() error:");
        break;
      default: { // 主进程回调直接跳出for()
        continue;
      }
        break; // 主进程继续执行for(;;)创建子进程
    }
  }

  // 2. 主进程，循环【异步】回收所有的子进程
  if (5 == i)
  {
    printf("---------【主进程】[%d]开始同步步回收子进程 ------------\n", i);
        
    // 循环回收子进程
    for (; (wait_pid = wait(-1, &status, 0)) > 0 ;)
    {
      if (WIFEXITED(status)) 
      { // 子进程【正常】退出 
        printf("【parent】回收子进程 pid = %d, wait_pid = %d, WEXITSTATUS(status) = %d\n", pid, wait_pid, WEXITSTATUS(status));      
      } 
    }

    // 已经没有子进程进行回收了
    printf("---------【主进程】[%d]结束同步回收子进程 ------------\n", i);

    // 结束主进程，不往下指向
    exit(0);
  }

  // 3. 子进程回调做的事情
child:
  printf("【child】pid = %d\n", getpid());
  // return 99; // 直接结束子进程
  // exit(99);
  _exit(99);
}

```

![](../../../../../collect_record/xzhblogs/ipc_05.gif)

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
---------【主进程】[5]开始同步步回收子进程 ------------
【child】pid = 4598
【child】pid = 4597
【parent】回收子进程 pid = 4598, wait_pid = 4598, WEXITSTATUS(status) = 99
【parent】回收子进程 pid = 4598, wait_pid = 4597, WEXITSTATUS(status) = 99
【child】pid = 4596
【parent】回收子进程 pid = 4598, wait_pid = 4596, WEXITSTATUS(status) = 99
【child】pid = 4595
【parent】回收子进程 pid = 4598, wait_pid = 4595, WEXITSTATUS(status) = 99
【child】pid = 4594
【parent】回收子进程 pid = 4598, wait_pid = 4594, WEXITSTATUS(status) = 99
---------【主进程】[5]结束同步回收子进程 ------------
➜  ipc
```



## 5、waitpid(pid, status, options)

### 1. waitpid() 

#### 1. 函数原型

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t 
waitpid(
  pid_t pid, 
  int *status, 
  int options
);
```

#### 2. **pid** 参数的选项，指定回收进程的范围

| 参数值     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| pid<-1     | 回收**指定进程组**内的所有子进程。比如：  **kill -9 -39878** 杀死组内全部进程。 |
| pid=0      | 回收**调用waitpid()**所在的**进程组内**的 **全部子进程**     |
| pid=**-1** | 回收【当前进程】的任意的【子进程】。此时的**waitpid()**函数就**退化**成了普通的**wait()**函数。 |
| pid>0      | 回收**指定pid进程号**的子进程。                              |

#### 3. status 传出参数，判断进程结束状态

同 wait() 

#### 4. **option** 参数的选项，指定回收子进程的模式

| 参数        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| 0           | 默认**同步阻塞**回收子进程，效果与**wait()**相同。           |
| **WNOHANG** | 如果pid指定的子进程没有结束，则waitpid()函数立即返回**0**，而**不是阻塞**在这个函数上等待；如果结束了，则返回该子进程的**进程号**。 |
| WUNTRACED   | 如果子进程进入**暂停**状态，则马上返回。                     |
| WCONTINUED  | pid指定的任一子进程在暂停后已经继续，但其状态未被报告，则返回其状态 |

#### 5. 返回值

- **返回值 > 0** => 回收成功，返回被回收的子进程id
- **返回值 == -1** => 如果调用进程**没有子进程**进行回收，同时**errno**被置为**ECHILD**
- 返回值 == 0 => 因设置了**WNOHANG**异步回收，此时没有子进程回收时返回0

### 2. waitpid() 中的 status 传出参数 判断 

```c
#include <stdio.h>
#include <string.h>
#include <sys/wait.h>  
#include <stdlib.h>  
#include <unistd.h>  

/* 
* wait 的 status参数 
* 高8位 记录进程调用exit退出的状态（正常退出） 
* 低8位 记录进程接受到的信号 （非正常退出） 
*/  
int main(int argc, char **argv) {  
  pid_t pid;  
  int status;  

  pid = fork();
  if (pid < 0) {  
    perror("fork");  
    exit(1);  
  } else if (pid == 0) {  
      fprintf(stdout, "child [%d] start!\n", getpid());  
      fprintf(stdout, "child [%d] ends\n", getpid());
#if defined(bad)
      /** 触发 SIGSEGV 信号 */
      char* p = "hello";
      p[0] = 'H';
#else
      exit(99);
#endif
  } 
  else 
  {  
    fprintf(stdout, "parent [%d] start to wait child [%d]!\n", getpid(), pid);  

    // 回收【当前进程】的任意一个【结束执行】的【子进程】
    waitpid(-1, &status, 0);  

    // status的低16位的值
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), status >> 8 );
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), (status & 0xff00) >> 8 );
    fprintf(stdout, "parent [%d] status低8位 =  %d\n", getpid(), status & 0xf);

    // 等待子进程执行完毕，将其回收
    waitpid(-1, &status, 0);

    // status的低16位的值
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), status >> 8 );
    fprintf(stdout, "parent [%d] status高8位 =  %d\n", getpid(), (status & 0xff00) >> 8 );
    fprintf(stdout, "parent [%d] status低8位 =  %d\n", getpid(), status & 0xff);
    fprintf(stdout, "parent [%d] status低4位 =  %d\n", getpid(), status & 0xf);

    // 综合子进程的退出情况：
    // => 1. 子进程执行 exit()
    // => 2. 未捕捉的信号而退出
    // => 3. 暂停信号
    // => 4. 恢复信号
    // => 5. 不可能的状态
    if (WIFEXITED(status)) 
    { // 1. 子进程【正常】退出 
      fprintf(stdout, "子进程【正常】退出 => parent [%d] WEXITSTATUS(status) = %d\n", getpid(), WEXITSTATUS(status));
    } 
    else if (WIFSIGNALED(status)) 
    { // 2. 子进程接收到【未捕捉信号】而导致【非正常】退出 
      fprintf(stderr, "子进程【非正常】退出 => 未捕捉信号导致子进程退出 = %d(%s)%s\n", WTERMSIG(status), strsignal(WTERMSIG(status)),  
#ifdef  WCOREDUMP  
              WCOREDUMP(status) ? " (core file generated)" : "");  
#else  
              "");  
#endif 
    } 
    else if (WIFSTOPPED(status)) 
    { // 3. 子进程被【暂停】
      printf("子进程【暂停】执行 => %d(%s)\n", WSTOPSIG(status), strsignal(WSTOPSIG(status)));
    }  
#ifdef WIFCONTINUED  
    else if (WIFCONTINUED(status)) 
    { // 4. 子进程被【恢复】
      printf("子进程【恢复】执行\n");  
    }  
#endif  
    else 
    { //5. 不可能被执行的情况      
      /* Should never happen */  
      printf("what happened to this child? (status=%x)\n", (unsigned int) status);  
    } 
  }  
}

```

### 3. `waipit(*, *, 0)` ==同步== 回收子进程 

#### 1. `waipit(-1, *, 0)` 与 wait() 作用相同，回收 ==任意== 子进程

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i;
  pid_t pid, wait_pid;
  
  // 创建子进程
  pid = fork(); 

  // 区分主进程or子进程回调
  switch(pid)
  {
    case 0: { // 子进程回调
      printf("【子进程】pid = %d\n", getpid());
      // return -1; // 直接结束子进程
      // exit(EXIT_SUCCESS);
      _exit(EXIT_SUCCESS);
    }
    break;
    case -1:
      perror("fork() error:");
      break;
    default: { // 主进程回调直接跳出for()
      /** 
       * 使用【同步回收】，会等待子进程执行完毕对其回收，
       * 所以不用使用sleep()延迟主进程
       */
      // sleep(1);
      
      printf("【主进程】将要回收子进程 pid = %d\n", pid);
      wait_pid = waitpid(-1, NULL, 0); // 回收当前进程的所有任意子进程
      printf("【主进程】回收子进程 wait_pid = %d\n", wait_pid);
    }
      break;
  }
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
【主进程】将要回收子进程 pid = 15378
【子进程】pid = 15378
【主进程】回收子进程 wait_pid = 15378
➜  ipc

```

#### 2. `waipit(pid, *, 0)` wait()扩展，来回收 ==指定 pid== 子进程 

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i;
  pid_t pid, wait_pid;
  
  // 创建子进程
  pid = fork(); 

  // 区分主进程or子进程回调
  switch(pid)
  {
    case 0: { // 子进程回调
      printf("【子进程】pid = %d\n", getpid());
      // return -1; // 直接结束子进程
      // exit(EXIT_SUCCESS);
      _exit(EXIT_SUCCESS);
    }
    break;
    case -1:
      perror("fork() error:");
      break;
    default: { // 主进程回调直接跳出for()
      printf("【主进程】将要回收子进程 pid = %d\n", pid);
      wait_pid = waitpid(pid, NULL, 0); // 回收指定pid的子进程
      printf("【主进程】回收子进程 wait_pid = %d\n", wait_pid);
    }
      break;
  }
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
【子进程】pid = 23780
【主进程】将要回收子进程 pid = 23780
【主进程】回收子进程 wait_pid = 23780
➜  ipc

```

#### 3. `waipit(0, *, 0)` wait() 扩展，来回收当前 ==进程组内== 所有子进程 

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i;
  pid_t pid, wait_pid;
  
  // 创建子进程
  pid = fork(); 

  // 区分主进程or子进程回调
  switch(pid)
  {
    case 0: { // 子进程回调
      printf("【子进程】pid = %d\n", getpid());
      // return -1; // 直接结束子进程
      // exit(EXIT_SUCCESS);
      _exit(EXIT_SUCCESS);
    }
    break;
    case -1:
      perror("fork() error:");
      break;
    default: { // 主进程回调直接跳出for()
      printf("【主进程】将要回收子进程 pid = %d\n", pid);
      wait_pid = waitpid(0, NULL, 0); // 回收当前进程组内所有子进程
      printf("【主进程】回收子进程 wait_pid = %d\n", wait_pid);
    }
      break;
  }
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
【子进程】pid = 15108
【主进程】将要回收子进程 pid = 15108
【主进程】回收子进程 wait_pid = 15108
➜  ipc

```

#### 4. `waitpid(-1,*, 0)` 回收 ==多个== 子进程 

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i;
  pid_t pid, wait_pid;

  for (i=0; i<5; i++)
  {
    pid = fork(); // 常见子进程
    switch(pid)
    {
      case 0: { // 子进程回调
        printf("【child】pid = %d\n", getpid());
        // return -1; // 直接结束子进程
        // exit(EXIT_SUCCESS);
        _exit(EXIT_SUCCESS);
      }
      break;
      case -1:
        perror("fork() error:");
        break;
      default: { // 主进程回调直接跳出for()
        /** 
         * 因使用【同步】阻塞方式回收子进程，所以无需使用sleep()等待子进程执行结束
         */
        // wait_pid = waitpid(0, NULL, 0); 作用同下
        wait_pid = waitpid(-1, NULL, 0);
        printf("【parent】回收子进程 pid = %d, wait_pid = %d\n", pid, wait_pid);
      }
        break;
    }
  }
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
【child】pid = 15349
【parent】回收子进程 pid = 15349, wait_pid = 15349
【child】pid = 15350
【parent】回收子进程 pid = 15350, wait_pid = 15350
【child】pid = 15351
【parent】回收子进程 pid = 15351, wait_pid = 15351
【child】pid = 15352
【parent】回收子进程 pid = 15352, wait_pid = 15352
【child】pid = 15353
【parent】回收子进程 pid = 15353, wait_pid = 15353
➜  ipc

```

#### 5. 一边创建子进程，一边同步回收子进程 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc, const char * argv[])
{
  int ret;
  char buff[1024] = {0};
  pid_t pid;

  // 1. 死循环创建n个子进程
  for (;;)
  {
    pid = fork();
    switch(pid)
    {
      case 0: { // 子进程回调
        printf("子进程：%d\n", getpid());
        /**
         *  如果是【子进程】回调，直接跳出for循环
         */
        goto child_process_created;
      }
      case -1: { // fork()函数执行错误
        perror("fork() error：");
      }
        break;
      default: { // 父进程回调
        /**
         *  在【父进程】上回调
         *  => 调用 waitpid() 【同步等待】子进程执行完毕，
         *  => 然后对其回收其占用的pcb
         */

        //【同步】阻塞主进程，等待所有的子进程执行完毕，然后对其回收
        ret = waitpid(-1, NULL, 0);
        printf("waitpid()返回值 = %d\n", ret);
      }
        break;
    }
  }

  /**
   *  2. 跳出上面的 for(;;) 外部的代码
   *  => 能执行到此，说明是【子进程】回调
   *  => 子进程fork()回调时，使用了break跳出上面的for(;;)死循环
   *  => 父进程fork()回调时，仍然卡在for(;;)死循环中，执行waitpid()等待回收子进程
   */
child_process_created:
  if (0 == pid)
  {
    printf("子进程%d正在工作ing，即将finish....\n", getpid());
    sleep(1);
    exit(0); // 结束子进程，触发父进程的waitpid()执行
  }
}

```

![](../../../../../collect_record/xzhblogs/ipc_06.gif)

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
子进程：15561
子进程15561正在工作ing，即将finish....
waitpid()返回值 = 15561
子进程：15562
子进程15562正在工作ing，即将finish....
waitpid()返回值 = 15562
子进程：15563
子进程15563正在工作ing，即将finish....
waitpid()返回值 = 15563
子进程：15564
子进程15564正在工作ing，即将finish....
^C
➜  ipc
```

### 4. `waipit(*, *, WNOHANG)` ==异步== 回收子进程

#### 1. `waipit(-1, *, WNOHANG)` 异步回收 ==任意== 子进程

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i;
  pid_t pid, wait_pid;
  
  // 创建子进程
  pid = fork();

  // 区分主进程or子进程回调
  switch(pid)
  {
    case 0: { // 子进程回调
      printf("【子进程】pid = %d\n", getpid());
      // return -1; // 直接结束子进程
      // exit(EXIT_SUCCESS);
      _exit(EXIT_SUCCESS);
    }
    break;
    case -1:
      perror("fork() error:");
      break;
    default: { // 主进程回调直接跳出for()

      // 等待让子进程执行完毕，否则子进程先执行完毕
      sleep(1); 
      
      // 异步回收子进程
      wait_pid = waitpid(-1, NULL, WNOHANG); 
      printf("【主进程】wait_pid = %d\n", wait_pid);
      
      // 判断是否回收到子进程
      if (wait_pid > 0)
        printf("【主进程】回收子进程 wait_pid = %d\n", wait_pid);
      else 
        printf("【主进程】没有子进程进行回收\n");
    }
      break;
  }
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
【子进程】pid = 15255
【主进程】wait_pid = 15255
【主进程】回收子进程 wait_pid = 15255
➜  ipc

```

- **主进程**中只能通过**sleep(1)**延迟一段时间
- 等待**子进程**执行结束，然后对其回收，此时waitpid()函数就会执行返回
- 否则waitpid()会**异步**执行完毕，无法等待子进程执行结束

#### 2. 当 ==没有== 回收到子进程时，==返回 0==

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i;
  pid_t pid, wait_pid;
  
  // 创建子进程
  pid = fork(); 

  // 区分主进程or子进程回调
  switch(pid)
  {
    case 0: { // 子进程回调
      printf("【子进程】pid = %d\n", getpid());
      // return -1; // 直接结束子进程
      // exit(EXIT_SUCCESS);
      _exit(EXIT_SUCCESS);
    }
    break;
    case -1:
      perror("fork() error:");
      break;
    default: { // 主进程回调直接跳出for()

      // 等待让子进程执行完毕，否则子进程先执行完毕
      // sleep(1); 
      
      // 异步回收子进程
      wait_pid = waitpid(-1, NULL, WNOHANG); 
      printf("【主进程】wait_pid = %d\n", wait_pid);
      
      // 判断是否回收到子进程
      if (wait_pid > 0)
        printf("【主进程】回收子进程 wait_pid = %d\n", wait_pid);
      else 
        printf("【主进程】没有子进程进行回收\n");
    }
      break;
  }
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
【主进程】wait_pid = 0
【主进程】没有子进程进行回收
【子进程】pid = 15209
➜  ipc

```

- **子进程**执行后，立马就执行完毕退出
- **主进程**上执行waitpid()会**异步**执行完毕，无法等待子进程执行结束
- 所以无法回收到子进程，返回值为0

#### 3. `waipit(pid, *, WNOHANG)` 异步回收 ==指定 pid== 子进程 

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
  int i;
  pid_t pid, wait_pid;
  
  // 创建子进程
  pid = fork();

  // 区分主进程or子进程回调
  switch(pid)
  {
    case 0: { // 子进程回调
      printf("【子进程】pid = %d\n", getpid());
      // return -1; // 直接结束子进程
      // exit(EXIT_SUCCESS);
      _exit(EXIT_SUCCESS);
    }
    break;
    case -1:
      perror("fork() error:");
      break;
    default: { // 主进程回调直接跳出for()

      // 等待让子进程执行完毕，否则子进程先执行完毕
      sleep(1); 
      
      // 异步回收子进程
      wait_pid = waitpid(pid, NULL, WNOHANG); 
      printf("【主进程】wait_pid = %d\n", wait_pid);
      
      // 判断是否回收到子进程
      if (wait_pid > 0)
        printf("【主进程】回收子进程 wait_pid = %d\n", wait_pid);
      else 
        printf("【主进程】没有子进程进行回收\n");
    }
      break;
  }
}

```

```
➜  ipc gcc waitpid.c
➜  ipc ./a.out
【子进程】pid = 15395
【主进程】wait_pid = 15395
【主进程】回收子进程 wait_pid = 15395
➜  ipc

```

#### 4. `waitpid(-1, NULL, 0)`异步回收 ==多个== 子进程

```c
linux_signal_SIGCHLD.md
```



## 6、waitpid() 对比 wait()

### 1. 同步 or 异步

#### 1. 都支持同步

- 1、**wait()** 只能 **同步** 阻塞方式回收子进程
- 2、**wapitpid(*, *, 0)** 等价于 wait() 同步回收子进程

#### 2. wapitpid() 支持 异步

- **wapitpid(*, *, WNOHANG)**  则可以指定为 **异步**方式回收子进程

### 2. 指定回收的 ==子进程范围==

#### 1. 都可以回收任意子进程

- wait() 无法指定回收子进程范围，默认就是回收 **任意的子进程**
- **waitpid(-1, *, *)**  回收 **任意的子进程**

#### 2. waitpid() 可以指定回收进程

- **waitpid(pid, *, *)**  回收指定pid对应的子进程
- **waitpid(0, *, *)**  回收**调用waitpid()**所在进程的**进程组**内的所有子进程
- **waitpid(-pid, *, *)**  回收指定的**pid**进程所在的**进程组**内的所有子进程

### 3. wait()/waitpid() 一次调用，都只能回收 ==一个== 子进程

- 1、不管同步、异步、回收指定pid、进程组...
- 2、wait()/waitpid() 一次调用，都只能回收一个子进程

### 4. wait()/waitpid() 都只能回收 ==血缘关系== 的进程



