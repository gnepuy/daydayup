[TOC]



## 1、execl() 函数簇

### 1. api 

```c
#include <unistd.h>

int execl(
  const char *path,
  const char *arg, .../* (char  *) NULL */
);
```

### 2. eg

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
  // 1、指定 /bin/ls 可执行文件的路径
	int ret = execl(
    "/bin/ls", 
    "我是占位参数，并没有什么卵用", 
    "-l", 
    NULL
  );

  /**
   * 2、
   * 只有当 execl() 执行 `出错` 时，才会继续往下执行，
   * 否则 execl() 后续的代码，都不会被执行，
   * 直接被其他的可执行文件替代了
   */
  if (0 != ret)
    printf("execl failed: %d\n", ret);
}
```

```
xzh@xzh-VirtualBox:~/share$ ls -l
total 20
-rwxrwxr-x 1 xzh xzh 8344 Nov  1 14:31 a.out
-rw-rw-r-- 1 xzh xzh  245 Nov  1 14:31 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
xzh@xzh-VirtualBox:~/share$
```

```
xzh@xzh-VirtualBox:~/share$ gcc main.c
xzh@xzh-VirtualBox:~/share$ ./a.out
total 20
-rwxrwxr-x 1 xzh xzh 8344 Nov  1 14:32 a.out
-rw-rw-r-- 1 xzh xzh  245 Nov  1 14:31 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
xzh@xzh-VirtualBox:~/share$
```



## 2、==占位== 参数

### 1. `main(int argv, char *argr[])` 中的 `argr[0]` => a.out

- C代码的命令行参数 **argr[0]** , 是被执行的程序的文件名 **a.out**
- 并不是 **第一个参数**

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argv, char* argr[])
{   
  for (int i = 0; i < argv; ++i)
  {
    printf("argv[%d] = %s\n", i, argr[i]);
  }
}
```

```
bogon:linux_sources xiongzenghui$ ./a.out hah1 hah2 hah3 hah4
argv[0] = ./a.out
argv[1] = hah1
argv[2] = hah2
argv[3] = hah3
argv[4] = hah4
```

- 也就是说要执行哪一个程序，是由**argr[0]**来指定的
- 对于C代码的**main()**的参数是从**argr[1]**开始的

### 2. ==错误==: 将命令行选项放在变参列表 ==第一个== 参数位置

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argv, char* argr[])
{   
  execl("/bin/ls", "-l", NULL);
}
```

```
->  gcc execl.c
->  ./a.out
aaa.txt  args.c   execl.c  main.c  Makefile  mktemp.c	      readv.c	  txt
a.out	 error.c  large.c  main.s  mem.c     process_envir.c  truncate.c  writev.c
->
```

- 1、输出的结果只是 **ls** 执行结果，而并不是 **ls -l** 的结果
- 2、因为 execl() 函数簇中的 **变参列表中的第一个参数** 实际上等价于C代码中的 **argr[0]**
- 3、**argr[0]** 在 main() 中则代表 **可执行程序的名字**，并不是 **第一个命令行参数**

### 3. 正确: execl() 变参列表中 ==第一个== 参数, 作为 ==无用占用参数==

```c
#include <stdio.h>
#include <stdlib.h>

extern char** environ;

int main()
{
	// myvar=xiong, 1表示覆盖已有的环境变量值
	setenv("myvar", "xiong", 1);

  char** p = environ;
  for (; *p != 0; p++)
    printf("%s\n", *p);
}

#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argv, char* argr[])
{
	execl(
    "/bin/ls",                      // 1、要执行的可执行文件的路径
    "我只是用来占位的，什么作用都没有",  // 2、仅仅只是为了占住 argr[0] 的位置
    "-l",                           // 3、传递给可执行文件内 main() 的参数 argr[1] 往后递增
    NULL                            // 4、变参列表结束符
  );
}
```

```
->  gcc execl.c
->  ./a.out
total 68
-rw-r--r-- 1 xzh xzh   25 1月  22 17:23 aaa.txt
-rwxrwxr-x 1 xzh xzh 7344 1月  23 12:52 a.out
-rw-rw-r-- 1 xzh xzh  134 1月  23 11:40 args.c
-rw-rw-r-- 1 xzh xzh  423 1月  22 16:36 error.c
-rw-rw-r-- 1 xzh xzh  162 1月  23 12:51 execl.c
-rw-rw-r-- 1 xzh xzh  249 1月  23 10:17 large.c
-rw-rw-r-- 1 xzh xzh  106 1月  23 10:53 main.c
-rw-rw-r-- 1 xzh xzh  370 1月  22 16:02 main.s
-rw-rw-r-- 1 xzh xzh  130 1月  22 15:24 Makefile
-rw-rw-r-- 1 xzh xzh  561 1月  22 21:51 mem.c
-rw-rw-r-- 1 xzh xzh  216 1月  23 10:26 mktemp.c
-rw-rw-r-- 1 xzh xzh  388 1月  23 12:03 process_envir.c
-rw-rw-r-- 1 xzh xzh  958 1月  23 09:25 readv.c
-rw-rw-r-- 1 xzh xzh  190 1月  23 09:50 truncate.c
-rw-r--r-- 1 xzh xzh    5 1月  23 09:52 txt
-rw-rw-r-- 1 xzh xzh 1133 1月  23 09:25 writev.c
->

```

- 这回就正确执行的是**ls -l**
- 所以对于execl()簇函数都有这个问题



## 3、execl() 后续代码 ==不再执行==

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argv, char* argr[])
{   
  // 1、
  execl("/bin/ls", ".....", "-l", NULL);

  /**
    重点: 后续的代码，都不会被执行
   */

  // 2、
  printf("--------------------------\n");
}
```

```
xzh@xzh-VirtualBox:~/share$ make
total 24
-rw-rw-r-- 1 xzh xzh   30 Nov  1 14:33 Makefile
-rwxrwxr-x 1 xzh xzh 8344 Nov  1 14:54 a.out
-rw-rw-r-- 1 xzh xzh  205 Nov  1 14:54 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
xzh@xzh-VirtualBox:~/share$
```

并没有打印 `2、` 横线字符内容。



## 4、execl() 启动 ==可执行文件==

### 1. exec_A.c

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argv, char* argr[])
{   
  printf("%s\n", "程序A start");
  printf("%s\n", "程序A end");
  return 99;
}
```

### 2. exec_B.c

程序B，替换物理内存的text段为程序A的text段

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argv, char* argr[])
{   
  printf("%s\n", "程序B start");

  // 替换程序A
  execl("./exec_A", "占位参数", "参数1", "参数2", "参数3", NULL);

  printf("%s\n", "程序B end");
  return 100;
}
```

### 3. 编译两个C文件

然后对两个c程序代码进行编译生成可执行程序

```
-> gcc -o exec_A exec_A.c 
-> gcc -o exec_B exec_B.c
```

### 4. 执行 exec_B

```
-> ./exec_B
程序B start
程序A start
程序A end
```

打印最后程序执行的返回值

```
-> echo $?
99
```

可以看到函数返回值是`99`。



## 5、execlp() 从环境变量中, 查找可执行文件

### 1. 直接指定执行的【可执行文件名】

```c
int execlp(
  const char *file, // 已经添加到 PATH 环境变量搜索路径中的【可执行文件名】
  const char *arg, .../* (char  *) NULL */
);
```

### 2. eg 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
  // 根据环境变量PATH记录的bin搜索路径中查找ls
	int ret = execlp(
    "ls", 
    "....", 
    "-l", 
    NULL
  );
  
	if (0 == ret)
		printf("execute success\n");
	else
		printf("execute faled\n");

	return 0;
}
```

```
xzh@xzh-VirtualBox:~/share$ ls -l
total 24
-rw-rw-r-- 1 xzh xzh   30 Nov  1 14:33 Makefile
-rwxrwxr-x 1 xzh xzh 8344 Nov  1 14:33 a.out
-rw-rw-r-- 1 xzh xzh  241 Nov  1 14:32 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
xzh@xzh-VirtualBox:~/share$
```

```
xzh@xzh-VirtualBox:~/share$ make
total 24
-rw-rw-r-- 1 xzh xzh   30 Nov  1 14:33 Makefile
-rwxrwxr-x 1 xzh xzh 8344 Nov  1 14:33 a.out
-rw-rw-r-- 1 xzh xzh  241 Nov  1 14:32 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
xzh@xzh-VirtualBox:~/share$
```

### 3. 读取当前输入的字符，作为执行的程序名

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/wait.h>
#include <unistd.h>

int main(int argv, char* argr[])
{
  // 1、
  char buff[BUFSIZ];
  pid_t pid;
  int status;

  // 2、
  printf("%% ");

  // 3、循环等待输入字符以 \n 结束，作为execlp()要执行的程序名
  while(fgets(buff, BUFSIZ, stdin) != NULL)
  {
    // 字符串末尾手动补充结束符\0
    if (buff[strlen(buff) - 1] == '\n')
      buff[strlen(buff) - 1] = 0;

    // fork创建子进程
    pid  = fork();

    // 区分父子进程回调
    if (pid < 0)
    {
      perror("fork error");
    }
    else if (pid == 0)
    {
     /**
      * 在子进程上，启动其他的程序
      */
      execlp(buff, "无效占位参数", (char*)0);

      // 如果往下执行，则说明出错
      perror("execlp出错:");
      exit(127);
    }
    else
    {
      /**
       * 主进程，等待子进程执行完毕，然后回收掉
       */
      pid = waitpid(pid, &status, 0);
      if (pid < 0)
        perror("waitpid error:");
    }  
  }

  return 0;
}
```

```
->  gcc main.c
->  ./a.out
% ls
 .jpeg				boost_1_66_0			main.cpp
$RECYCLE.BIN			ccodes				myblogs
CF-1151.16			cjiajia				nodejs
Cpp_Concurrency_In_Action	demos				protobuf
Makefile			glibc				rails_projects
PDF				gnustep-base-1.24.9		ruby.rb
SnippetsLabs			groovy				sh
Thumbs.db			iOS				sources
_MG_8467.jpg			jenkin_project			视频教程
a.out				main.c				最低习题练习.pdf
^C
->
```

注意执行的程序必须在 **PATH** 环境变量中配置路径的可执行程序。



## 6、execle() ==传递== 环境变量

### 1.  api

```c
int execle(
  const char *path, 
  const char *arg, .../*, (char *) NULL, char * const envp[] */
);
```

### 2. 进程1

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
  int ret;

  // 1、要传递的环境变量
  char * const envp[] = {"AA=11", "BB=22", NULL};

  // 2、启动进程自定义程序，并传递环境变量
  printf("Entering main ...\n");
  ret = execle("./hello", "hello", NULL, envp);

  // 3、
  if(ret == -1)
    perror("execl error");
  printf("Exiting main ...\n");
  return 0;
}
```

### 3. 进程2

```c
#include <unistd.h>
#include <stdio.h>

extern char** environ;

int main(void)
{
  printf("hello pid=%d\n", getpid());

  // 读取环境变量数组
  int i;
  for (i=0; environ[i]!=NULL; ++i)
  {
    printf("%s\n", environ[i]);
  }
  return 0;
}
```

### 4. 执行结果

```
xzh@xzh-VirtualBox:~/share$ gcc main.c
xzh@xzh-VirtualBox:~/share$ gcc other.c -o other
xzh@xzh-VirtualBox:~/share$ ./a.out
AA=11
BB=22
xzh@xzh-VirtualBox:~/share$
```



## 7、结合 ==文件重定向==

### 1. dup2(fd1, fd2) 

```
linux_io_dup.md
```

### 2. 获取 `ls -l` 的结果

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main(void)
{
  // 1、打开一个磁盘文件
	int fd = open("./abc.txt", O_CREAT | O_RDWR | O_EXCL, 0766);
	
  // 2、stdout 指向 `自己打开的磁盘文件`
	dup2(fd, STDOUT_FILENO);
	
  // 3、执行 `ls -l`
	execl("/bin/ls", "ls", "-l", NULL);
}
```

```
xzh@xzh-VirtualBox:~/share$ ls -l
total 40
-rw-rw-r-- 1 xzh xzh   30 Nov  1 14:33 Makefile
-rwxrwxr-x 1 xzh xzh 8384 Nov  1 15:28 a.out
-rw-rw-r-- 1 xzh xzh  282 Nov  1 15:28 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
-rwxrwxr-x 1 xzh xzh 8368 Nov  1 15:16 other
-rw-rw-r-- 1 xzh xzh  232 Nov  1 15:16 other.c
xzh@xzh-VirtualBox:~/share$
```

```
xzh@xzh-VirtualBox:~/share$ gcc main.c
xzh@xzh-VirtualBox:~/share$ ./a.out
xzh@xzh-VirtualBox:~/share$
```

```
xzh@xzh-VirtualBox:~/share$ ls -l
total 44
-rw-rw-r-- 1 xzh xzh   30 Nov  1 14:33 Makefile
-rwxrwxr-x 1 xzh xzh 8384 Nov  1 15:30 a.out
-rwxrw-r-- 1 xzh xzh  334 Nov  1 15:30 abc.txt
-rw-rw-r-- 1 xzh xzh  282 Nov  1 15:28 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
-rwxrwxr-x 1 xzh xzh 8368 Nov  1 15:16 other
-rw-rw-r-- 1 xzh xzh  232 Nov  1 15:16 other.c
xzh@xzh-VirtualBox:~/share$
```

```
xzh@xzh-VirtualBox:~/share$ cat abc.txt
total 40
-rw-rw-r-- 1 xzh xzh   30 Nov  1 14:33 Makefile
-rwxrwxr-x 1 xzh xzh 8384 Nov  1 15:30 a.out
-rwxrw-r-- 1 xzh xzh    0 Nov  1 15:30 abc.txt
-rw-rw-r-- 1 xzh xzh  282 Nov  1 15:28 main.c
-rw-rw-r-- 1 xzh xzh  194 Nov  1 12:26 main2.c
-rwxrwxr-x 1 xzh xzh 8368 Nov  1 15:16 other
-rw-rw-r-- 1 xzh xzh  232 Nov  1 15:16 other.c
xzh@xzh-VirtualBox:~/share$
```



## 8、fork() + execve() + wait()

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char * argv[])
{
  int pid;

  // 1.【核心】创建一个新的进程
  pid = fork();

  // 2. 判断 fork() 函数的返回值
  if (pid < 0)
  { // 2.1 fork() 执行失败
    fprintf(stderr,"Fork Failed!");
    exit(-1);
  }
  else if (pid == 0)
  { // 2.2 fork() == 0 ， 当前处于【子进程】
    /**
     * 【核心】在【子进程】中加载指定的【可执行文件】
     */
    execlp("/bin/ls","ls",NULL);
  } 
  else
  { // 2.3 fork() > 0 ， 当前处于【父进程】，获取【子进程 pid】

    	// parent will wait for the child to complete
      wait(NULL);
      printf("Child Complete!");
      exit(0);
  }
}
```