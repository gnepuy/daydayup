[TOC]



## 1. 指针变量 占用 ==固定== 字节数

```c
#include <stdio.h>
#include <stdlib.h>

struct person
{
  int     age;
  char    name[64];
};

int main()
{   
  // 1. 栈上分配的内存块
  char ch = 'A';
  int age = 19;
  double seconds = 19999;
  struct person per = {19, "xiongzenghui"};

  // 2. 栈上指针变量，指向上面分配的内存块
  char* p1            = &ch;
  int* p2             = &age;
  double* p3          = &seconds;
  struct person* p4   = &per;
  struct person** p5;
  struct person*** p6;
  struct person**** p7;

  // 3.
  printf("sizeof(p1) = %ld\n", sizeof(p1));
  printf("sizeof(p2) = %ld\n", sizeof(p2));
  printf("sizeof(p3) = %ld\n", sizeof(p3));
  printf("sizeof(p4) = %ld\n", sizeof(p4));
  printf("sizeof(p5) = %ld\n", sizeof(p5));
  printf("sizeof(p6) = %ld\n", sizeof(p6));
  printf("sizeof(p7) = %ld\n", sizeof(p7));
}
```

```
 ~/Desktop/main  gcc main.c
 ~/Desktop/main  ./a.out
sizeof(p1) = 8
sizeof(p2) = 8
sizeof(p3) = 8
sizeof(p4) = 8
sizeof(p5) = 8
sizeof(p6) = 8
sizeof(p7) = 8
```

- 1、不管是 **指向任何类型** 的指针变量，自身的长度都是等于 **8字节**
- 2、不管是 **多少级** 的指针变量，，自身的长度都仍然等于 **8字节**
- 3、在 **32** 位下，等于 **4字节**



## 2. 指针 ==步长== 是什么？

当一个 **指针变量** 进行如下 **数值计算** 时，

- 1) 加法：指针变量 + 1、指针变量 + n、指针变++、++指针变量
- 2) 减法：指针变量 - 1、指针变量 - n、指针变—、--指针变量

**跳过** 的 **字节数** 就叫指针的 **步长**.



## 3. p + 1

### 1. 测试 `char*` 步长

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
  char ch = 'A';
  char* p1 = &ch;
  char* p2 = p1+1;

  printf("p1 = %p\n", p1);
  printf("p2 = %p\n", p2);
}
```

```
p1 = 0x7ffeefa9a09f
p2 = 0x7ffeefa9a0a0
```

0x7ffeefa9a0a0 - 0x7ffeefa9a09f = 0x1 = **1 字节**

### 2. 测试 `int*` 步长

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
  int age = 99;
  int* p1 = &age;
  int* p2 = p1+1;

  printf("p1 = %p\n", p1);
  printf("p2 = %p\n", p2);
}
```

```
p1 = 0x7ffee3cd509c
p2 = 0x7ffee3cd50a0
```

0x7ffee3cd50a0 - 0x7ffee3cd509c = 0x4 = **4 字节**

### 3. 测试 `struct xxx*` 步长

```c
#include <stdio.h>

struct person
{
  int age;
  char name[64];
  char sex;
};

int main(int argc, char const *argv[])
{
  struct person per = {99, "xiong", 'm'};
  printf("sizeof(per) = 0x%lx\n", sizeof(per));

  struct person* p1 = &per;
  struct person* p2 = p1+1;

  printf("p1 = %p\n", p1);
  printf("p2 = %p\n", p2);
}
```

```
sizeof(per) = 0x48
p1 = 0x7ffeea904060
p2 = 0x7ffeea9040a8
```

0x7ffeea9040a8 - 0x7ffeea904060 = 0x48 = **72 字节**

### 4. 总结 p + 1

| **指向变量** 指向的 **数据类型** | **(p + 1)** 向后 跳动的字节数 |
| --------------- | ------------------- |
| `char*` p           | sizeof(**char**) 字节数 |
| `int*` p            | sizeof(**int**) 字节数 |
| `double*` p         | sizeof(**double**) 字节数 |
| `struct person*` p  | sizeof(**struct person**) 字节数 |



## 4. p + n

```c
int *p = ..;
p++;
p--;
p+20;
p-20;
```

跳过 `n * sizeof(指向类型)` 字节数.



## 5.  `*p` 与 `*p = 1`

| 指针变量操作 | 含义                                 |
| ------------ | ------------------------------------ |
| int a = `*p` | **读取** 指针变量指向的内存中的数据  |
| `*p` = 1     | 向指针变量指向的内存中 **写入** 数据 |

指向 **类型的不同**，读写的 **字节数** 也不同

### 1. 一次性 读写 **4** 字节

```c
int *p = ..;
int age = *p; // 读取内存
*p = 99; // 写入内存
```

### 2. 一次性 读写 **1** 字节

```c
char ch = 'A';
char* p = &ch;
*p = 'B';
```

### 3. 示例: 查找 ==指定数据== 所在的 ==内存地址==

- 1) 从起始地址 **0x1db8000** ，结束地址 **0x1db81000**
- 2) 假定检索数据是 **int** 类型，值是 **10**
- 3) **1个字节** 最大表示数值 **2^8 -1**  == **255**
- 4) 所以 **10** 可以被 **1个字节** 表示
- 5) 只需要从 **起始地址** 开始，向后逐个 **1个字节** 检索即可

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void* getAddr()
{
  // 1. 开始检索的内存地址
  void* start = 0x1db8000;

  // 2. 结束检索的内存地址
  void* end 	= 0x1db81000;
  
  /**
   *	3. 将指针变量的【步长】变为【1个字节】
   *	
   *  - (1) 首先将 `void*` 转换为 `char*`
   *  - (2) 将指针变量【步长】变为【1个字节】
   *  - (3) 即每次【p+1】，只向后跳跃【1个字节】
   */
  char* p = start;
  
  // 4. 从起始地址处，逐字节向后检索内存中是否存储的是 10
  for (; p != end; p++) 
  {
    /**
     *	4.1 取出当前【1个字节】内存中，存储的【int】类型值
     *	- 1）
     *  - 2）
     * - 3）
     */

    /**
     * 4.2: 按照【int 类型】占用【4个字节】的内存，即需要一次性【吞4个字节】
     * 
     * - 1) `char*` => `int*` 完成 `指针变量的步长` 转换 
     * - 2) 改变指针的【步长】从【1个字节】变成【4个字节】
     */
    int *px = (int*)p;
    
    // 4.2: 如果开始的【1字节】值为10，那么返回逐个【4字节】内存的【起始地址】
    if (*px == 10) 
      return p;
  }
  
  return NULL;
}
```

- `&` => 取地址 => `定位` 内存块的 **起始地址**
- `*` => 解地址 => `读/写` 地址对应的的 **内存块中的数据**



## 6. 指针 ==类型转换==

### 1. 就是 ==改变== 指针变量的 ==步长==

```c
int *p1 = ..;         // 指针步长=4字节
char* p2 = (char*)p1; // 指针步长=1字节
long* p3 = (long*)p1; // 指针步长=8字节
```

### 2. 示例1

```c
#include <stdio.h>

int main()
{
  int buf;

  // p1 步长为 sizeof(int) = 4字节
  int* p1 = &buf; 
  
  // p2 步长为 sizeof(char) = 1字节
  char* p2 = (char*)p1; 
  
  // p2 每次【读写 1个字节】的内存
  *(p2) = 0x11; // *(p2 + 0) = 0x11;
  *(p2 + 1) = 0x22;
  *(p2 + 2) = 0x33;
  *(p2 + 3) = 0x44;

  // p1 每次【读写 4个字节】的内存
  printf("*p1 = %02x\n", *p1);
}
```

```
➜  main make
gcc main.c
./a.out
*p1 = 44332211
➜  main
```

### 3. 示例2

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
  int data = 0x11223344;
  
  // int*(4字节) => char*(1字节)
  char* p = (char*)&data;
  
  // 从4字节连续内存的起始位置，【逐个字节】读取
  printf("*p = %x\n",*p);
  printf("*(p+1) = %x\n",*(p+1));
  printf("*(p+2) = %x\n",*(p+2));
  printf("*(p+3) = %x\n",*(p+3));
}
```

```
➜  main make
gcc main.c
./a.out
*p = 44
*(p+1) = 33
*(p+2) = 22
*(p+3) = 11
➜  main
```




## 7. 指针变量 ==相减==

### 1. 两个指针变量 ==已经== 明确 ==指向== 内存地址

```c
#include <stdio.h>
#include <string.h>

int main(int argc, char const *argv[])
{
  char* str = "HelloWorld!";
  char* p1 = &str[0];
  char* p2 = &str[5];

  printf("p2 - p1 = %ld\n", p2 - p1);
}
```

```
p2 - p1 = 5
```

其实就是2个 **地址值** 的相减.

### 2. 错误: 两个指针变量 相减

```c
#include <stdio.h>

struct person
{
  int age;
  char name[64];
  char sex;
};

int main(int argc, char const *argv[])
{
  struct person per = {99, "xiong", 'm'};
  struct person* p = &per;

  printf("p = %p\n", p);
  printf("p+1 = %p\n", p+1);
  printf("(p+1) - p = %ld\n", (p+1) - p);
}
```

```
p = 0x7ffee1bef060
p+1 = 0x7ffee1bef0a8
(p+1) - p = 1
```

- 1) 明显 0x7ffee1bef0a8 - 0x7ffee1bef060 = 0x48 = **72** 字节
- 2) 但是最终 **(p+1) - p** = **1** 字节 ??? 而不是 **72** 字节？

### 3. `(p+1) - p = 1` 等于 1 的原因

```c
(p+1) - p
```

最终编译器展开的表达式:

```c
(
  (起始地址 + sizeof(struct person)  * 1)  - 起始地址
) / sizeof(struct person)
```

- 1) 对于 **普通变量** 相减时，就是 **直接相减**

- 2) 但是 **指针变量** 相减时
  - 并 **不是** 直接相减
  - 而是会转换成 **内存地址** 规则计算 **差值**

### 4. 改成 ==2个== 指针变量, 仍然相减为 1

```c
#include <stdio.h>

struct person
{
  int age;
  char name[64];
  char sex;
};

int main(int argc, char const *argv[])
{
  struct person per = {99, "worinima", 'm'};
  
  // 改成2个指针变量，并明确2个指针变量的内存地址值
  struct person* p1 = &per;
  struct person* p2 = p1 + 1;


  printf("p1 = %p\n", p1);
  printf("p2 = %p\n", p2);
  printf("p2 - p1 = %ld\n", p2 - p1);
}
```

```
p1 = 0x7ffee4f0c060
p2 = 0x7ffee4f0c0a8
p2 - p1 = 1
```

- 实际上与上面的 **(p+1) - p** 没什么区别
- 只不过多了 **2个变量**

### 5. 如上两种方式计算得到 ==1(步数)== 的原因


- 最终计算结果就是 **1** 

- 注意此时 **1** 的度量 **单位** 并不是 **字节数**

- 我理解为 **步数** (指针 **走了几步**)

- 此时结果 **1** , 也就是说指针 **向后** 走了 **1步**

- 而这 **1步** 的 **长度(字节数)** 等于 **sizeof(struct person) 字节数** 

### 6. 指针1 - 指针2 = n(单位: ==步数==) , 并不是 字节数

```c
#include <stdio.h>

struct person
{
  int age;
  char name[64];
  char sex;
};

int main(int argc, char const *argv[])
{
  struct person per = {99, "worinima", 'm'};
  printf("sizeof(struct person) = %ld\n", sizeof(struct person));
  
  struct person* p = &per;

  // p 指向的 内存地址
  intptr_t addr1 = (intptr_t)p;
  printf("addr1 = %ld\n", addr1);

  // p+3 指向的 内存地址
  intptr_t addr2 = (intptr_t)(p + 3);
  printf("addr2 = %ld\n", addr2);

  // 两个内存地址值的差值
  printf("(addr2 - addr1) = %ld\n", (addr2 - addr1));

  // 计算 (p+1) - p 跳过的【步数】
  printf("(addr2 - addr1) / sizeof(struct person) => %ld / %ld = %ld\n", 
    (addr2 - addr1),
    sizeof(struct person),
    (addr2 - addr1) / sizeof(struct person)
  );
}
```

```
sizeof(struct person) = 72
addr1 = 140732663328864
addr2 = 140732663329080
(addr2 - addr1) = 216
(addr2 - addr1) / sizeof(struct person) => 216 / 72 = 3
```

### 7. 但是 ==sizeof(char)== 刚好等于 ==1个字节==

- 1) 指针变量1 - 指针变量2 = **步数**

- 2) 也就是说描述 两个指针变量 之间 **有多少(步)** , 而并不是 **有多少(字节数)**

- 3) 但是刚好 **sizeof(char)** 等于 **1(字节)** , 可以近似的转换为 **1(步)**

- 4) 所以使用 **char** 可以把 **1(步)** === **1(字节)** 进行等价

### 6. 通过 `char*` 类型的指针, 等价计算 两个指针 之间的 ==字节数==

```c
#include <stdio.h>

struct person
{
  int age;
  char name[64];
  char sex;
};

int main(int argc, char const *argv[])
{
  struct person per = {99, "worinima", 'm'};
  
  // 1、改成2个指针变量，并明确2个指针变量的内存地址值
  struct person* p1 = &per;
  struct person* p2 = p1 + 1;

  printf("p1 = %p\n", p1);
  printf("p2 = %p\n", p2);

  // 2、在执行 p2 - p1 之前，先分别将 p2、p1 强制转换为 char* 类型的指针
  printf("(char*)p2 - (char*)p1 = %ld\n", (char*)p2 - (char*)p1);
}
```

```
p1 = 0x7ffee0180060
p2 = 0x7ffee01800a8
(char*)p2 - (char*)p1 = 72
```

- 1) 此时已经能够正确计算出 p2 和 p1 之间的差值字节为 **72**
- 2) 但是这个 **72** 实际意义: **指针** 走了 **72(步)**
- 3) 其中 **每一步** 占据的长度为 **1字节**
- 4) 但是可以等价的认为 两个指针 之间 相差 **72(字节)**

### 8. 根本不需要 p2 第二个变量

```c
#include <stdio.h>

struct person
{
  int age;
  char name[64];
  char sex;
};

int main(int argc, char const *argv[])
{
  struct person per = {99, "worinima", 'm'};
  
  struct person* p = &per;


  printf("p = %p\n", p);
  printf("p+1 = %p\n", p+1);

  /**
   * 在执行 (p+1) - p 之前，先分别将 (p+1)、p 强制转换为 char* 类型
   */
  printf("(char*)(p+1) - (char*)p = %ld\n", (char*)(p+1) - (char*)p);
}
```

```
p = 0x7ffee76a1060
p+1 = 0x7ffee76a10a8
(char*)(p+1) - (char*)p = 72
```

一样可以得到差值字节数为 **72**

### 9. 而 `(char*)(p+1)` - `(char*)p` = 72 ?

```c
#include <stdio.h>

struct person
{
  int age;
  char name[64];
  char sex;
};

int main(int argc, char const *argv[])
{
  int a = 1;
  double b = 9.9;
  struct person c = {99, "worinima", 'm'};
  
  {
    printf("sizeof(int) = %ld\n", sizeof(int));
    int* p = &a;
    
    printf("p = %p\n", p);
    printf("p+3 = %p\n", p+3);
    printf("(p+3) - p = %ld\n", (p+3) - p);
    printf("(char*)(p+3) - (char*)p = %ld\n", (char*)(p+3) - (char*)p);
  }

  printf("-------------------------------------- \n");

  {
    printf("sizeof(double) = %ld\n", sizeof(double));
    double* p = &b;

    printf("p = %p\n", p);
    printf("p+3 = %p\n", p+3);
    printf("(p+3) - p = %ld\n", (p+3) - p);
    printf("(char*)(p+3) - (char*)p = %ld\n", (char*)(p+3) - (char*)p);
  }

  printf("-------------------------------------- \n");

  {
    printf("sizeof(struct person) = %ld\n", sizeof(struct person));
    struct person* p = &c;

    printf("p = %p\n", p);
    printf("p+3 = %p\n", p+3);
    printf("(p+3) - p = %ld\n", (p+3) - p);
    printf("(char*)(p+3) - (char*)p = %ld\n", (char*)(p+3) - (char*)p);
  }
}
```

```
sizeof(int) = 4
p = 0x7ffee664504c
p+3 = 0x7ffee6645058
(p+3) - p = 3
(char*)(p+3) - (char*)p = 12
--------------------------------------
sizeof(double) = 8
p = 0x7ffee6645040
p+3 = 0x7ffee6645058
(p+3) - p = 3
(char*)(p+3) - (char*)p = 24
--------------------------------------
sizeof(struct person) = 72
p = 0x7ffee6645060
p+3 = 0x7ffee6645138
(p+3) - p = 3
(char*)(p+3) - (char*)p = 216
```

- 1) 相同: **(p+1) - p** = **3** (单位: **步**)
- 2) 不同: **(char*)(p+1) - (char*)p** = 12、24、216 (单位: **字节数**)

所以 `(char*)(p+1) - (char*)p`:

- 1) 12 总字节数/sizeof(**char**) = 12
- 2) 24 总字节数/sizeof(**char**) = 24
- 3) 216 总字节数/sizeof(**char**) = 216

### 10. 总结

| 2个指针变量的相减             | 得到数值的 **单位** 含义 |
| -------------------------- | --------------------- |
| `(p+1) - p`                | **步数** (跳动的总字节数/sizeof(指向的类型)) |
| `(char*)(p+1) - (char*)p`  | **字节数**                               |



## 8. `void*` 指针变量 不能进行 数值运算(GNU C 允许)

- 1、对于 **标准c语言**，是进制对 `void*` 类型指针变量进行运算的
- 2、因为 `void*` 类型的指针变量
  - 1) 可以指向 **任意数据类型** 的内存
  - 2) 无法确定到底 **指向** 的是 **哪一种数据类型**
- 3、无法确定 **指向的类型** 带来的问题:
  - 1) 无法确定 **指针变量** 的 **步长**
  - 2) 也就无法知道到底要 **跳动多少个字节数**
- 4、但是在 **gnu c** 扩展中，确实可以进行运算的，跳动字节数为 **1个字节**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct man 
{
  int age;
  char name[10];
};

struct women 
{
  char name[20];
  long age;
};

struct animal 
{
  int favor;
  float level;
};

int main(int argc, const char * argv[]) 
{
  //1.
  printf("sizeof(struct man) = %ld\n", sizeof(struct man));
  printf("sizeof(struct women) = %ld\n", sizeof(struct women));
  printf("sizeof(struct animal) = %ld\n", sizeof(struct animal));

  //2.
  void* p1 = malloc(sizeof(struct man));
  void* p2 = malloc(sizeof(struct women));

  //3. 必须对void*强制转换，确定指针的步长，才能使用
  struct man* man_p1 = p1;
  man_p1->age = 19;
  strcpy(man_p1->name, "man001");
  printf("age = %d, name = %s\n", man_p1->age, man_p1->name);

  //4. 如果类型转换错误继续使用，则会访问到其他的内存块，可能导致程序崩溃
  struct women* man_p2 = p1; //类型转换错误，导致指针的步长不对
  printf("age = %ld, name = %s\n", man_p2->age, man_p2->name);
}
```

```
sizeof(struct man) = 16
sizeof(struct women) = 32
sizeof(struct animal) = 8
age = 19, name = man001
age = -4611686018427387904, name =
```

- 在`ANSI C`标准中，不允许对`void*`指针变量进行算术运算
- 但是在GNU中则允许，默认跳动1个字节
- 总而言之不要对`void*`指针变量进行数学运算



## 9. 两个指针 ==共享== 一份内存

### 1. 各自占据 一半内存

- 1、两个不同类型的 struct 实例，共用同一个内存空间
- 2、但是起始地址 **不** 重叠
- 3、一个在占用 **前半部分** 空间，另一个在占用 **后半部分** 空间

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct women
{
  int32_t did;
  char     name[64];
};

struct man
{
  int32_t pid;
  char     name[64];
  int32_t order;
};


struct man* get_man()
{
  //1. 分配总内存
  // => 分配 women 与 man 结构体长度之和的内存空间
  // => 使用 man 类型指针变量来指向分配的内存块
  struct man *p = (struct man *)malloc(sizeof(struct man) + sizeof(struct women));

  //2. 将内存前半段按照man类型进行填充
  p->pid = 111;
  strcpy(p->name, "man_01");
  p->order = 1;

  //3. 将内存后半段交给 women 类型使用
  // => p+1 往后跳动 sizeoif(struct man) 个字节的内存地址处
  // => 然后将往后挪动到的内存地址，交给 women 类型的指针变量来指向
  struct women *q = (struct women*)(p + 1);

  //4. 按照 women 类型来填充后半段内存
  q->did = 222;
  strcpy(q->name, "women_01");

  //5. 返回前半段内存填充类型 man 类型的地址
  return p;
}

int main(int argc, const char * argv[])
{
  //1.
  struct man *p = get_man();
  printf("man.pid     = %d\n",     p->pid);
  printf("man.name    = %s\n",     p->name);
  printf("man.order   = %d\n",     p->order);
  printf("\n");

  //2. 类型强转
  struct women* q = (struct women*)(p + 1);
  printf("women.did   = %d\n",     q->did);
  printf("women.name  = %s\n",     q->name);
  
  //3. 按照起始地址释放
  free(p);
}
```

```
➜  main make
gcc main.c
./a.out
man.pid     = 111
man.name    = man_01
man.order   = 1

women.did   = 222
women.name  = women_01
➜  main
```

### 2. 起始地址 ==重叠==

与上面的区别是: 两个不同类型的 struct 实例 的 **内存起始地址** 是 **一样** :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct women
{
  int32_t did;
  char name[64];
};

struct man
{
  ////////////// 1. 包含其他类型内存 //////////////
  struct women wm; // 是一个对象，并不是内存地址

  ////////////// 2. man自己的成员 ////////////////
  int32_t pid;
  char name[64];
  int32_t order;
};

struct man* get_man()
{
  //1. 分配总内存
  struct man *p = (struct man *)malloc(sizeof(struct man));

  //2. 填充前半段women内存
  p->wm.did = 111;
  strcpy(p->wm.name, "women");

  //3. 填充后半段man内存
  p->pid = 222;
  p->order = 1;
  strcpy(p->name, "man");

  //4.
  return p;
}

int main(int argc, const char * argv[])
{
  //1.
  struct man *p = get_man();
  printf("man.pid     = %d\n",     p->pid);
  printf("man.name    = %s\n",     p->name);
  printf("man.order   = %d\n",     p->order);
  printf("\n");

  //2. 类型强转
  struct women* q = (struct women*)p;
  printf("women.did   = %d\n",     q->did);
  printf("women.name  = %s\n",     q->name);

  //3. 同样释放内存起始地址
  free(p);
}
```

```
➜  main  make
gcc main.c
./a.out
man.pid     = 222
man.name    = man
man.order   = 1

women.did   = 111
women.name  = women
➜  main
```

