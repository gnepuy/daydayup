[TOC]



## 1、环境变量 属于 ==某个进程==

![Snip20180123_10](image/Snip20180123_10.png)



## 2、读取 进程的 环境变量列表

```c
#include <stdio.h>

// 引用外部符号
// => environ 指向 char* arr[n] 数组，arr[i]指向rodata段中的字符串
// => p = &arr[0] 指向数组的第一个元素的内存地址
// => p++ 往后移动 sizeof(char*) 字节数
// => *p == '\0'(0) 时停止遍历
extern char** environ;

int main()
{
  // 1. 
#if 0
  char* arr[n];
  char** p = &arr[0]; // p 步长为 sizeof(char**) 字节数
#endif
  char** p = environ;

  // 2.
  for (; *p != 0; p++)
    printf("%s\n", *p);
}
```

```
->  gcc process_envir.c
->  ./a.out
LC_PAPER=zh_CN.UTF-8
LC_ADDRESS=zh_CN.UTF-8
XDG_SESSION_ID=2
LC_MONETARY=zh_CN.UTF-8
TERM=xterm-256color
SHELL=/bin/bash
SSH_CLIENT=192.168.125.34 54975 22
LC_NUMERIC=zh_CN.UTF-8
SSH_TTY=/dev/pts/18
USER=xzh
LC_TELEPHONE=zh_CN.UTF-8
MAIL=/var/mail/xzh
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
QT_QPA_PLATFORMTHEME=appmenu-qt5
LC_IDENTIFICATION=zh_CN.UTF-8
PWD=/home/xzh/share/linux_c
LANG=en_US.UTF-8
LC_MEASUREMENT=zh_CN.UTF-8
PS1=\[\033[00m\]->
SHLVL=1
HOME=/home/xzh
ts=4
LOGNAME=xzh
SSH_CONNECTION=192.168.125.34 54975 192.168.125.43 22
LC_CTYPE=zh_CN.UTF-8
XDG_RUNTIME_DIR=/run/user/1000
LC_TIME=zh_CN.UTF-8
LC_NAME=zh_CN.UTF-8
OLDPWD=/home/xzh/share
_=./a.out
->
```



## 3、修改 进程 环境变量

### 1. C 标准库


```c
#include <stdlib.h>

char *
getenv(const char *name);

int
setenv(const char *name, const char *value, int overwrite);

int
putenv(char *string);

int
unsetenv(const char *name);
```


```c
#include <stdio.h>
#include <stdlib.h>

extern char** environ;

int main()
{
  // 添加环境变量
	putenv("myvar=xiong");

  // 打印所有环境变量键值对
  char** p = environ;
  for (; *p != 0; p++)
    printf("%s\n", *p);
 
  // 删除环境变量
  unsetenv("myvar");
}
```

### 2. linux 系统函数

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
```



## 4、删除 进程 环境变量

```c
#include <stdio.h>
#include <stdlib.h>

extern char** environ;

int main(int argv, char* argr[])
{
	int i;
	char** p;

	// 1. 清除进程环境变量列表
	clearenv();

	// 2. 至少传入2个命令行参数
	if (argv < 2) {
		fprintf(stderr, "usage: ./a.out key1=value1 key2=value2 ...\n");
		return -1;
	}
  
  // 3. 添加到进程的环境变量列表
	for(i = 1; i < argv; i++)
		putenv(argr[i]);
	
  // 4. 遍历环境变量列表
	p = environ;
	for (; *p != 0; p++)
		printf("%s\n", *p);
}
```

```
->  gcc process_envir.c
->  ./a.out name=xiong age=19
name=xiong
age=19
->
```



## 5、环境变量的修改, 只针对 ==当前进程==

### 1. 进程1 代码: main1.c

```c
#include <stdio.h>
#include <stdlib.h>

extern char** environ;

int main()
{
	// 方便查看环境变量输出，先删除进程的所有环境变量
	clearenv();
	
	// myvar=xiong, 1表示覆盖已有的环境变量值
	setenv("myvar", "xiong", 1);
	
	// 遍历打印进程的所有环境变量
  char** p = environ;
  for (; *p != 0; p++)
    printf("%s\n", *p);
}
```

### 2. 进程2 代码: main2.c

```c
#include <stdio.h>
#include <stdlib.h>

extern char** environ;

int main()
{
	// 遍历打印进程的所有环境变量
  char** p = environ;
  for (; *p != 0; p++)
    printf("%s\n", *p);
}
```

### 3. 编译链接运行 main1.c

```
xzh@xzh-VirtualBox:~/share$ gcc main1.c
xzh@xzh-VirtualBox:~/share$ ./a.out
myvar=xiong
```

### 4. 编译链接运行 main2.c

```
xzh@xzh-VirtualBox:~/share$ gcc main2.c
xzh@xzh-VirtualBox:~/share$ ./a.out
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
LC_MEASUREMENT=ru_RU.UTF-8
SSH_CONNECTION=10.13.55.199 53508 10.13.55.105 22
LESSCLOSE=/usr/bin/lesspipe %s %s
LC_PAPER=ru_RU.UTF-8
LC_MONETARY=ru_RU.UTF-8
LANG=en_US.UTF-8
LC_NAME=ru_RU.UTF-8
XDG_SESSION_ID=4
USER=xzh
PWD=/home/xzh/share
HOME=/home/xzh
LC_CTYPE=zh_CN.UTF-8
SSH_CLIENT=10.13.55.199 53508 22
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
LC_ADDRESS=ru_RU.UTF-8
LC_NUMERIC=ru_RU.UTF-8
SSH_TTY=/dev/pts/2
MAIL=/var/mail/xzh
TERM=xterm-256color
SHELL=/bin/bash
SHLVL=1
LC_TELEPHONE=ru_RU.UTF-8
LOGNAME=xzh
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
XDG_RUNTIME_DIR=/run/user/1000
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
LC_IDENTIFICATION=ru_RU.UTF-8
LESSOPEN=| /usr/bin/lesspipe %s
LC_TIME=ru_RU.UTF-8
OLDPWD=/home/xzh
_=./a.out
```

删除 main1.c 最终对应的进程的所有环境变量，并无法删除 main2.c 对应的进程拥有的环境变量。