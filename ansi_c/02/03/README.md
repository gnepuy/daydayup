[TOC]



## 1. 三种 const 指针变量 写法的区别

### 1. `const T *p` 或 `T const *p` : const 修饰 `*p` 值

> 简单记忆: const 总是在 `*p` 前面.

```c
int main()
{
  // 1、T const *p;
  char const *p =  "hello";

  // 2、const T* const p;
  // const char *p =  "hello"; 与上没有差别

  // 修改【p】的【指向】：编译、运行都 ok
  p = "aaa"; 

  // 修改【*p】指向的【内存中的数据】：编译 error、运行 crash
  *(p+2) = 'b';
}
```

`const T* p` 的特定：

- 1、【允许】修改【p】的【地址值】
- 2、【不允许】修改【*p】指向的【内存数据】

### 2. `T* const p` : const 修饰 `p` 指向

> const 放在 `*` 后面, `p` 前面

```c
int main()
{    
  char * const p =  "hello";

  // 修改【p】的【指向】：编译 error
  p = "aaa";

  // 修改【*p】指向的【内存中的数据】：编译 ok、运行 crash
  *(p+2) = 'b';
}
```

`T* const p` 的特定：

- 1、【不允许】修改【p】的【地址值】
- 2、【允许】修改【*p】指向的【内存数据】

刚好与上面的`const T* p`相反。

### 3. `const T* const p` : ==同时== 修饰 `p` 与 `*p`

```c
int main()
{    
  const char * const p =  "hello";
 
  // 编译error
  p = "aaa";

  // 编译error
  *(p+2) = 'b';
}
```

`const T* const p` 的特定：

- 1、【不允许】修改【p】的【地址值】
- 2、【不允许】修改【*p】指向的【内存数据】

结论：`p` 与 `*p` 都不允许进行修改。


## 2. C 与 C++ 中的 const 区别

### 1. c 语言 const : 运行时 - 只读变量

- 1) 只能修饰: **变量**、**函数的返回值**

- 2) 被修饰的变量，是一个 **运行期** 保存在 **内存** 中的 **只读变量**
  - 注意: 只针对 **函数内部** 的 **局部 const 变量**）
  - 仍然可以通过 **地址** 的方式 **间接修改**

- 3) 对于 const 修饰 **函数内部**  局部变量 => 运行时的 **只读变量** (数据段)

- 4) **但是** const 修饰 **全局变量** => **常量** (代码段)

### 2. c++ 语言 const : 编译期 - 常量

- 1) **C** 能办到的，**C++** 都支持
- 2) 还能修饰 **成员方法**（实际上修饰的是 **this** 指针变量）
- 3) 被修饰的变量，是一个 **编译期** 保存在 **代码段** 的 **常量**，无法对其修改
- 4) 只要试图对其 **取地址**，都会触发 **栈帧上重新分配内存**
- 5) 而 **分配后的内存** 与存储在 **代码段** 中的 **常量** 没有半毛钱关系
- 6) **C++** 中的 const 是 **真正** 的 **常量**



## 3. const 修饰 ==函数栈帧== 上的 ==局部== 变量

### 1. const 修饰的变量，从 ==语法== 层面, 直接修改会 报错

```c
#include <stdlib.h>
#include <stdio.h>

int main()
{   
  //1. 
  const int a = 10;

  //2. 与上等价
  int const b = 10;

  //3. 编译报错 error: read-only variable is not assignable
  a = 11;

  //4. 编译报错 error: read-only variable is not assignable
  b = 12;
}
```

编译报错

```c
->  gcc main.c
main.c:13:4: error: read-only variable is not assignable
        a = 11;
        ~ ^
main.c:16:4: error: read-only variable is not assignable
        b = 12;
        ~ ^
2 errors generated.
```

### 2. 但是可以通过 ==内存地址== 间接修改

```c
#include <stdlib.h>
#include <stdio.h>

int main()
{
  //1.
  const int name = 10;

  //2.
  int* p = &name;

  //3.
  *p = 20;
  printf("name = %d\n", name);
}
```

编译运行

```
->  gcc main.c
main.c: In function ‘main’:
main.c:10:14: warning: initialization discards ‘const’ qualifier from pointer target type [-Wdiscarded-qualifiers]
     int* p = &name;
              ^
->  ./a.out
name = 20
->

```

- const 变量值已经被 改变
- **局部栈帧** 上的 const 变量，在C语言中只是一个 **运行时的只读变量**
- 仍然可以通过 **取地址** 的方式间接修改

### 3. name 符号 存储在 ==局部栈帧== 内存区

查看 **name 符号** 存储位置

```
->  readelf -s a.out | grep name
->  objdump -D a.out | grep name
->
```

- 没有找到
- 因为局部栈帧上的符号，只有在 **程序运行时** 创建 **栈帧** 之后，才会分配 **该符号的内存地址**



## 4. const 修饰 ==全局== 变量

### 1. 此时的 const 变量, 真的是 ==编译期== 常量，修改就会 ==崩溃==

```c
#include <stdio.h>

// const 修饰函数体外的全局变量name
const int name = 10;

int main(int argc, const char * argv[])
{   
  int* p = &name;
  *p = 11;
}
```

```c
->  gcc const.c
const.c:7:10: warning: initializing 'int *' with an expression of type 'const int *' discards qualifiers
      [-Wincompatible-pointer-types-discards-qualifiers]
    int* p = &a;
         ^   ~~
1 warning generated.
->  ./a.out
Bus error: 10
->
```

- 编译有警告
- 运行时【直接崩溃】

### 2. name 符号 存储在 ==rodata 只读段== 内存区 

符号 name 放到了序号为 **16** 的section

```c
->  readelf -s a.out | grep name
  53: 08048480     4 OBJECT  GLOBAL DEFAULT   16 name
->
```

查看序号为 **16** 的 section 是 **rodata 只读** 数据段

```
->  readelf -S a.out | grep 16
  [ 2] .note.ABI-tag     NOTE            08048168 000168 000020 00   A  0   0  4
  [12] .plt              PROGBITS        080482b0 0002b0 000020 04  AX  0   0 16
  [14] .text             PROGBITS        080482e0 0002e0 000182 00  AX  0   0 16
  [16] .rodata           PROGBITS        08048478 000478 00000c 00   A  0   0  4
  [28] .shstrtab         STRTAB          00000000 0016bd 00010a 00      0   0  1
->
```

所以如果是修改了此时的 **name 变量的值** , 那么在 **运行时** 就会 **崩溃**





## 5. const 修饰数组

### 1. const T `arr[n]` : 修饰 `arr[i]`

```c
#include <stdlib.h>
#include <stdio.h>

int main() 
{    
  const int arrA[3] = {1, 2, 3};

  // 修改数组中的【元素】的值
  arrA[0] = 4;
}
```

编译报错

```
➜  main make
gcc main.c
main.c:8:11: error: read-only variable is not assignable
  arrA[0] = 4;
  ~~~~~~~ ^
1 error generated.
make: *** [all] Error 1
➜  main
```

提示不能修改 **arrA[0]**

### 2. const `T* arr[n]` : 修饰 `arr[i]`

#### 1. 无法修改 `arr[i]` 值

```c
#include <stdlib.h>
#include <stdio.h>

int main() 
{    
  // const char* 数组
  const char* arr[3] = {"AAA", "BBB", "CCC"};

  // error
  arr[2][1] = 'D'; 
}
```

编译报错

```
➜  main make
gcc main.c
main.c:28:13: error: read-only variable is not assignable
  arr[2][1] = 'D';
  ~~~~~~~~~ ^
1 error generated.
make: *** [all] Error 1
➜  main
```

#### 2. 但是可以修改 `arr[i]` 指向

```c
#include <stdlib.h>
#include <stdio.h>

int main() 
{    
  // const char* 数组
  const char* arr[3] = {"AAA", "BBB", "CCC"};

  // 允许修改 char* 指向其他的内存块（"DDD"存储在text段中某一块内存）
  *(arr+1) = "DDD";
  printf("arr[1] = %s\n", arr[1]);

  arr[1] = "EEE";
  printf("arr[1] = %s\n", arr[1]);
}
```

```
➜  main make
gcc main.c
./a.out
arr[1] = DDD
➜  main
```

### 3. const `T*` const `arr[n]` : 同时修饰 `arr[i]` 值 和 指向

```c
#include <stdlib.h>
#include <stdio.h>

int main() 
{    
  // const char* const [i] 数组
  const char* const arr[3] = {"AAA", "BBB", "CCC"};

  // error
  arr[2][1] = 'D'; // "BBB"返回的text段中的内存地址

  // error
  *(arr+1) = "DDD";
  arr[1] = "DDD"; // 与上等价
  printf("arrA[1] = %s\n", arr[1]);
}
```

```
➜  main make
gcc main.c
main.c:10:13: error: read-only variable is not assignable
  arr[2][1] = 'D'; // "BBB"返回的text段中的内存地址
  ~~~~~~~~~ ^
main.c:13:12: error: read-only variable is not assignable
  *(arr+1) = "DDD";
  ~~~~~~~~ ^
main.c:14:10: error: read-only variable is not assignable
  arr[1] = "DDD"; // 与上等价
  ~~~~~~ ^
3 errors generated.
make: *** [all] Error 1
➜  main
```


两处都不能再修改。



## 6. func(const 形参)

### 1. 基本数据类型

```c
void func(const int a) 
{
  a = 5;
}
```

只是从 **语法检测** 的层面上告知别人这个函数 **不允许修改** 形参的值.

### 2. 指针类型

```c
void func(const int* const p) 
{
}
```
