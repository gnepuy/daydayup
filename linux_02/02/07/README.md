[TOC]



## 1、fcntl() 修改文件的 ==控制属性==

### 1. 修改 文件为 ==非阻塞==


```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argv, char* argr[])
{	
  //1.
  int fd = open("./foo.txt", O_RDWR);

  //2. getter 获取文件的控制属性
  int flags = fcntl(fd, F_GETFL);

  //3. 设置文件的读写方式为【非阻塞】
  flags |= O_NONBLOCK;
  
  //4. setter 再将修改后的文件属性，设置回去
  fcntl(fd, F_SETFL, flags);
}
```

### 2. 还可以在 open() 时，直接指定 **非阻塞**

```c
int fd = open("abc.txt", O_RDWR | O_NONBLOCK);
```



## 2、ioctl() 修改文件的 ==物理属性==

### 1.【设备文件】的物理属性值

- 所有的**硬件设备**连接到系统时
- 都会被抽象为一个**文件**
- 是一种特殊的**设备文件**
- **设备文件**与普通文件是有区别的

### 2. 必须按照硬件厂商设置的参数进行设置

比如，获取 终端显示屏 的大小：

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>

int main(int argv, char* argr[])
{	
	struct winsize size = {0};
	ioctl(STDOUT_FILENO, TIOCGWINSZ, &size);

	printf("屏幕宽度 = %d, 屏幕高度 = %d\n", size.ws_row, size.ws_col);
}
```

```
->     gcc file_11.c
->     ./a.out
屏幕宽度 = 25, 屏幕高度 = 158
```