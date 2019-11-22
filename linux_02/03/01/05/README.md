[TOC]



## 1、exit() 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
  printf("%s", " wo ri ni ma");
  exit(1);
}
```

```
$ gcc exit与_exit的区别.c
$ ./a.out
$
```

- 打印的正常输出到屏幕
- 总结 exit()
  - 标准C库提供的函数
  - 先刷新缓冲区
  - 再调用 **内核函数** 完成 **结束进程**



## 2、`_exit()` 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
  printf("%s", " wo ri ni ma");
  _exit(1);
}
```

```
$ gcc exit与_exit的区别.c
$ ./a.out
$
```

- 并 **没有** 看到任何的打印输出
- 总结 `_exit()`
  - linux 内核函数
  - 不会刷新缓冲区
  - 内部调用 其他的内核函数 结束进程

