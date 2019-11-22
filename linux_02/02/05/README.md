[TOC]



## 1、读取 目录下的文件

```
➜  demo1 tree
.
├── abc.txt
├── a.out
├── main.c
└── Makefile

0 directories, 4 files
➜  demo1
```

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#include <dirent.h>

int main()
{
  // 1. 打开目录
  DIR* dir = opendir("./");

  // 2. 遍历读取目录下的文件，（包括目录名）
  struct dirent* dirp = NULL;
  while(NULL != (dirp = readdir(dir)))
    printf("%s\n", dirp->d_name);
}
```

这个代码目前是有 bug 的。



## 2、读取 某个目录下 ==文件个数==

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#include <dirent.h>

static int __recursive_dir_statis_files(char* dir_name)
{
  //1. 保存DIR指向的所有的子路径
  struct dirent* p = NULL;

  //2. 记录当前目录的文件名
  char filename[100];

  //3. 初始目录下总文件数为0，因为不可能为负数，所以使用unsigned int无符号类型
  unsigned int file_nums = 0;

  //4. 打开当前目录
  DIR* dir = opendir(dir_name);

  //5. 递归结束、如果无法识别文件，默认为一个普通文件
  if (NULL == dir)
    return 1;

  //6. 遍历读取目录中存放的所有的记录
  while(NULL != (p = readdir(dir)))
  {
    //6.1 过滤掉 . 和 .. 两个目录
    if (0 == strcmp(p->d_name, ".") || \
        0 == strcmp(p->d_name, ".."))
      continue;

    // 6.2 区分是普通文件还是目录
    if (DT_DIR == p->d_type)
    {
      /**
       * 6.2.1 继续递归子目录
       */

      //6.2.1.1 拷贝 p->d_name 记录的文件名
      memset(filename, 0, sizeof(filename)); //清除缓冲区数据，避免保存上一次数据
      strcpy(filename, p->d_name); //拷贝当前目录名，准备进入目录下递归

      //6.2.1.2 递归遍历子目录
      file_nums += __recursive_dir_statis_files(filename);
    }
    else if (DT_REG == p->d_type) //6.3 递归结束二、普通文件，
    {
      /**
       * 普6.2.2 通文件
       * => 结束递归
       * => 总文件数 + 1
       */
      //printf("file_name = %s\n", p->d_name);
      file_nums++;
    }
  }

  //7. 当前目录遍历技术，关闭当前打开的 dir_name 目录
  if (NULL != dir)
  {
    closedir(dir);
    dir = NULL;
  }

  // 8. 返回统计的总文件数
  return file_nums;
}

int main(int argv, const char * argr[])
{
  char* dir_name = (char*)argr[1];
  printf("total %d\n", __recursive_dir_statis_files(dir_name));
}
```

```
➜  demo1 tree
.
├── abc.txt
├── a.out
├── dir1
│   └── dir2
│       └── a1.txt
├── dir2
│   └── a2.txt
├── dir3
│   └── a3.txt
├── main.c
└── Makefile

4 directories, 7 files
➜  demo1 vim main.c
➜  demo1 gcc main.c
➜  demo1 pwd
/home/xzh/share/linuxio/demo1
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1
total 7
➜  demo1
```



## 3、==递归== 打印 目录下 ==普通文件== 并 ==统计== 普通文件的个数

### 1. ls -R 递归打印效果

```
➜  demo1 ls -R .
.:
abc.txt  a.out  dir1  dir2  dir3  link  main.c  Makefile

./dir1:
dir2

./dir1/dir2:
a1.txt

./dir2:
a2.txt

./dir3:
a3.txt
➜  demo1
```

### 2. 代码实现

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#include <dirent.h>

/**
 *    递归统计传入的文件路径下的所有的普通文件，并且打印所有文件的具体信息
 *    @param filepath 传入的文件路径
 *    @param padding 缩进打印的空格字符数
 */
int wc(char* file)
{
  char filepath[1024] = {0};
  char* p;
  unsigned int filenum = 0;
  unsigned int i;
  struct dirent* ent = NULL;
  DIR* dir = NULL;

  //1. 递归结束条件1
  if (NULL == file)
    return 0;

  //2. 递归结束条件2
  dir = opendir(file);
  if (NULL == dir)
    return 0;

  //3. 遍历当前目录下的所有类型的文件
  while(NULL != (ent = readdir(dir)))
  {
    //3.1 过滤掉 . 和 .. 两个隐藏文件
    if (0 == strcmp(ent->d_name, ".") || \
        0 == strcmp(ent->d_name, ".."))
      continue;

    // 3.2 拼接目录下的子文件的全路径：当前工作目录/子文件名
    memset(filepath, 0, sizeof(filepath));
    memcpy(filepath, file, strlen(file));
    
    if (filepath[strlen(filepath) - 1] == '/') // 替换掉末尾的`/`符号
      filepath[strlen(filepath) - 1] = '\0';
    
    p = filepath + strlen(filepath); // 指针变量p指向当前filepath[]数组内容的末尾
    sprintf(p, "/%s", ent->d_name); // 在p指向的filepath[]数组内容的末尾，追加 "/文件名"
    /** 注意：filepath文件路径在linux下，最长只能为256字节 */

    // 3.3 判断当前文件的类型
    switch(ent->d_type)
    {
      case DT_DIR: // 目录
        filenum += wc(filepath); // 继续递归子目录
        break;
      case DT_REG: // 普通文件
      default: // 无法识别的文件
        ++filenum; // 累加1文件数
        printf("%s\n", filepath);
        break;
    }
  }

  // 4.
  closedir(dir);
  dir = NULL;
  return filenum;
}

int main(int argv, char* argr[])
{
  printf("总文件数 = %d\n", wc(argr[1]));
}
```

```
➜  demo1 tree
.
├── abc.txt
├── a.out
├── dir1
│   └── dir2
│       └── a1.txt
├── dir2
│   └── a2.txt
├── dir3
│   └── a3.txt
├── link -> main.c
├── main.c
└── Makefile

4 directories, 8 files
➜  demo1
➜  demo1 pwd
/home/xzh/share/linuxio/demo1
➜  demo1
➜  demo1 gcc main.c
```

```
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1
/home/xzh/share/linuxio/demo1/dir2/a2.txt
/home/xzh/share/linuxio/demo1/Makefile
/home/xzh/share/linuxio/demo1/main.c
/home/xzh/share/linuxio/demo1/dir1/dir2/a1.txt
/home/xzh/share/linuxio/demo1/a.out
/home/xzh/share/linuxio/demo1/link
/home/xzh/share/linuxio/demo1/abc.txt
/home/xzh/share/linuxio/demo1/dir3/a3.txt
总文件数 = 8
➜  demo1
```

```
➜  demo1 ./a.out .
./dir2/a2.txt
./Makefile
./main.c
./dir1/dir2/a1.txt
./a.out
./link
./abc.txt
./dir3/a3.txt
总文件数 = 8
➜  demo1
```