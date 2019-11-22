[TOC]



## 1、获取 文件的 ==基本属性==

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

int main(int argv, char* argr[])
{
  int i, ret, mask, bit;
  struct stat st = {0}; // 存储文件的所有信息 => 需要在栈上自己分配内存
  struct group* grp;    // gid => 下面都是系统分配内存
  struct passwd *pwd;   // uid
  struct tm* time;      // 文件修改时间

  // 1. 文件路径
  char* filepath = argr[1];
  printf("filepath = %s\n", filepath);

  // 2. lstat() 读取文件的各种属性
  ret = lstat(filepath, &st);
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  // 3. 读取文件的基本信息
  printf("st_dev = %ld\n",         st.st_dev);
  printf("st_ino = %ld\n",         st.st_ino);
  printf("st_mode = %d\n",         st.st_mode);
  printf("st_nlink = %ld\n",       st.st_nlink);
  printf("st_uid = %d\n",          st.st_uid);
  printf("st_gid = %d\n",          st.st_gid);
  printf("st_rdev = %ld\n",        st.st_rdev);
  printf("st_size = %ld\n",        st.st_size);
  printf("st_blksize = %ld\n",     st.st_blksize);
  printf("st_blocks = %ld\n",      st.st_blocks);

  // 4. group
  grp = getgrgid(st.st_gid);
  printf("group_name = %s\n", grp->gr_name);

  // 5. uid
  pwd = getpwuid(st.st_uid);
  printf("pw_name = %s\n", pwd->pw_name);

  // 6. 创建时间
  time = localtime(&st.st_ctime);
  printf("time = %d:%d:%d-%d-%d-%d\n", time->tm_year + 1990, time->tm_mon + 1, time->tm_mday, time->tm_hour, time->tm_min, time->tm_sec);
}
```

```
➜  demo1 ls -l
total 36
-rwxrwxr-x 1 xzh xzh 4096 4月   2 15:42 abc.txt
-rwxrwxr-x 1 xzh xzh 9032 4月   2 16:14 a.out
drwxrwxr-x 3 xzh xzh 4096 4月   2 16:07 dir1
drwxrwxr-x 3 xzh xzh 4096 4月   2 16:08 dir2
drwxrwxr-x 3 xzh xzh 4096 4月   2 16:08 dir3
-rw-rw-r-- 1 xzh xzh 1538 4月   2 16:18 main.c
-rw-rw-r-- 1 xzh xzh   74 4月   2 15:58 Makefile
➜  demo1 gcc main.c
➜  demo1 pwd
/home/xzh/share/linuxio/demo1
➜  demo1
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1/main.c
filepath = /home/xzh/share/linuxio/demo1/main.c
st_dev = 2054
st_ino = 180807
st_mode = 33204
st_nlink = 1
st_uid = 1000
st_gid = 1000
st_rdev = 0
st_size = 1538
st_blksize = 4096
st_blocks = 8
group_name = xzh
pw_name = xzh
time = 2108:4:2-16-18-13
➜  demo1
```



## 2、lstat()/stat() 获取 文件的 ==类型==

### 1. main.c

使用宏定义，区分是有 lstat() 和 stat() 区别

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

int main(int argv, char* argr[])
{
  int i, ret, mask, bit;
  struct stat st = {0}; // 存储文件的所有信息
  struct group* grp; // gid
  struct passwd *pwd; // uid
  struct tm* time; // 文件修改时间

  //1.
  char* filepath = argr[1];
  printf("filepath = %s\n", filepath);

  //2. lstat()或stat() 读取文件的各种属性
#if defined(use_lstat)
  ret = lstat(filepath, &st);
#else
  ret = stat(filepath, &st);
#endif
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
    3. 获取文件的类型:
    ********************************************************
    st_mode 所有的选项值(都是8进制位值)
    S_IFMT     0170000   bit mask for the file type bit field
    S_IFSOCK   0140000   socket
    S_IFLNK    0120000   symbolic link
    S_IFREG    0100000   regular file
    S_IFBLK    0060000   block device
    S_IFDIR    0040000   directory
    S_IFCHR    0020000   character device
    S_IFIFO    0010000   FIFO
    ********************************************************
    S_IFMT => 为获取下面各种文件类型的二进制位mask掩码
    st_mode & S_IFMT => 即可获得mask掩码对应二进制位的值，从而得到文件的类型
   */
  switch (st.st_mode & S_IFMT)
  {
    case S_IFSOCK:
      printf("socket文件\n");
      break;
    case S_IFLNK:
      printf("软链接文件\n");
      break;
    case S_IFREG:
      printf("普通文件\n");
      break;
    case S_IFBLK:
      printf("块设备文件\n");
      break;
    case S_IFDIR:
      printf("目录文件\n");
      break;
    case S_IFCHR:
      printf("字符设备文件\n");
      break;
    case S_IFIFO:
      printf("fifo管道文件\n");
      break;
  }
}
```

### 2. stat() 会 ==穿透==

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
➜  demo1 ls -l
total 36
-rwxrwxr-x 1 xzh xzh 4096 4月   2 15:42 abc.txt
-rwxrwxr-x 1 xzh xzh 8928 4月   2 16:44 a.out
drwxrwxr-x 3 xzh xzh 4096 4月   2 16:07 dir1
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir2
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir3
lrwxrwxrwx 1 xzh xzh    6 4月   2 16:43 link -> main.c
-rw-rw-r-- 1 xzh xzh 1607 4月   2 16:42 main.c
-rw-rw-r-- 1 xzh xzh   74 4月   2 15:58 Makefile
➜  demo1
➜  demo1 gcc main.c
➜  demo1 pwd
/home/xzh/share/linuxio/demo1
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1/link
filepath = /home/xzh/share/linuxio/demo1/link
普通文件
➜  demo1
```

- 1、**stat()** 获取 **软链接 link** 文件的类型时，
- 2、直接返回了 **link 指向** 的 **main.c** 真实文件类型

### 3. lstat() 只会获取 ==当前文件== 的类型

```
➜  demo1 gcc main.c -Duse_lstat
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1/link
filepath = /home/xzh/share/linuxio/demo1/link
软链接文件
➜  demo1
```

**lstat()** 获取的就是文件的类型。



## 3、通过 宏定义 获取 文件类型

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

int main(int argv, char* argr[])
{
  int i, ret, mask, bit;
  struct stat st = {0}; // 存储文件的所有信息
  struct group* grp; // gid
  struct passwd *pwd; // uid
  struct tm* time; // 文件修改时间

  //1.
  char* filepath = argr[1];
  printf("filepath = %s\n", filepath);

  //2. lstat()或stat() 读取文件的各种属性
#if defined(use_lstat)
  ret = lstat(filepath, &st);
#else
  ret = stat(filepath, &st);
#endif
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
    可以通过如下函数来判断是否是对应的类型
    S_ISREG(m)  is it a regular file?
    S_ISDIR(m)  directory?
    S_ISCHR(m)  character device?
    S_ISBLK(m)  block device?
    S_ISFIFO(m) FIFO (named pipe)?
    S_ISLNK(m)  symbolic link?  (Not in POSIX.1-1996.)
    S_ISSOCK(m) socket?  (Not in POSIX.1-1996.)
    */
  if (S_ISREG(st.st_mode))
  {
    printf("普通文件\n");
  }
    else if (S_ISDIR(st.st_mode))
  {
  printf("目录文件\n");
  }
    else if (S_ISCHR(st.st_mode))
  {
    printf("字符设备文件\n");
  }
    else if (S_ISBLK(st.st_mode))
  {
    printf("块设备文件\n");
  }
    else if (S_ISFIFO(st.st_mode))
  {
    printf("fifo管道文件\n");
  }
  else if (S_ISLNK(st.st_mode))
  {
    printf("软链接文件\n");
  }
  else if (S_ISSOCK(st.st_mode))
  {
    printf("socket文件\n");
  }
}
```



## 4、格式化 打印 文件类型

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

int main(int argv, char* argr[])
{
  int i, ret, mask, bit;
  struct stat st = {0}; // 存储文件的所有信息
  struct group* grp; // gid
  struct passwd *pwd; // uid
  struct tm* time; // 文件修改时间

  //1.
  char* filepath = argr[1];
  printf("filepath = %s\n", filepath);

  //2. lstat()或stat() 读取文件的各种属性
#if defined(use_lstat)
  ret = lstat(filepath, &st);
#else
  ret = stat(filepath, &st);
#endif
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
    3. 获取文件的类型:
   */
  switch (st.st_mode & S_IFMT)
  {
    case S_IFSOCK:
      printf("%c", 's');
      break;
    case S_IFLNK:
      printf("%c", 'l');
      break;
    case S_IFREG:
      printf("%c", '-');
      break;
    case S_IFBLK:
      printf("%c", 'b');
      break;
    case S_IFDIR:
      printf("%c", 'd');
      break;
    case S_IFCHR:
      printf("%c", 'c');
      break;
    case S_IFIFO:
      printf("%c", 'p');
      break;
  }
}
```



## 5、打印文件的 ==三组权限值==

### 1. 以 ==二进制== 形式打印

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

int main(int argv, char* argr[])
{
  int i, ret, mask, bit;
  struct stat st = {0}; // 存储文件的所有信息
  struct group* grp; // gid
  struct passwd *pwd; // uid
  struct tm* time; // 文件修改时间

  //1.
  char* filepath = argr[1];
  printf("filepath = %s\n", filepath);

  //2. lstat()或stat() 读取文件的各种属性
#if defined(use_lstat)
  ret = lstat(filepath, &st);
#else
  ret = stat(filepath, &st);
#endif
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
    3. 打印文件的权限值
    ************************************
    st_mode的低9位表示权限值：rwx-rwx-rwx
    ************************************
    需要分别获取每一个二进制位的值
    => 如果是0，表示有权限
    => 如果是1，表示没有权限
    ************************************
    二进制位排列顺序是 user -> group -> other
    二进制位的顺序是【876】【543】【210】
   */
  printf("文件权限值: ");
  for (i = 8; i >=0; i--)
  {
    // 获取每一个二进制位的mask掩码（当前i位1，其他位都是0）
    mask = 1 << i;

    // 获取下当前i位的二进制位的值
    bit = (st.st_mode & mask) >> i;

    // 控制每三位逗号输出
    if (i%3 == 0 && (i != 0))
      printf("%d, ",bit);
    else
      printf("%d",bit);
  }
  printf("\n");
}
```

```
➜  demo1 ls -l
total 36
-rwxrwxr-x 1 xzh xzh 4096 4月   2 15:42 abc.txt
-rwxrwxr-x 1 xzh xzh 8928 4月   2 17:00 a.out
drwxrwxr-x 3 xzh xzh 4096 4月   2 16:07 dir1
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir2
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir3
lrwxrwxrwx 1 xzh xzh    6 4月   2 16:43 link -> main.c
-rw-rw-r-- 1 xzh xzh 1569 4月   2 17:02 main.c
-rw-rw-r-- 1 xzh xzh   74 4月   2 15:58 Makefile
➜  demo1 pwd
/home/xzh/share/linuxio/demo1
➜  demo1 gcc main.c
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1/a.out
filepath = /home/xzh/share/linuxio/demo1/a.out
文件权限值: 111, 111, 101
➜  demo1
```

### 2. 以 ==rwx-rwx-rwx== 形式打印

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

int main(int argv, char* argr[])
{
  int i, ret, mask, bit;
  struct stat st = {0}; // 存储文件的所有信息
  struct group* grp; // gid
  struct passwd *pwd; // uid
  struct tm* time; // 文件修改时间
  char rwx[] = "xwr-";

  //1.
  char* filepath = argr[1];
  printf("filepath = %s\n", filepath);

  //2. lstat()或stat() 读取文件的各种属性
#if defined(use_lstat)
  ret = lstat(filepath, &st);
#else
  ret = stat(filepath, &st);
#endif
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
    3. 打印文件的权限值
   */
  printf("文件权限值: ");
  for (i = 8; i >=0; i--)
  {
    // 获取每一个二进制位的mask掩码（当前i位1，其他位都是0）
    mask = 1 << i;

    // 获取下当前i位的二进制位的值
    bit = (st.st_mode & mask) >> i;

    // rwx输出每一个权限二进制位
    // 从 "xwr-"选出对应的字符显示
    //
    //	rwx-rwx-rwx
    //	876-543-210
    //
    //	8,5,2 => r => 0 => 取余3=2 => rwx[2]
    //  7,4,1 => w => 1 => 取余3=1 => rwx[1]
    //  6,3,0 => x => 2 => 取余3=0 => rwx[0]
    if (1 == bit)
      printf("%c", rwx[ i % 3 ]);
    else
      printf("%c", rwx[3]); // 没有权限值 => rwx[3] => `-`
  }
  printf("\n");
}
```

```
➜  demo1 ls -l
total 36
-rwxrwxr-x 1 xzh xzh 4096 4月   2 15:42 abc.txt
-rwxrwxr-x 1 xzh xzh 8928 4月   2 17:02 a.out
drwxrwxr-x 3 xzh xzh 4096 4月   2 16:07 dir1
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir2
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir3
lrwxrwxrwx 1 xzh xzh    6 4月   2 16:43 link -> main.c
-rw-rw-r-- 1 xzh xzh 1588 4月   2 17:05 main.c
-rw-rw-r-- 1 xzh xzh   74 4月   2 15:58 Makefile
➜  demo1 pwd
/home/xzh/share/linuxio/demo1
➜  demo1 gcc main.c
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1/a.out
filepath = /home/xzh/share/linuxio/demo1/a.out
文件权限值: rwxrwxr-x
➜  demo1
```



## 6、完整解析一个文件的所有属性

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

/**
 * @brief  通过lstat(file)获取文件属性，解析后格式化打印
 *
 * @param  filepath: 要获取文件属性的文件路径
 * @param  is_seperator: 是否在每一项属性之间使用 | 分隔开
 * @retval None
 */
void show_file_info(char* filepath, int is_seperator)
{
  int i, ret, mask, bit;
  struct stat st = {0};
  struct group* grp;
  struct passwd *pwd;
  struct tm* time;
  char rwx[] = "xwr-";

  // 1. lstat() 读取文件的各种属性
  // printf("filepath = %s\n", filepath);
  ret = lstat(filepath, &st);
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
   * 2. 打印文件的类型
   */
  switch (st.st_mode & S_IFMT)
  {
    case S_IFSOCK:
      printf("%c", 's');
      break;
    case S_IFLNK:
      printf("%c", 'l');
      break;
    case S_IFREG:
      printf("%c", '-');
      break;
    case S_IFBLK:
      printf("%c", 'b');
      break;
    case S_IFDIR:
      printf("%c", 'd');
      break;
    case S_IFCHR:
      printf("%c", 'c');
      break;
    case S_IFIFO:
      printf("%c", 'p');
      break;
  }

  /**
   * 3. 打印文件的权限值
   */
  for (i = 8; i >=0; i--)
  {
    mask = 1 << i;
    bit = (st.st_mode & mask) >> i;
    if (1 == bit)
      printf("%c", rwx[ i % 3 ]);
    else
      printf("%c", rwx[3]);
  }
  if (1 == is_seperator)
    printf(" |");

  // 4. 打印文件的链接数
  printf(" %ld", st.st_nlink);
  if (1 == is_seperator)
    printf(" |");

  // 5. 用户id
  pwd = getpwuid(st.st_uid);
  printf(" %s", pwd->pw_name);
  if (1 == is_seperator)
    printf(" |");

  // 6. 组id
  grp = getgrgid(st.st_gid);
  printf(" %s", grp->gr_name);
  if (1 == is_seperator)
    printf(" |");

  // 7. 文件大小
  printf(" %ld", st.st_size);
  if (1 == is_seperator)
    printf(" |");

  // 8. 时间
  time = localtime(&st.st_ctime);
  printf(" %d:%d:%d-%d-%d-%d", time->tm_year + 1990, time->tm_mon + 1, time->tm_mday, time->tm_hour, time->tm_min, time->tm_sec);
  if (1 == is_seperator)
    printf(" |");

  // 9. 文件名
  printf(" %s\n", filepath);
}

int main(int argv, char* argr[])
{
  show_file_info(argr[1], 1);
  show_file_info(argr[1], 0);
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
➜  demo1 gcc main.c
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1/a.out
-rwxrwxr-x | 1 | xzh | xzh | 9128 | 2108:4:2-20-20-42 | /home/xzh/share/linuxio/demo1/a.out
-rwxrwxr-x 1 xzh xzh 9128 2108:4:2-20-20-42 /home/xzh/share/linuxio/demo1/a.out
➜  demo1
```



## 7、实现 `ls -l`

### 1. 非递归只能遍历当前目录

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

/**  
 * @brief  打印文件的完整属性
 * @param  filepath: 文件路径
 */
void show_file_info(char* filepath)
{
  int i, ret, mask, bit;
  struct stat st = {0};
  struct group* grp;
  struct passwd *pwd;
  struct tm* time;
  char rwx[] = "xwr-";

  //1. 打印获取新的文件
  printf("filepath = %s\n", filepath);

  //2. lstat() 读取文件的各种属性
  ret = lstat(filepath, &st);
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
    3. 打印文件的类型
   */
  switch (st.st_mode & S_IFMT)
  {
    case S_IFSOCK:
      printf("%c", 's');
      break;
    case S_IFLNK:
      printf("%c", 'l');
      break;
    case S_IFREG:
      printf("%c", '-');
      break;
    case S_IFBLK:
      printf("%c", 'b');
      break;
    case S_IFDIR:
      printf("%c", 'd');
      break;
    case S_IFCHR:
      printf("%c", 'c');
      break;
    case S_IFIFO:
      printf("%c", 'p');
      break;
  }

  /**
    4. 打印文件的权限值
   */
  for (i = 8; i >=0; i--)
  {
    // 获取每一个二进制位的mask掩码（当前为1，其他位都是0）
    mask = 1 << i;

    // 获取下标为i的二进制位的值
    bit = (st.st_mode & mask) >> i;

    // 从 "xwr-"选出对应的字符显示
    if (1 == bit)
      printf("%c", rwx[ i % 3 ]);
    else
      printf("%c", rwx[3]);
  }

  //5. 打印文件的链接数
  printf(" %ld", st.st_nlink);

  //6. 用户id
  pwd = getpwuid(st.st_uid);
  printf(" %s", pwd->pw_name);

  //7. 组id
  grp = getgrgid(st.st_gid);
  printf(" %s", grp->gr_name);

  //8. 文件大小
  printf(" %ld", st.st_size);

  //9. 时间
  time = localtime(&st.st_ctime);
  printf(" %d:%d:%d-%d-%d-%d", time->tm_year + 1990, time->tm_mon + 1, time->tm_mday, time->tm_hour, time->tm_min, time->tm_sec);

  //10. 文件名
  printf(" %s\n", filepath);
}

int main(int argv, char* argr[])
{
  show_file_info(argr[1]);
}
```

```
➜  demo1 ls -l
total 36
-rwxrwxr-x 1 xzh xzh 4096 4月   2 15:42 abc.txt
-rwxrwxr-x 1 xzh xzh 9128 4月   2 17:25 a.out
drwxrwxr-x 3 xzh xzh 4096 4月   2 16:07 dir1
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir2
drwxrwxr-x 2 xzh xzh 4096 4月   2 16:29 dir3
lrwxrwxrwx 1 xzh xzh    6 4月   2 16:43 link -> main.c
-rw-rw-r-- 1 xzh xzh 2115 4月   2 17:25 main.c
-rw-rw-r-- 1 xzh xzh   74 4月   2 15:58 Makefile
➜  demo1 pwd
/home/xzh/share/linuxio/demo1
➜  demo1 gcc main.c
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1/a.out
filepath = /home/xzh/share/linuxio/demo1/a.out
-rwxrwxr-x 1 xzh xzh 9128 2108:4:2-17-26-22 /home/xzh/share/linuxio/demo1/a.out
➜  demo1
```

### 2. 递归遍历子目录

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <dirent.h>

/** 
 * @brief  通过lstat(file)获取文件属性，解析后格式化打印
 * 
 * @param  filepath: 要获取文件属性的文件路径
 * @param  is_seperator: 是否在每一项属性之间使用 | 分隔开
 * @retval None
 */
void show_file_info(char* filepath, int is_seperator)
{
  int i, ret, mask, bit;
  struct stat st = {0};
  struct group* grp;
  struct passwd *pwd;
  struct tm* time;
  char rwx[] = "xwr-";

  //1. 打印获取新的文件
  //printf("filepath = %s\n", filepath);

  //2. lstat() 读取文件的各种属性
  ret = lstat(filepath, &st);
  if (-1 == ret)
  {
    printf("filepath = %s\n", filepath);
    perror("lstat() error");
    exit(-1);
  }

  /**
    3. 打印文件的类型
   */
  switch (st.st_mode & S_IFMT)
  {
    case S_IFSOCK:
      printf("%c", 's');
      break;
    case S_IFLNK:
      printf("%c", 'l');
      break;
    case S_IFREG:
      printf("%c", '-');
      break;
    case S_IFBLK:
      printf("%c", 'b');
      break;
    case S_IFDIR:
      printf("%c", 'd');
      break;
    case S_IFCHR:
      printf("%c", 'c');
      break;
    case S_IFIFO:
      printf("%c", 'p');
      break;
  }

  /**
    4. 打印文件的权限值
   */
  for (i = 8; i >=0; i--)
  {
    // 获取每一个二进制位的mask掩码（当前为1，其他位都是0）
    mask = 1 << i;

    // 获取下标为i的二进制位的值
    bit = (st.st_mode & mask) >> i;

    // 从 "xwr-"选出对应的字符显示
    if (1 == bit)
      printf("%c", rwx[ i % 3 ]);
    else
      printf("%c", rwx[3]);
  }
  if (1 == is_seperator)
    printf(" |");

  //5. 打印文件的链接数
  printf(" %ld", st.st_nlink);
  if (1 == is_seperator)
    printf(" |");

  //6. 用户id
  pwd = getpwuid(st.st_uid);
  printf(" %s", pwd->pw_name);
  if (1 == is_seperator)
    printf(" |");

  //7. 组id
  grp = getgrgid(st.st_gid);
  printf(" %s", grp->gr_name);
  if (1 == is_seperator)
    printf(" |");

  //8. 文件大小
  printf(" %ld", st.st_size);
  if (1 == is_seperator)
    printf(" |");

  //9. 时间
  time = localtime(&st.st_ctime);
  printf(" %d:%d:%d-%d-%d-%d", time->tm_year + 1990, time->tm_mon + 1, time->tm_mday, time->tm_hour, time->tm_min, time->tm_sec);
  if (1 == is_seperator)
    printf(" |");

  //10. 文件名
  printf(" %s\n", filepath);
}

int _ls(char* file, int padding)
{
  const char PADDING = '>';
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
    p = filepath + strlen(filepath);
    sprintf(p, "/%s", ent->d_name); // 拼接当前子目录的全路径

    // 3.3 判断当前文件的类型
    switch(ent->d_type)
    {
      case DT_DIR: // 目录
        filenum += _ls(filepath, padding+2); // 继续递归子目录
        break;
      case DT_REG: // 普通文件
      default: // 无法识别的文件
      {
        ++filenum; // 累加1文件数
        for (i = 0; i < padding; i++) // 打印当前文件所在目录的缩进字符
          printf("%c", PADDING);
         show_file_info(filepath, 1); // 打印文件的所有属性
      }
        break;
    }
  }

  // 4.
  closedir(dir);
  dir = NULL;
  return filenum;
}

/** 
 * @brief  递归遍历当前目录，打印并统计非目录文件
 * 
 * @param  file: 目录路径
 * @param  padding: 
 * @retval 
 */
int ls(char* file)
{
  return _ls(file, 0);
}

int main(int argv, char* argr[])
{
  printf("总文件数 = %d\n", ls(argr[1]));
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
```

```
➜  demo1 gcc main.c
➜  demo1 ./a.out /home/xzh/share/linuxio/demo1
>>-rw-rw-r-- | 1 | xzh | xzh | 0 | 2108:4:2-16-29-35 | /home/xzh/share/linuxio/demo1/dir2/a2.txt
-rw-rw-r-- | 1 | xzh | xzh | 74 | 2108:4:2-15-58-27 | /home/xzh/share/linuxio/demo1/Makefile
-rw-rw-r-- | 1 | xzh | xzh | 4378 | 2108:4:2-20-12-7 | /home/xzh/share/linuxio/demo1/main.c
>>>>-rw-rw-r-- | 1 | xzh | xzh | 0 | 2108:4:2-16-29-29 | /home/xzh/share/linuxio/demo1/dir1/dir2/a1.txt
-rwxrwxr-x | 1 | xzh | xzh | 13680 | 2108:4:2-20-12-30 | /home/xzh/share/linuxio/demo1/a.out
lrwxrwxrwx | 1 | xzh | xzh | 6 | 2108:4:2-16-43-23 | /home/xzh/share/linuxio/demo1/link
-rwxrwxr-x | 1 | xzh | xzh | 4096 | 2108:4:2-15-42-8 | /home/xzh/share/linuxio/demo1/abc.txt
>>-rw-rw-r-- | 1 | xzh | xzh | 0 | 2108:4:2-16-29-38 | /home/xzh/share/linuxio/demo1/dir3/a3.txt
总文件数 = 8
➜  demo1
```