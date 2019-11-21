[TOC]



## 1. ==C 语言== 严格来说, ==没有== 字符串

- 1) 所以我们称呼的 **字符串** 大部分使用 **数组** 存储 (也有其他的: 字符串表 ...)

- 2) 基本上使用 **数组** 这种数据结构，模拟实现 **字符串**

- 3) 只不过 数组中的 **元素** 类型都是 **char** , 比如: `char buf[n]`

- 3) 当使用 **字符数组** 表示 **字符串** 时, 必须在末尾添加 **'\0'** 字符, 表示 **结束** , 否则在 **读取** 会出现 **乱码**



## 2. ==常量== --- 字符串 --> 指针

### 1. 这种类型的 ==字符串==, 存储在 ==text 代码段== 中, 属于 ==编译期== 的常量

```c
#include <stdio.h>
#include <string.h>

int main()
{
  /**
   * 【指针变量】指向【常量字符串】在【text 段】内的【地址】
   * => str 变量存储的只是【数组的起始内存地址】
   * => 常量字符串中的【每一个字符】都是以【数组】方式存储
   * => 而【数组】则是存储在【text 段】内
   */
  char* str = "Hello";

  printf("str = %p\n", str);
  printf("sizeof(str) = %ld\n", sizeof(str));
  printf("strlen(str) = %ld\n", strlen(str));
}
```

```
 ~/Desktop/main  make
gcc main.c
./a.out
str = 0x105c02f74
sizeof(str) = 8
strlen(str) = 5
```

- 1) str 是一个 **指针变量**

- 2) str 变量的 **值** ==> **内存地址**

- 3) **sizeof(str)** ==> sizeof(**指针变量**)
  - 1) 32位系统下占用 **4字节**
  - 2) 64位系统下占用 **8字节**

- 4) **strlen(str)** => 同上

- 5) **"Hello"** 字符串的存储
  - 存储在 **TEXT段** 中
  - str 指向 **TEXT段** 的存储字符串的 **起始地址**

### 2. 多次赋值 ==指向== 

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
  char* str1 = "hello";
  char* str2 = "hello";
  char* str3 = "hello";

  printf("str1 = %p\n", str1);
  printf("str2 = %p\n", str2);
  printf("str3 = %p\n", str3);
}
```

```
str1 = 0x101e81f8e
str2 = 0x101e81f8e
str3 = 0x101e81f8e
```

都是 **指向同一个** 常量字符串 的 内存起始地址

### 3. 修改 ==常量字符串== 会导致 ==运行崩溃==

```c
#include <stdio.h>
#include <string.h>

int main()
{
  char* str = "Hello"; 

  // 修改【常量】字符串中的某一个【字符】
  str[0] = 'h';
}
```

```
 ~/Desktop/main  make
gcc main.c
./a.out
make: *** [all] Bus error: 10
```

- 进程 **崩溃** 退出，报错 **Bus error: 10**
- 因为 **非用户区** 中的数据, **内核不允许** 对其进行 **修改** 



## 3. ==栈区== --- 字符串 --> 字符数组

### 1. `char str[] = {'a', 'b', 'c'}`  ==不会== 在末尾补充结束符 `\0`

#### 1. code

```c
#include <stdio.h>
#include <string.h>

int main() 
{
  // => 这种方式【不会】在末尾补充结束符 '\0' 表示【字符数组】的【结束字符】
  char str[] = {'1', '2', '3', '4', '5'};

  // 打印【str 数组】的【起始地址】
  printf("&str = %p\n", str);

  // 打印【str 数组】的【长度】
  printf("sizeof(str) = %ld\n", sizeof(str));

  // 打印【字符串】的【长度】
  printf("strlen(str) = %ld\n", strlen(str));

  // 打印【字符数组】的【字符】
  printf("str = %s\n", str);
}
```

#### 2. Linux (gcc) 运行效果

```
&str = 0x7ffe590be8e3
sizeof(str) = 5
strlen(str) = 5
str = 12345
```

**不会** 出现 **乱码**

#### 3. Mac (clang) 运行效果

```
&str = 0x7ffeee03a17b
sizeof(str) = 5
strlen(str) = 11
str = 12345����
```

- 最终打印字符串是 **乱码** 的
- 因为 **字符数组** 中没有 **结束字符** 

#### 4. 结论

- 两种系统下 **编译器** 出来的效果 **不一样**
- 可能 **Linux GCC** 做了一些 **特殊** 处理，导致 **不会乱码**

### 2. `char str[] = {'a', 'b', 'c', '\0'}` 需要 ==手动== 在末尾 ==追加== 结束字符 '\0'

#### 1. code 

```c
#include <stdio.h>
#include <string.h>

int main() 
{
  // => 这种方式【不会】在末尾补充结束符 '\0' 表示【字符数组】的【结束字符】
  char str[] = {'1', '2', '3', '4', '5', '\0'};

  // 打印【str 数组】的【起始地址】
  printf("&str = %p\n", str);

  // 打印【str 数组】的【长度】
  printf("sizeof(str) = %ld\n", sizeof(str));

  // 打印【字符串】的【长度】
  printf("strlen(str) = %ld\n", strlen(str));

  // 打印【字符数组】的【字符】
  printf("str = %s\n", str);
}
```

#### 2. Linux (gcc) 运行效果

```
&str = 0x7fffebff21d2
sizeof(str) = 6
strlen(str) = 5
str = 12345
```

**不会** 出现 **乱码**

#### 3. Mac (clang) 运行效果

```
&str = 0x7ffee056417a
sizeof(str) = 6
strlen(str) = 5
str = 12345
```

#### 4. 结论

数组的长度 = 字符串的长度 + 1



#### 2. 自动在最后添加结束符 `\0`

```c
#include <stdio.h>
#include <string.h>

int main() {
  // => 这种方式【会】自动在末尾补充结束符 \0
  char str[] = "12345";

  // 打印 `str 数组` 的 `起始地址`
  printf("str = %p\n", str);

  // 打印 `str 数组` 的 `长度`
  printf("sizeof(str) = %ld\n", sizeof(str));

  // 打印 `字符串` 的 `长度`
  printf("strlen(str) = %ld\n", strlen(str));
}
```

```
 ~/Desktop/main  make
gcc main.c
./a.out
str = 0x7ffeecf491ba
sizeof(str) = 6
strlen(str) = 5
```

### 3. `char str[n] = "123"` ==自动== 在末尾追加 字符数组 结束字符 '\0'

#### 1. code 

```c
#include <stdio.h>
#include <string.h>

int main() {
  // => 这种方式【会】自动在末尾补充结束符 \0
  char str[50] = "12345";

  // 打印 `str 数组` 的 `起始地址`
  printf("str = %p\n", str);

  // 打印 `str 数组` 的 `长度`
  printf("sizeof(str) = %ld\n", sizeof(str));

  // 打印 `字符串` 的 `长度`
  printf("strlen(str) = %ld\n", strlen(str));
}
```

#### 2. Linux (gcc) 运行效果

```
str = 0x7fff87652c00
sizeof(str) = 50
strlen(str) = 5
```

#### 3. Mac (clang) 运行效果

```
str = 0x7ffee7b3b140
sizeof(str) = 50
strlen(str) = 5
```

#### 4. 结论

- 1) 表现行为 **一致**
- 2) 直接分配 **数组** 指定长度 的 **连续内存** 来存储 **字符串**
- 3) **数组** 长度(容量) > **字符** 长度(个数)

### 4. `char str[n]` 使用 ==strcpy()== 填充 字符 (赋值)

```c
#include <stdio.h>
#include <string.h>

int main() 
{
  // 1. 定义一个 字符数组
  char str[101] = {0};

  // 2. 【清零】字符数组，这种方式【会】自动在末尾补充结束符 '\0'
  memset(str, 0, 101);

  // 3. 将处于【text 段】内的【常量字符串】挨个拷贝到【栈区】【str[n] 字符数组】中
  strcpy(str, "12345");

  // 4. 打印 `str 数组` 的 `起始地址`
  printf("str = %p\n", str);

  // 5. 打印 `str 数组` 的 `长度`
  printf("sizeof(str) = %ld\n", sizeof(str));

  // 6. 打印 `字符串` 的 `长度`
  printf("strlen(str) = %ld\n", strlen(str));
}
```

```
str = 0x7ffeeacba140
sizeof(str) = 101
strlen(str) = 5
```

### 5. sprintf() 增加 ==格式化== 拷贝 字符

```c
#include <stdio.h>
#include <string.h>

int main() 
{
  // 1.
  char str[101] = {0};

  // 2.
  memset(str, 0, 101);

  // 3.
  sprintf(str, "%s", "12345");

  // 4. 打印 `str 数组` 的 `起始地址`
  printf("str = %p\n", str);

  // 5. 打印 `str 数组` 的 `长度`
  printf("sizeof(str) = %ld\n", sizeof(str));

  // 6. 打印 `字符串` 的 `长度`
  printf("strlen(str) = %ld\n", strlen(str));
}
```

```
 ~/Desktop/main  make
gcc main.c
./a.out
str = 0x7ffeea98f140
sizeof(str) = 101
strlen(str) = 5
```

### 6. "Hello" 字符串 存储

- 1) 在 **局部函数栈帧** 分配一块连续内存
- 2) 作为 **字符数组** 的存储



## 4. ==堆区== --- 字符串 —> 字符数组 (只是存储在 ==堆区==)

### 1. code

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[]) 
{
  // 1、【栈上】指针变量，指向【堆区】存储的字符数组
  char *str = (char *)malloc(sizeof(char) * 100);

  // 2、向堆区数组内存中，写入字符
  strcpy(str, "dddd");

  // 3、打印数组的首地址
  printf("str = %p\n", str);

  // 4、sizeof(指针变量))
  printf("sizeof(str) = %ld\n", sizeof(str));

  // 5、strlen(指针变量)
  printf("strlen(str) = %ld\n", strlen(str));
}
```

```
➜  main make
gcc main.c
./a.out
str = 0x7fbdb9403220
sizeof(str) = 8
strlen(str) = 4
➜  main
```

### 2. 内存模型

![](cyuyan_05.png)

### 3. 通过 ==数组下标== 读写 字符数组

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
  // 1. 
  char str[4] = "ABC";
  
  // 2. 
  for (int i = 0; i < strlen(str); i++) 
    printf("%c ", str[i]);
}
```

### 4. 通过 ==指针变量== 读写 字符数组

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
  // 1. 
  char str[4] = "ABC";

  // 2. 
  char*p;
  for (p = str; *p != '\0'; p++) 
    printf("%c ", *p);
}
```



## 5. 字符串 ==一级== 指针模型

### 1. `char* p = "hello"` 

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[])
{
  char *str = "Hello";
  printf("str = %p\n", str);
  printf("&str = %p\n", &str);
  printf("sizeof(str) = %lu\n", sizeof(str));
  printf("strlen(str) = %lu\n", strlen(str));
}
```

```
->  gcc main.c
->  ./a.out
str = 0x10590ef54
&str = 0x7fff5a2f1038
sizeof(str) = 8
strlen(str) = 5
->
```

- **str** 是一个**指针变量** 占用 **8(字节)**
  - **自己** 在 **局部栈** 上的 **内存地址** 为 **0x7fff5a2f1038**
  - 变量的值为 **0x10590ef54**

- 内存地址 **0x10590ef54** 才是字符串 **Hello\0** 存储位置

### 2. `char str[n]`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[])
{	
  //1.
  char str1[10] = "Hello";
  printf("sizeof(str1) = %lu, strlen(str1) = %lu\n", sizeof(str1), strlen(str1));

  //2. 
  char str2[] = "Hello";
  printf("sizeof(str2) = %lu, strlen(str2) = %lu\n", sizeof(str2), strlen(str2));
}
```

```
->  gcc main.c
->  ./a.out
sizeof(str1) = 10, strlen(str1) = 5
sizeof(str2) = 6, strlen(str2) = 5
```

- 都是字符的 **数组**
- 都是局部栈帧上的内存块，直接用来存储字符串
- 两个数组的 **有效** 长度为5
- 两个数组的 **实际** 长度为6，因为最后自动补一个`'\0'`作为**结束符**

### 3. `char* p = malloc(n) & strcpy(p, "hello") `

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char const *argv[])
{
  char* p = (char*)malloc(100);
  memset(p, 0, 100);
  strcpy(p, "hello");
  
  printf("sizeof(p) = %lu\n", sizeof(p));
  printf("p = %p\n", p);
  printf("strlen(p) = %lu\n", strlen(p));
  printf("%s\n", p);

  free(p);
}
```

```
sizeof(p) = 8
p = 0x7fa55ec02d10
strlen(p) = 5
hello
```

- 1) **sizeof(p) = 8** : p 是一个 **指针变量** 
- 2) **p = 0x7fa55ec02d10** : 指向一块 0x7fa55ec02d10 **堆区的** 起始地址
- 3) **strlen(p) = 5** : 0x7fa55ec02d10 **堆区的** 内存中，存储 **5** 个字符 **hello**



## 6. 字符串 ==错误== 使用

### 1. 修改 ==常量== 字符串

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[]) 
{
  //1. 
  char *str = "cccc";

  //2.
  str[0] = 'd';

  //3. 同上一样崩溃
#if 0
  strcpy(str, "dddd");
#endif
}
```

```
->  gcc main.c
->  ./a.out
Bus error: 10
```

- str是一个 **指针变量**
- str指向的内存是`.text`段，也就是说`常量字符`存储在`.text`段
- `.text`段是`只读`的，不能够被修改
- 一旦修改`.text`段的数据，直接就程序崩溃了
- 但是有可能 **其他类型的编译器** 是允许这样使用的

### 2. `strcpy(NULL, "hello")`

#### 1. 

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[]) 
{
  //1. 仅仅只是个栈区的指针变量，但并没有赋值其指向的【内存块】
  char *str; 

  //2.
  strcpy(str, "aaaaa");
}
```

```c
->  gcc main.c
->  ./a.out
Segmentation fault: 11
```

- 可以使用 gdb 调试找到, 最终 **崩溃** 的地方就是 **strcpy()** 调用地方
- 因为指针变量 **str** 根本 **没有指向** 任何的可使用的 **内存空间**

#### 2. 

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[]) 
{
  // 1. malloc() 失败, 返回 NULL
  char *str = malloc(100);

  // 2. str == NULL
  strcpy(str, "aaaaa");

  // 3. 一定要对 str == NULL 判断
#if 1
  if (str)
  {
    strcpy(str, "aaaaa");
  }
#endif
}
```

其实和上面是一样的.

### 5. 错误3: 修改 ==数组名==

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, const char * argv[]) 
{    
  //1. 数组名是【常量指针】，不能对其赋值
  char arr[10];
  
  //2. 
  arr = "adawdw"; 
}
```

编译 **报错** 

```c
main.c:10:9: error: array type 'char [10]' is not assignable
    arr = "adawdw";
    ~~~ ^
1 error generated.
```

'char [10]' 这个数组类型变量，是不能被修改的。

为什么不能修改？

- 数组名 **arr** 一个指针变量 , 且是 **常量指针变量** , 所以不能被修改
- 数组名 **arr** 指向着数组的 **第一个元素的地址**
- **编译器** 规定 数组名 **arr** 是 **只读** , 为了能够正确的 **废弃** 数组对应的 **内存块**



## 7. `char* p = 'haha'` 存储在 elf 文件 `.text 段`

### 1. 使用一次常量字符串的 a.out 文件大小

```c
#include <stdio.h>

int main()
{
  char* p1 = "Hello World!";
}
```

```
➜  share gcc -c main.c
➜  share size main.o
   text	   data	    bss	    dec	    hex	filename
     89	      0	      0	     89	     59	main.o
➜  share
```

### 2. 使用多次常量字符串的 a.out 文件大小

```c
#include <stdio.h>

int main()
{
  char* p1 = "Hello World!";
  char* p2 = "Hello World!";
  char* p3 = "Hello World!";
}
```

```
➜  share gcc -c main.c
➜  share size main.o
   text	   data	    bss	    dec	    hex	filename
    103	      0	      0	    103	     67	main.o
➜  share
```

### 3. 对比前后两次发生的变化

- 1）【data段】没有发生变化
- 2）【text段】由 **103 - 89 = 14** , 增加了14个字节
  - 肯定没有增加三次 `"Hello World!"`字节长度
  - 增加的 **14(字节)**  应该是 **程序代码**

所以对同一段 **常量字符串** 多次引用，只会在 **text段** 有一份占用内存，其他的引用只是一个指针变量指向这块内存。

### 4. 多次引用 ==常量字符串== , 只使用 ==一份== 内存

```c
#include <stdio.h>

char* str1 = "1234567890";

int main()
{
  char* str2 = "1234567890";
  char* str3 = "1234567890";
  char* str4 = "1234567890";

  printf("str1 = %p\n", str1);
  printf("str2 = %p\n", str2);
  printf("str3 = %p\n", str3);
  printf("str4 = %p\n", str4);
}
```

```
->  gcc main.c
->  ./a.out
str1 = 0x8048510
str2 = 0x8048510
str3 = 0x8048510
str4 = 0x8048510
->
```



## 8. 字符串 ==二级== 指针模型

### 1. `char str[10][30]` 

#### 1. 赋值 `str[i][j]`

```c
#include <stdio.h>

int main(int argc, const char * argv[])
{
  // 栈上分配 10列*30行 的二维数组
  char str[10][30] = {"111", "2222", "333333"};
  printf("sizeof(str) = %lu\n", sizeof(str));
}
```

```
sizeof(str) = 300
```

从长度看，是一个二维的字符数组，总长度为 `10 * 30 * 1` 字节。

此种情况下对于 `str`

- (1) str是一个二维的数组，也可以看做是一个指针
- (2) `str[i]` 就是**一维数组**的起始地址
- (3) 二维数组的内存空间，全部再栈上分配
- (4) 一维数组内存中，存储字符

#### 2. 修改 `str[i][j]`

```c
#include <stdio.h>

int main(int argc, const char * argv[])
{
  //1. 栈上分配 10列*30行 的二维数组
  char str[10][30] = {"111", "2222", "333333"};

  //2. 修改str[i][j]存储的字符
  str[0][0] = '4';

  printf("sizeof(str) = %lu\n", sizeof(str));
}
```

```
->  gcc main.c
->  ./a.out
sizeof(str) = 300
```

正常运行，因为字符都是存储在**局部栈帧**上，都是可以读写的。

### 2. `char* str[3]`

#### 1. `str[i]` 都是 指针变量

```c
#include <stdio.h>

int main(int argc, const char * argv[])
{
  char *str[3] = {
    "111",  //len=3
    "2222", //len=4 
    "333333" // len=5
  };

  printf("sizeof(void*) = %lu\n", sizeof(void*));
  printf("sizeof(str) = %lu\n", sizeof(str));
}
```

```
➜  main make
gcc main.c
./a.out
sizeof(void*) = 8
sizeof(str) = 24
➜  main
```

- 从长度看 , 就是 **三个指针变量** , 一共占用`8*3=24`个字节。

- 此种情况下对于 **str**
  - str是一个 `char*` 类型 指针变量 的 **数组**
  - `str[i]` 对应一个指针变量，指向存储在 text 段 的某一个 **内存地址**
  - 字符存储在 **只读内存区**，依然按照 **字符数组** 的形式存储

#### 2. 指针操作 `char*` 数组

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
  //1. char* 数组
  char* strs[10] = {
    "11111", //=> char* 指针变量
    "222222",
    "3333333",
  };
  
  //2. 获取每一个子串的长度
  printf("strlen(strs[0]) = %lu\n", strlen(strs[0]));
  printf("strlen(strs[1]) = %lu\n", strlen(strs[1]));
  printf("strlen(strs[2]) = %lu\n", strlen(strs[2]));
  
  //3. p指向的是 char* 指针变量，所以 *p 才能取到 char* 指向的 char内存
  char** p = strs;
  for (; *p ; p++)
  {
    printf("%s\n", *p);
  }
}
```

```
➜  gcc 004-char_pointer_arr.c
➜  ./a.out
strlen(strs[0]) = 5
strlen(strs[1]) = 6
strlen(strs[2]) = 7
11111
222222
3333333
➜
```

### 3. `main(int argc, const char * argv[])`

- 1) 第二个参数 `char * argv[]` 
  - 只是一个 **指向** `char*` **指针数组** 的 **指针变量**
  - 但是并 **不知道** 这个 **指针数组** 有多少个元素 (数组的 **长度**)

- 2) 所以还需要 **额外** 传入一个参数 **int argc** 来告诉 main() , 数组的 **长度**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[])
{
  int i;
  for(i=0; i<argc; ++i)
    printf("argv[%d] = %s\n", i, argv[i]);
}
```

```
➜  main gcc main.c
➜  main ./a.out 1 2 3 4 5 wo ri ni ma
argv[0] = ./a.out
argv[1] = 1
argv[2] = 2
argv[3] = 3
argv[4] = 4
argv[5] = 5
argv[6] = wo
argv[7] = ri
argv[8] = ni
argv[9] = ma
➜  main
```

### 4. `char **str`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, const char * argv[])
{
  //1. 分配内存一、分配 char*[3] 数组的内存
  char **str = (char **)malloc(sizeof(char*) * 3); 
  
  //2. str[i]写入字符串
  for (int i = 0; i < 3; i++) 
  {
    // 分配内存二、给str[i]继续分配`指向`的内存
    str[i] = (char*)malloc(sizeof(char) * 64);
    strcpy(str[i], "AAAAAA");
  }
  
  //3.
  printf("sizeof(str)      = %lu\n", sizeof(str));
  printf("sizeof(str[0])   = %lu\n", sizeof(str[0]));
  printf("sizeof(str[1])   = %lu\n", sizeof(str[1]));
  printf("sizeof(str[2])   = %lu\n", sizeof(str[2]));

  //4. 修改其中的字符
  str[0][0] = 'a';

  //5. 
  printf("str[0]   = %s\n", str[0]);

  // 6. 释放内存
  for (int i = 0; i < 3; i++) 
  {
    free(str[i]);
  }
  free(str);
}
```

```
➜  main make
gcc main.c
./a.out
sizeof(str)      = 8
sizeof(str[0])   = 8
sizeof(str[1])   = 8
sizeof(str[2])   = 8
str[0]   = aAAAAA
➜  main
```

- 从长度看，对于  **str** 和 `str[i]` 都是 **指针变量** ， 占用 8(字节)

- 此种情况下的 str、`str[i]` 
  - 1) 都是 **指针变量**
  - 2) **str** 指向 `&str[0]` 元素的 **内存地址**
  - 3）`str[i]` 指向的 **内存** , 是最终 **存储字符** 的内存


## 9. 定义错误码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define INFO  0
#define WARN  1
#define ERR   2

const char* const STRINGS[] = {
  [INFO] = "info",
  [WARN] = "warning",
  [ERR] = "error",
};

const char* err_to_str(int err)
{
  return STRINGS[err];
}

int main()
{
  printf("%s\n", err_to_str(INFO));
  printf("%s\n", err_to_str(WARN));
  printf("%s\n", err_to_str(ERR));
}
```
