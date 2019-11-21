[TOC]



## 1. 普通 指针变量

```c
int main() {
  int age = 99;
  int*   p1 = &age;
  int**  p2 = &p1;
  int*** p3 = &p2;
}
```



## 2. 数组 ==指针== (重点: 指针)

```c
int main()
{    
  // 1、数组
  int arr[10] = {0};

  // 2、指向数组第一个元素的指针
  int* p = arr;
}
```



## 3. 指针 ==数组== (重点: 数组)

### 1. ==栈区== 指针 数组

```c
int main()
{
  // 1、数组元素类型为【指针变量】
  int* arr[10] = {0};
  
  // 2、每一个数组元素（指针变量）分配指向一块堆区内存
  for(int i=0; i<10; i++)
  {
    arr[i] = malloc(sizeof(int));
    free(arr[i]);
  }
  
  // 3、字符串指针变量数组
  char *arr2[3] = {"1", "123", "345"};
}
```

### 2. ==堆区== 指针 数组

```c
....
```



## 4. 函数 ==指针== (重点: 指针)

```c
int func(int age)
{
  printf("age = %d\n", age);
  return 199;
}

int main()
{ 
  // 1、定义函数指针，指向func()函数的起始内存地址
  int (*funcp)(int) = func;

  // 2、通过函数指针，调用func()
  printf("funcp(99) = %d\n", funcp(99));
}
```

```
➜  main make
gcc main.c
./a.out
age = 99
funcp(99) = 199
➜  main
```



## 5. 函数 指针 ==数组== (重点: 数组)

```c
#include <stdio.h>

int sub(int a, int b)
{  
  printf("sub = %d\n",a-b);
  return 0;
}

int add(int a, int b)
{
  printf("add = %d\n",a+b);
  return 0;
}
int main()
{
  // 1、定义一个 函数指针数组
  // => 数组元素 => 函数指针类型
  // => 其调用返回值为int，参数为(int,int)
  int (*p[2])(int , int);

  // 2、
  p[0] = sub;
  p[1] = add;  

  // 3、
  p[1](10,3);
  p[0](10,3);
}
```



## 6. 指针函数 : ==返回值== 为 ==指针== 类型的 ==函数==

```c
char *func(void)
{
  char *str = "test";
  return str;
}

int main()
{
 	char *test = func();  
  printf("%s\n",test); 
}
```



## 7. 结构体 指针

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person
{
  int pid;
  char name[64];// 64字节
  struct person* child;// 8字节
};

int main(int argc, const char * argv[])
{
  struct person per = {19, "haha", NULL};
  struct person* p = &per; // 结构体 指针
}
```

