[TOC]



## 1、==移动== 文件的 ==读写指针==

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>

int main()
{
  int len;
  char buf[1024] = {"hello\nworld\nworinima"};
  int fd = open("abc.txt",  O_CREAT | O_RDWR, 0644);

  // 先写入 => 文件指针已经移动到末尾
  len = write(fd, buf, strlen(buf));

  // 让文件指针先移动到文件开始位置
  lseek(fd, 0, SEEK_SET);

  // 再读取 => 文件指针已经指向文件末尾，所以无法再读取到数据
  memset(buf, 0, 1024);
  len = read(fd, buf, 1024);
  printf("buf = %s", buf);
}
```

```
➜  demo3 ls
main.c
➜  demo3 gcc main.c
➜  demo3 ./a.out
buf = hello
world
➜  demo3
➜  demo3 ls
abc.txt  a.out  main.c
➜  demo3 cat abc.txt
hello
world
➜  demo3
```



## 2、文件 ==扩容== 0x1000 字节

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>

int main()
{
  //1.
  int fd = open("abc.txt",  O_RDWR | O_CREAT, 0777);

  //2. 
  if (fd < 0)
  {
    perror("读取文件出错");	
    return -1;	
  }

  //3. 先移动文件指针到4095个字节的位置
  lseek(fd, 0x1000 - 1, SEEK_SET);

  //4. 再从4095字节的位置，随便写入1个字节数据即可完成扩充4096个字节
  // 注意：必须要对文件至少有一次写入操作，否则扩充容量失败。
  write(fd, "A", 1);
}
```

```c
➜  demo1 make
gcc main.c
./a.out
ls -l
total 24
-rwxrwxr-x 1 xzh xzh 4096 4月   2 15:42 abc.txt
-rwxrwxr-x 1 xzh xzh 8760 4月   2 15:42 a.out
-rw-rw-r-- 1 xzh xzh  453 4月   2 15:42 main.c
-rw-rw-r-- 1 xzh xzh   73 4月   2 15:27 Makefile
➜  demo1
```



## 3、获取 ==文件大小==

> 更好的方式应该使用 lstat() 获取文件的大小. 

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>

int main(int argc, const char * argv[])
{
  // 1.
  int fd = open("abc.txt",  O_RDWR);
  if (fd < 0) perror("读取文件出错");

  // 2.【重要】从0开始，文件指针便宜到最末尾
  int offset = lseek(fd, 0, SEEK_END);

  // 3
  printf("%d\n", offset);
  close(fd);
}
```

```
➜  demo1 make
gcc main.c
./a.out
4096
ls -l
total 24
-rwxrwxr-x 1 xzh xzh 4096 4月   2 15:42 abc.txt
-rwxrwxr-x 1 xzh xzh 8808 4月   2 15:55 a.out
-rw-rw-r-- 1 xzh xzh  446 4月   2 15:55 main.c
-rw-rw-r-- 1 xzh xzh   73 4月   2 15:27 Makefile
➜  demo1
```



## 4、truncate 扩充 文件容量

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>

int main()
{
	int fd;
	system("ls -l | grep file.txt");
	fd = open("./file.txt", O_RDWR | O_CREAT, 0644);
	ftruncate(fd, 100);
	close(fd);
	system("ls -l | grep file.txt");
}
```

```
➜  ipc ls -l | grep file.txt
➜  ipc gcc main.c
➜  ipc ./a.out
-rw-r--r-- 1 xzh xzh  100 4月   7 21:27 file.txt
➜  ipc ls -l | grep file.txt
-rw-r--r-- 1 xzh xzh  100 4月   7 21:27 file.txt
➜  ipc cat file.txt
➜  ipc
```

创建了一个100字节的空文件。