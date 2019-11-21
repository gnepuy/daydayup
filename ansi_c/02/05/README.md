[TOC]



## 1. 指针 : 函数的 ==传入== 形参

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int pid;
  char name[64];
  struct person* child;
};

void person_print(const struct person* origin) // const T *p 传入指针
{
  printf("%s\n", origin->name);
}

int main()
{
  struct person p = {99, "haha", NULL};
  person_print(&p);
}
```



## 2. 指针 : 函数的 ==传出== 形参

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int pid;
  char name[64];
  struct person* child;
};

void person_create(struct person** p) // 使用【二级指针】接收【一级指针】的【地址值】
{
  *p = (void*)malloc(sizeof(struct person));
  (*p)->pid = 1;
  strcpy((*p)->name, "xiongzenghui");
  (*p)->child = NULL;
}

int main()
{
  struct person* p;
  person_create(&p);
  printf("name = %s\n", p->name);
  free(p);
}
```

```
 ~/Desktop/main  gcc main.c
 ~/Desktop/main  ./a.out
name = xiongzenghui
```



## 3. ==值== 传递

### 1. 基本类型 的变量

```c
#include <stdio.h>
#include <stdlib.h>

void func(int a) 
{
  a += 1;
  printf("【func】&a = %p, a = %d\n", &a, a);
}

int main() 
{
  //1. 
  int a = 10;
  printf("【main 1】&a = %p, a = %d\n", &a, a);
  
  //2. 
  func(a);
  printf("【main 2】&a = %p, a = %d\n", &a, a);
}
```

```
->  make
gcc main.c
->  ./a.out
【main 1】&a = 0x7fff55f00a0c, a = 10
【func】&a = 0x7fff55f009ec, a = 11
【main 2】&a = 0x7fff55f00a0c, a = 10
->
```

- (1) `main` 函数中的 变量a 与 `func` 函数中 变量a，标记的是 **不同内存块**
- (2) 所以在 `func` 函数内, 无法修改 `main` 函数中的 变量a 对应内存块中的值

### 2. 等价于 函数内的 ==局部变量==

```c
#include <stdio.h>
#include <stdlib.h>

void func1(int a)
{
  a += 1;
  printf("【func1】&a = %p, a = %d\n", &a, a);
}

void func2()
{
  int a = 10;
  a += 1;
  printf("【func2】&a = %p, a = %d\n", &a, a);
}
```

- func1() 与 func2() 使用的内存是 **一样** 的
- 因为所有的 **函数调用栈帧** 都是利用同一个 **栈结构**
- **系统栈** 不断的分配、不断的擦除，重复使用

### 3. struct 实例

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int 	age;        // 4b
  char 	name[64];  	// 64b
};

void func(struct person per) 
{
  per.age = 20;
  strcpy(per.name, "2222");
  printf("【func】&per = %p, per.age = %d, per.name = %s\n", &per, per.age, per.name);
}

int main(int argc, const char * argv[]) 
{
  // 栈上分配内存
  struct person per = {19, "1111"};
  printf("【main 1】&per = %p, per.age = %d, per.name = %s\n", &per, per.age, per.name);
  
  // 调用函数，传入struct实例，而不是struct示例的内存地址
  func(per);
  printf("【main 2】&per = %p, per.age = %d, per.name = %s\n", &per, per.age, per.name);
}
```

```
->  make
gcc main.c
./a.out
【main 1】&per = 0x7fff5e22d9b0, per.age = 19, per.name = 1111
【func】&per = 0x7fff5e22d920, per.age = 20, per.name = 2222
【main 2】&per = 0x7fff5e22d9b0, per.age = 19, per.name = 1111
->
```

- 同样可以看到 `func()` 内部的 per对象内存地址 与 `main()` 内部的 per对象 **内存地址不一样**
- 但是 struct实例 的实例变量的值是一样的，这是 **浅拷贝** 的原因
- 所以同样在 `func()` 内也 **无法** 修改 `main()` 内部的 per对象 的成员数据进行修改

### 4. 一级 指针变量

> 核心：【指针变量】与【指针变量的值】，这是两个完全不同的概念。

#### 1. 基本类型

```c
#include <stdio.h>
#include <stdlib.h>

void func(int* p) 
{
  printf("【func】&p = %p, p = %p\n", &p, p);
}

int main() 
{
  int a = 10;
  int* p = &a;
  printf("【main 1】&p = %p, p = %p\n", &p, p);
  
  func(p);
  printf("【main 2】&p = %p, p = %p\n", &p, p);
}
```

```
->  make
gcc main.c
./a.out
【main 1】&p = 0x7fff59bc99f0, p = 0x7fff59bc99fc
【func】&p = 0x7fff59bc99c8, p = 0x7fff59bc99fc
【main 2】&p = 0x7fff59bc99f0, p = 0x7fff59bc99fc
->
```

- 1、两个指针变量的 **值** 是 **一样** 的
- 2、两个指针变量的 **内存地址** 是 **不一样** 的

#### 2. struct 类型

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int	age;       // 4b
  char 	name[64];  // 64b
};

void func(struct person* per)
{
  //1.
  printf("【func 1】per = %p, &per = %p, per->age = %d, per->name = %s\n", per, &per, per->age, per->name);

  //2. 修改形参指针变量的值，重新让指针【指向】其他的内存块
  per = (struct person*)malloc(sizeof(struct person));
  per->age = 20;
  strcpy(per->name, "2222");

  //3. 
  printf("【func 2】per = %p, &per = %p, per->age = %d, per->name = %s\n", per, &per, per->age, per->name);
}

int main(int argc, const char * argv[]) 
{
  //1. 赋值指针变量的值，让指针指向一块内存
  struct person *per = (struct person*)malloc(sizeof(struct person));

  // 2. 修改指针变量指向的内存块中的数据
  per->age = 19;
  strcpy(per->name, "1111");
  printf("【main 1】per = %p, &per = %p, per->age = %d, per->name = %s\n", per, &per, per->age, per->name);
  
  // 3. 将指针传入函数，试图修改指针的指向
  func(per);
  printf("【main 2】per = %p, &per = %p, per->age = %d, per->name = %s\n", per, &per, per->age, per->name);
}
```

```
->  make
gcc main.c
./a.out
【main 1】per = 0x7f98f0403200, &per = 0x7fff568279e8, per->age = 19, per->name = 1111
【func 1】per = 0x7f98f0403200, &per = 0x7fff568279b8, per->age = 19, per->name = 1111
【func 2】per = 0x7f98f0403250, &per = 0x7fff568279b8, per->age = 20, per->name = 2222
【main 2】per = 0x7f98f0403200, &per = 0x7fff568279e8, per->age = 19, per->name = 1111
->
```

- 两个指针变量的 **值** 是 **一样** 的
- 两个知者你变了的 **内存地址** 是 **不一样** 的
- 所以在 **func()** 内部无法修改 **main()** 内部的指针变量实参的值



## 4. ==地址== 传递

### 1. 基本数据类型变量, 必须显示 ==取地址== 后传递

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void func(int *a)
{
  *a += 1;
  printf("【func】a = %p, *a = %d\n", a, *a);
}

int main(int argc, const char * argv[])
{
  //1.
  int a = 10;
  printf("【main 1】&a = %p, a = %d\n", &a, a);

  //2.
  func(&a);
  printf("【main 2】&a = %p, a = %d\n", &a, a);
}
```

```
【main 1】&a = 0x7fff5fbff7fc, a = 10
【func】a = 0x7fff5fbff7fc, *a = 11
【main 2】&a = 0x7fff5fbff7fc, a = 11
```

通过`&a`将变量a在栈上对应的内存起始地址，传递给了被调用函数`func()`。

### 2. 但是 ==数组名== 默认就是 ==地址== 传递，==不需要== 取地址

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int 	age;       // 4b
  char 	name[64];  // 64b
};

void func1(struct person pers[])
{
  for (int i = 0; i < 4; i++)
    printf("age = %d, name = %s\n", pers[i].age, pers[i].name);
}

/**
*	数组当做形参时，会退化为指针变量
*/
void func2(struct person *p)
{
  for (int i = 0; i < 4; i++)
  {
    printf("age = %d, name = %s\n", p->age, p->name);
    p++;
  }
}

int main(int argc, const char * argv[])
{
  //1.
  struct person per[4] = {
    {19, "1111"},
    {20, "2222"},
    {21, "3333"},
    {22, "4444"}
  };

  //2.
  func1(per);
  printf("\n");
  func2(per);
}
```

```
->  make
gcc main.c
./a.out
age = 19, name = 1111
age = 20, name = 2222
age = 21, name = 3333
age = 22, name = 4444

age = 19, name = 1111
age = 20, name = 2222
age = 21, name = 3333
age = 22, name = 4444
->
```

### 3. 栈区 ==struct 实例== 同样需要 ==取地址== 后传递

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int 	age;        // 4b
  char 	name[64];  	// 64b
};

void func(struct person* p) 
{
  p->age = 20;
  strcpy(p->name, "2222");
}

int main(int argc, const char * argv[]) 
{
  // 栈上分配内存
  struct person per = {19, "1111"};
  printf("【main 1】&per = %p, per.age = %d, per.name = %s\n", &per, per.age, per.name);
  
  // 调用函数，传入struct实例的内存地址
  func(&per);
  printf("【main 2】&per = %p, per.age = %d, per.name = %s\n", &per, per.age, per.name);
}
```

```
->  make
gcc main.c
./a.out
【main 1】&per = 0x7fff5299c9b0, per.age = 19, per.name = 1111
【main 2】&per = 0x7fff5299c9b0, per.age = 20, per.name = 2222
->
```



##  5. ==一级== 指针变量 ==地址传递==

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int		age; 	
  char 	name[64]; 	
};

/**
 * 【二级指针变量】才能接收一级指针变量的【内存地址】
 */
void func(struct person** per)
{
  *per = (struct person*)malloc(sizeof(struct person));
}

int main()
{
  //1. 赋值指针变量的值，让指针指向一块内存
  struct person *per = (struct person*)malloc(sizeof(struct person));
  printf("【main 1】&per = %p, per = %p\n", &per, per);

  // 2. 将指针传入函数，试图修改指针的指向
  func(&per);
  printf("【main 2】&per = %p, per = %p\n", &per, per); 
}
```

```
->  make
gcc main.c
./a.out
【main 1】&per = 0x7fff5cede9f8, per = 0x7fc8e8403200
【main 2】&per = 0x7fff5cede9f8, per = 0x7fc8e8403250
->
```

- 指针变量per的 **内存地址** 等于 **0x7fff5cede9f8**
- 指针变量per的 **值** 最开始等于 **0x7fc8e8403200**
- 指针变量per的 **值** 后来变成 **0x7fc8e8403250**



## 6. ==n级== 指针变量 ==地址传递==

### 1. 函数形参定义的规则

使用 ==n+1级== 指针变量 作为 函数形参

```
1. 用 一级 指针变量 作为形参，去间接修改 0级指针（普通变量） 实参变量的值
2. 用 二级 指针变量 作为形参，去间接修改 一级指针 实参变量的值
3. 用 三级 指针变量 作为形参，去间接修改 二级指针 实参变量的值
4. 用 四级 指针变量 作为形参，去间接修改 三级指针 实参变量的值
5. .......................................
6. 用 n+1级 指针变量 作为形参，去间接修改 n级指针 实参变量的值
```

### 2. 示例

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void func1(int* p)
{}

void func2(int** p)
{}

void func3(int*** p)
{}

void func4(int***** p)
{}

void func5(int****** p)
{}

void func6(int******* p)
{}

int main(int argc, const char * argv[]) 
{
  //1. 
  int a = 1;
  func1(&a);

  //2.
  int* p1 = &a;
  func2(&p1);

  //3.
  int** p2 = &p1;
  func3(&p2);

  //4.
  int*** p3 = &p2;
  func4(&p3);

  //5.
  int**** p4 = &p3;
  func5(&p4);

  //6.
  int***** p5 = &p4;
  func6(&p5);
}
```



##  7. 函数指针: 指向一个函数的 指针变量

```c
#include <stdio.h>
#include <stdlib.h>

// 函数指针类型定义
typedef int (*IMP)(int x, char *s);

// 具体函数实现
int work(int x, char *s) {
  printf("--- work() ---\n");
  return 1;
}

int main() 
{
  //1. 指向函数的内存地址
  IMP imp = work;

  //2. 调用的直接形式
  int ret1 = imp(19, "Hello World!");

  //3. 函数指针类型强转后的完整形式
  int ret2 = ((int (*)(int,char*))imp)(19, "Hello World!");
}
```

```
->  make
gcc main.c
./a.out
--- work() ---
--- work() ---
->
```



## 8. 函数指针的 ==比较==

```c
#include <stdio.h>
#include <stdlib.h>

// 函数指针类型定义
typedef int (*IMP)(int x, char *s);

// 具体函数实现
int work(int x, char *s) {
  printf("--- work() ---\n");
  return 1;
}

int main() 
{
  //1. 指向函数的内存地址
  IMP imp = work;

  // 2.
  if (imp == work)
    printf("imp == work\n");
  else 
    printf("imp != work\n");
}
```

```
->  make
gcc main.c
./a.out
imp == work
->
```



## 9. 函数指针的 ==类型强转==

### 1. 强制转换的格式

```
((函数原型)函数指针变量)(形参1, 形参2, ...., 形参n);
```

### 2. 具体实例

```c
#include <stdio.h>
#include <stdlib.h>

void work1(void *self, char *sel)
{
  printf("%s\n", "work1");
}

int work2(void *self, char *sel ,int x) 
{
  printf("%s\n", "work2");
  return 1;
}

long work3(void *self, char *sel ,int x, int y) 
{
  printf("%s\n", "work3");
  return 111;
}

char* work4(void *self, char *sel ,int x, int y, int z) 
{
  printf("%s\n", "work4");
  return "Hello world~!!";
}

int main(int argc, const char * argv[]) 
{
  void* ptr;
  char* sel = "sel";

  //1.
  void* func;

  //2. 
  func = work1;
  ((void (*)(void *, char *))func)(ptr, sel);

  //3.
  func = work2;
  int ret1 = ((int (*)(void *, char *, int))func)(ptr, sel, 19);

  //4.
  func = work3;
  long ret2 = ((long (*)(void *, char *, int, int))func)(ptr, sel, 19, 20);

  //5.
  func = work4;
  char* ret3 = ((char* (*)(void *, char *, int, int, int))func)(ptr, sel, 19, 20, 21);
}
```

编译运行

```
work1
work2
work3
work4
```



## 10. 函数指针, 类型强转错误, 会导致崩溃

### 1. ==不使用== 参数 没有问题

```c
#include <stdio.h>

int work(int x, char *s) {
  // printf("x = %d, s = %s\n", x, s); // 不使用参数
  return 1;
}

int main() 
{
  ((int (*)(char*, int)) work)("haha", 10);
}
```

```
->  make
gcc main.c
./a.out
->
```

### 2. ==使用参数== 会导致崩溃（涉及 ==栈== 访问）

```c
#include <stdio.h>

int work(int x, char *s) {
  printf("x = %d, s = %s\n", x, s);
  return 1;
}

int main() 
{
  ((int (*)(char*, int)) work)("haha", 10);
}
```

```
->  make
gcc main.c
./a.out
make: *** [all] Segmentation fault: 11
->
```



## 11. 对 ==函数名== 取地址

### 1. `func、&func、*func` 值都是一样的

```c
#include <stdlib.h>
#include <stdio.h>

void fun(void)
{}

int main(void)
{
  printf("%p\n", fun);      // 函数名
  printf("%p\n", &fun);     // 函数名 取地址
  printf("%p\n", *fun);     // 函数名 解引用(解地址)
}
```

```
➜  main make
gcc main.c
./a.out
0x106c3bef0
0x106c3bef0
0x106c3bef0
➜  main
```

可以看到`func、&func、*func`得到的地址值，**都是一样** 的。

### 2. func 与 &func 

| 对象 | 代表的含义 | 数据的类型 |
|:--------|:--------|:------------|
| func | 函数名, 即: 函数的起始 **地址值** | `void ()` |
| &func | **指向** 函数起始地址的 **指针变量** | `void (*)()` |

类似 **arr** 与 `&arr` 关系.

### 3. 示例

```c
#include <string.h>
#include <stdio.h>

void func()
{
  printf("worinima\n");
}

int main()
{
  // 1、
  void (*funcp1)() = func;
  funcp1();

  // 2、
  void (*funcp2)() = &func;
  funcp2();

  // 3、
  void (*funcp3)() = &func;
  (*funcp3)();
}
```

```
➜  main make
gcc main.c
./a.out
worinima
worinima
worinima
➜  main
```

效果一样的.

