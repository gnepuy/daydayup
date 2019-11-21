[TOC]



## 1. ==C 语言== 中的 struct 内 ==只能== 定义 ==成员变量==

```c
struct person
{
  // 1. 基本类型 - 成员变量
  int age;
  
  // 2. 指针类型 - 成员变量
  int* p1;
  struct node* p2;

  // 3. 函数 指针类型 - 成员变量
  void (*p3)(int, int);
  
  // 4. 数组类型 - 成员变量
  char name[64];
  int* arr1[10];
  struct node* arr2[10];

  // 5. 函数指针 数组类型 - 成员变量
  void (*arr3[10])(int, int);
};
```

只有在 **C++** 的 struct 才允许定义 **函数** 类型的成员.



## 2. struct 实例 初始化

### 1. 成员变量 初始化

```c
#include <string.h>

struct stuff
{
  char job[20];  
  int age;  
  float height;  
};

int main()
{
  struct stuff ins;
  strcpy(ins.job, "manager");
  ins.age = 99;
  ins.height = 185.99;
}
```

### 2. `{val1, val2, val3}` 顺序初始化

```c
struct stuff
{
  char job[20];  
  int age;  
  float height;  
};

int main()
{
  struct stuff ins = {"manager", 30, 185};  
}
```

### 3. GNU/Clang `.ivar = ...` 乱序初始化

```c
struct stuff
{
  char job[20];  
  int age;  
  float height;  
};

int main()
{
  struct stuff ins = {
    .age = 99,
    .job = "worinima",
    .height = 189.343
  };
}
```

### 4. 如上方式在 struct ==可移植性== 比较

#### 1. struct 定义

```c
struct sembuf {
  unsigned short sem_num;
  short sem_op;
  short sem_flag;
};
```

#### 2. ==不可移植== 初始化方式

```c
struct sembuf buf = {3, -1, SEM_UNDO};
```

- 1、struct 成员变量的 **初始化顺序** 已经被 **固定**
- 2、一旦 **改变** struct sembuf 内部的成员 **定义顺序**、**个数**，都会导致上面初始化代码 **编译报错**

#### 3. ==可移植== 初始化方式 1

```c
struct sembuf buf;
buf.sem_op = 3;
buf.sem_flag = SEM_UNDO;
```

#### 4. ==可移植== 初始化方式 2（GNU/Clang）

```c
struct sembuf buf = {
  .sem_num = 3, 
  .sem_op = -1, 
  .sem_flag = SEM_UNDO
};
```



## 3. struct 应用

### 1. 同时做 ==传入== 与 ==传出==

```c
struct Context
{
  //1.【传入】参数集合: 值
  const char *username;
  const char *password;
  
  //2.【传出】参数集合: 地址
  int *isLogin;
};

// 传递指针，因为要修改内部成员变量值
void login(struct Context *ctx)
{
  // 1. 取出参数
  读取 ctx->username;
  读取 ctx->password;

  // 2. 执行逻辑
  //......
  
  // 3. 回传结果
  if (ok)
    *(ctx->isLogin) = 1;
  else
    *(ctx->isLogin) = 0;
}

int main()
{
  // 1. 保存调用 login() 函数的【返回值】
  int isLogin = 0;
  
  // 2.
  struct Context ctx = {
    "xiongzenghui", // 传入
    "123456",       // 传入
    &isLogin,       // 传出
  };
  
  //3.
  login(&ctx);

  //4. 
  if (isLogin) {
    // 登录
  } else {
    // 未登录
  }
}
```

### 2. 结构体 ==位域==

- 1) 把 **一个字节** 中的8个二进位，划分为几个 **不同的区域**
- 2) 让 每个 二进制位，用来表示 **不同的信息**

```c
#include <stdio.h>
#include <stdint.h>

#if 0
struct 位结构名
{
  数据类型 变量名: 占用二进制位长度;
  数据类型 变量名: 占用二进制位长度;
} 位结构变量;
#endif

// 【匿名】结构体
struct
{
  uint8_t a:2;  // 占用【0、1】两个二进制位，取值范围 00(0) ~ 11(3)
  uint8_t b:2;  // 占用【2、3】两个二进制位，取值范围 00(0) ~ 11(3) 
  uint8_t c:2;  // 占用【4、5】两个二进制位，取值范围 00(0) ~ 11(3) 
  uint8_t d:2;	// 占用【6、7】两个二进制位，取值范围 00(0) ~ 11(3) 
} bits; // 全局【匿名】实例 bits

int main()
{
  printf("sizeof(bits) = %ld字节\n", sizeof(bits));

  bits.a = 0; //00
  bits.b = 1; //01
  bits.c = 2; //10
  bits.d = 3; //11

  printf("struct bs 0~1: %d\n", bits.a);
  printf("struct bs 2~3: %d\n", bits.b);
  printf("struct bs 4~5: %d\n", bits.c);
  printf("struct bs 6~6: %d\n", bits.d);
}
```

```
➜  main make
gcc main.c
./a.out
sizeof(bits) = 1字节
struct bs 0~1: 0
struct bs 2~3: 1
struct bs 4~5: 2
struct bs 6~6: 3
➜  main
```

如上的 位域结构体 有一个比较 **麻烦** 的地方: 需要 **依次** 读写位域结构体的 **每一个成员值**

### 3. ==一次性== 读写 所有的 位域

#### 1. 1个字节 长度的 ==位域 struct==

```c
struct
{
#if 1
  uint8_t sleep:2;  // 占用0、1两个二进制位，mask=11 => 3
  uint8_t run:1;    // 占用2一个二进制位，mask=100 => 1<<2
  uint8_t eat:2;    // 占用3、4两个二进制位，mask=11000 => 3<<3
  uint8_t sing:1;	  // 占用5一个二进制位，mask=100000 => 1<<5
#else
  /** uint8_t => unsigned char */
  unsigned char sleep:2;
  unsigned char run:1;
  unsigned char eat:2;
  unsigned char sing:1;
#endif
} bitfield;
```

#### 2. 1个字节 长度的 uint

```c
/**
 * 用于快速获取 union 1字节 内存数据
 * 注意：bits 的内存长度 >= 位域结构体的总长度
 */
#if 1
  uint8_t bits;
#else
  /** uint8_t => unsigned char */
  unsigned char bits;
#endif
```

#### 3. 使用 ==union==  共享 ==位域 struct== 和 ==uint== 内存

```c
/**
 * 所有成员共用1个字节的内存
 * - 1）位域结构体
 * - 2）与位于结构体字节数相等的连续内存
 */
union Flag
{
  /**
   * 1、位域结构体实例，划分1个字节的8个二进制位，表示不同的含义
   */
  struct
  {
#if 1
    uint8_t sleep:2;  // 占用0、1两个二进制位，mask=11 => 3
    uint8_t run:1;    // 占用2一个二进制位，mask=100 => 1<<2
    uint8_t eat:2;    // 占用3、4两个二进制位，mask=11000 => 3<<3
    uint8_t sing:1;	  // 占用5一个二进制位，mask=100000 => 1<<5
#else
    /** uint8_t => unsigned char */
    unsigned char sleep:2;
    unsigned char run:1;
    unsigned char eat:2;
    unsigned char sing:1;
#endif
  } bitfield;

  /**
   * 2、用于快速获取union整个1字节的内存数据
   * 注意：bits的内存长度 >= 位域结构体的总长度
   */
#if 1
  uint8_t bits;
#else
  /** uint8_t => unsigned char */
  unsigned char bits;
#endif
};
```

#### 4. 完整示例

```c
#include <stdio.h>
#include <stdint.h>

union Flag
{
  struct
  {
    uint8_t sleep:2;  // 占用0、1两个二进制位，mask=11 => 0x3
    uint8_t run:1;    // 占用2一个二进制位，mask=100 => 1<<2
    uint8_t eat:2;    // 占用3、4两个二进制位，mask=11000 => 3<<3
    uint8_t sing:1;   // 占用5一个二进制位，mask=100000 => 1<<5
  } bitfield;

  uint8_t bits;
};


int main()
{
  printf("sizeof(union Flag) = %ld字节\n", sizeof(union Flag));
  union Flag flag;
  
  /**
   * 下面两种方式修改位域结构体的成员值
   * => 11,1110
   * sleep:10
   * run:1
   * eat:11
   * sing:1
   */


  /**
   * 1. 直接赋 1字节的 bits 内存 => 0011,1110
   */
  {
    flag.bits = 0;    // 清零
    flag.bits = 0x3e; // 二进制：11,1110
  }

  /**
   * 2. 逐个修改 位域结构体 成员值 => 0011,1110
   */
  {
    flag.bits = 0; // 清零
    flag.bitfield.sleep = 0x2;  // 10
    flag.bitfield.run = 0x1;    // 1
    flag.bitfield.eat = 0x3;    // 11
    flag.bitfield.sing = 0x1;   // 1
  }

  /**
   * 3. 统一打印每一个位域值
   */
  printf("flag.mask = %d\n",  flag.bits);
  printf("sleep\t = %d\n",    flag.bits & 0x3);
	printf("run\t = %d\n",      (flag.bits & (0x1<<2)) >> 2);
	printf("eat\t = %d\n",      (flag.bits & (0x3<<3)) >> 3);
  printf("sing\t = %d\n",     (flag.bits & (0x1<<5)) >> 5);
}
```

```
 ~/Desktop/main  clang main.c
 ~/Desktop/main  ./a.out
sizeof(union Flag) = 1字节
flag.mask = 62
sleep	 = 2
run	 = 1
eat	 = 3
sing	 = 1
```



## 4. struct 提供 ==工厂方法== 创建实例 : CGRectMake()

### 1. 内部 struct 定义

```c
struct rect
{
  int x;
  int z;
  int w;
  int h;
};
```

### 2. 提供 CGRectMake() 创建 struct 实例的 工厂方法

```c
struct rect
CGRectMake(int x, int y, int w, int h)
{
  return {1,2,3,4}; // 拼接构建内部需要的结构体实例
}
```

### 3. 结构体发生改变

```c
struct rect
{
  struct point point;
  int w;
  int h;
};
```

### 4. 只需要改变 CGRectMake() 工厂方法实现, ==外部调用者== 不需要改变

- 1) 因为 **外部调用者** 依赖的是 **CGRectMake()** 工厂方法 
- 2）而并不是 **struct rect** 具体的 结构体 定义

```c
struct rect
CGRectMake(int x, int y, int w, int h)
{
  return {{1,2},3,4};
}
```



## 5. C语言中的 ==封装==

### 1. 定义 ==class== 

```c
/**
 * 注意: C 语言中, 规定在 struct 内部, 只能定义【变量】类型的成员
 */
struct person
{
	// 1. 普通变量 - 成员
	int age;
	char name[64];

	/**
	 * 2. 函数指针变量 - 成员
	 * - 1) 都是一个个的【函数指针】，指向对应的【全局函数】
	 * - 2) 要求每一个【全局函数】必须包含【struct person *】这个【指针】类型参数，
	 * - 3) 用于在【被调用的函数】才能【知道】当前操作的是【哪个 person 对象
	 */
	int (*getAge)(struct person* this);
	void (*setAge)(struct person* this, int age);
	char* (*getName)(struct person* this);
	void (*setName)(struct person* this, char* name);
	void (*print_age)(struct person* this);
	void (*print_name)(struct person* this);
};
```

### 2. 实现外部 ==全局函数==

```c
/** 
 *	所有的对象方法必须至少存在一个参数，
 *  接收当前要操作的是哪一个 struct person 对象，
 *  然后在函数中读写传入的struct person 对象
 */

int getAge(struct person* this)
{
	return this->age;
}

void setAge(struct person* this, int age)
{
	this->age = age;
}

char* getName(struct person* this)
{
	return this->name;
}
void setName(struct person* this, char* name)
{
	strcpy(this->name, name);
}

void print_age(struct person* this)
{
	printf("age = %d\n", this->age);
}

void print_name(struct person* this)
{
	printf("name = %s\n", this->name);
}
```

### 3. 创建对象

#### 1. 栈区 对象

```c
struct person p = {
  /**
   * 成员变量
   */
  .age = 19,
  .name = "hahahah",
  /**
   * 成员函数指针变量
   */
  .getAge = getAge,
  .setAge = setAge,
  .getName = getName,
  .setName = setName,
  .print_age = print_age,
  .print_name = print_name
};
```

#### 2. 堆区 对象

```c
// 创建对象
struct person* p = (struct person*)malloc(sizeof(struct person));

//【重要】初始化struct实例的成员函数指针变量，指向对应的具体c函数
// 否则会报空指针错误，导致程序崩溃
p->getAge = getAge;
p->setAge = setAge;
p->getName = getName;
p->setName = setName;
p->print_age = print_age;
p->print_name = print_name;
```

### 4. 调用 对象方法

#### 1. 栈区 实例 方法调用

```c
//1. 
struct person p = ....;

//2.
p.print_age(&p);
p.print_name(&p);
```

#### 2. 堆区 实例 方法调用

```c
//1.
struct person* p = (struct person*)malloc(sizeof(struct person));
//....

//2.
p->setAge(p, 19);
p->setName(p, "hahahah");
p->print_age(p);
p->print_name(p);
```

### 5. 完整示例

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

///////////////////////// 1. 类 //////////////////////////

struct person 
{
  // 1.1 普通数据类型的对象成员
  int age;
  char name[64];

  /**
   *	1.2 【函数指针】类型的对象成员
   *	
   *	成员函数，必须包含【person对象指针】这个参数，
   *	被调用的函数，才能知道操作的是哪个person对象
   */
  int (*getAge)(struct person* this);
  void (*setAge)(struct person* this, int age);
  char* (*getName)(struct person* this);
  void (*setName)(struct person* this, char* name);
  void (*print_age)(struct person* this);
  void (*print_name)(struct person* this);
};

/////////////////// 2. 函数指针指向的具体c函数 //////////////////////

/** 
 *	所有的对象方法必须至少存在一个参数，
 *  接收当前要操作的是哪一个 struct person 对象，
 *  然后在函数中读写传入的struct person 对象
 */
int getAge(struct person* this)
{
  return this->age;
}

void setAge(struct person* this, int age)
{
  this->age = age;
}

char* getName(struct person* this)
{
  return this->name;
}
void setName(struct person* this, char* name)
{
  strcpy(this->name, name);
}

void print_age(struct person* this)
{
  printf("age = %d\n", this->age);
}

void print_name(struct person* this)
{
  printf("name = %s\n", this->name);
}
///////////////////////////////////////////////////////////

#if 1
int main()
{
  // 创建对象
  struct person p = {
    .age = 19,
    .name = "hahahah",
    .getAge = getAge,
    .setAge = setAge,
    .getName = getName,
    .setName = setName,
    .print_age = print_age,
    .print_name = print_name
  };

  // 调用对象的成员函数
  p.print_age(&p);
  p.print_name(&p);
}
#else
int main()
{
  // 创建对象
  struct person* p = (struct person*)malloc(sizeof(struct person));

  //【重要】初始化struct实例的成员函数指针变量，指向对应的具体c函数
  // 否则会报空指针错误，导致程序崩溃
  p->getAge = getAge;
  p->setAge = setAge;
  p->getName = getName;
  p->setName = setName;
  p->print_age = print_age;
  p->print_name = print_name;

  // 调用struct实例的成员函数指针变量指向的c函数
    // 从效果上开起来就好像调用成员函数似的
  p->setAge(p, 19);
  p->setName(p, "hahahah");
  p->print_age(p);
  p->print_name(p);

  //
  free(p);
}
#endif

```

c++的类的成员函数，也就是通过 **成员函数指针变量** 来实现的。



## 6. 封装进阶: 模拟 C++ class 中的 public、private 作用域

### 1. C++ class  具有 ==不同== 成员访问权限

```c++
QObject
{
public:
  aaa
  bbb
private:
  QObjectPrivate * priv;
};
```

### 2. C 中的 struct 只有一种【public】成员访问权限

```c
struct person 
{
  // 1、普通数据类型的对象成员
  int age;
  char name[64];

  /**
   *	2、【函数指针】类型的对象成员
   *	- 成员函数，必须包含【person对象指针】这个参数
   *	- 被调用的函数，才能知道操作的是哪个person对象
   */
  int (*getAge)(struct person* this);
  void (*setAge)(struct person* this, int age);
  char* (*getName)(struct person* this);
  void (*setName)(struct person* this, char* name);
  void (*print_age)(struct person* this);
  void (*print_name)(struct person* this);
};
```

只要是定义在 C struct 中的 成员, 就都可以被外界方法

### 3. qt 让 C 也具有 private 效果

#### 1. 向外暴露的 st_qobject.h 头文件

```c
// 1、【私有】类型的 struct 只做【向前声明】，并不给出【具体定义】
// 只是 forward declaration of 'struct st_qobject_private'
// 如果【不想暴露】公有 struct 中的某个【成员的定义】, 那么就使用【forward declaration】这个成员
struct st_qobject_private;

// 2、【公有】类型的 struct 可以暴露【具体定义】
struct st_qobject
{
  /**
   * public 【变量】成员
   */ 
  int aaa;
  int bbb;

  /**
   * public 【函数指针】成员
   */ 
  void (*func)(struct st_qobject*);

  /**
   * private 【变量】成员
   */
  struct st_qobject_private* root;
};

/**
 * 创建 struct st_qobject 对象的 工厂方法
 */
struct st_qobject* get_qo_object_ins();
void free_qo_object_ins(struct st_qobject* qo);
```

#### 2. st_qobject.c

```c
#include <stdio.h>
#include <stdlib.h>

// 包含【自己】声明
#include "st_qobject.h"

// 包含【私有类型】声明
#include "st_qobject_private.h"

void st_qobject_func_impl(struct st_qobject* qo)
{
  printf("aaa = %d\n", qo->aaa);
  printf("bbb = %d\n", qo->bbb);
}

struct st_qobject* get_qo_object_ins()
{
  // 1.
  struct st_qobject* p = malloc(sizeof(struct st_qobject*));

  // 2. 公有 成员
  p->aaa = 1;
  p->aaa = 2;

  // 3. 公有 成员
  p->func = st_qobject_func_impl;


  // 4. 私有 成员
  p->root = malloc(sizeof(struct st_qobject_private*));
  p->root->data = 0;
  p->root->prev = p->root->next = NULL;

  return p;
}

void free_qo_object_ins(struct st_qobject* qo)
{
  // 1.
  struct st_qobject_private* root = qo->root;
  while (root)
  {
    root = root->next;
    free(root);
  }
  

  // 2.
  free(qo);
}
```

#### 3. ==私有== 类型 st_qobject_private.h 结构体定义

```c
struct st_qobject_private
{
  struct st_qobject_private* prev, *next;
  int data;
};
```

#### 4. main.c

```c
#include <stdio.h>
#include "st_qobject.h"

int main(int argc, char const *argv[])
{
  // 1.
  struct st_qobject* qoobject = get_qo_object_ins();

  // 2.
  qoobject->aaa = 222;
  qoobject->bbb = 333;
  
  // 3.
  qoobject->func(qoobject);

  // 4. 如下操作 qoobject->root 都是【编译报错】的
  // qoobject->root->next = NULL;
  // qoobject->root->prev = NULL;
  // qoobject->root->data = 2;

  // 5.
  free_qo_object_ins(qoobject);
}
```

### 4. 总结

- 如果【不想暴露】公有 struct 中的某个【成员的定义】
- 第一步: 在 **公有类型.h** 使用 **forward declaration** **隐藏** 这个成员的 **具体定义**
- 第二步: 在 **公有类型.c** 再使用 `#include "xxx_private.h"` 导入 **私有类型** 的 **具体定义**



## 7. 继承

### 1. C语言实现 ==继承== 核心

#### 1. 父类

```c
struct st_base
{
  xxx;
};
```

#### 2. 子类

```c
struct st_derived
{
  /**
   * 实现【继承】的【重点】
   * - 1) 一定要在把【父类】的【对象】放在【子类成员】列表中的【第一位】
   * - 2) 一定是【父类】的【对象】，而【不是】【指针】
   */
  struct sb_base base;

  /**
   * 【子类】自己的成员
   */
  yyy;
};
```

### 2. 子类 调用 ==父类成员==

```c
// 子类对象
struct st_derived child;
// 子类对象通过【成员变量 - 父类对象】调用父类的成员
child.base.父类成员
```

### 3. 示例

#### 1. 父类

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

///////////////////////// 父类定义 //////////////////////////
struct person 
{
	// 1.1 成员变量
	int age;
	char name[64];
  
	// 1.2 成员函数指针
	int (*getAge)(struct person* this);
	void (*setAge)(struct person* this, int age);
	char* (*getName)(struct person* this);
	void (*setName)(struct person* this, char* name);
	void (*print_age)(struct person* this);
	void (*print_name)(struct person* this);
};

///////////////// 父类对象的成员函数指针指向的具体函数 ////////////////////
int getAge(struct person* this)
{
	return this->age;
}

void setAge(struct person* this, int age)
{
	this->age = age;
}

char* getName(struct person* this)
{
	return this->name;
}
void setName(struct person* this, char* name)
{
	strcpy(this->name, name);
}

void print_age(struct person* this)
{
	printf("age = %d\n", this->age);
}

void print_name(struct person* this)
{
	printf("name = %s\n", this->name);
}
```

#### 2. 子类

```c
///////////////////////// 子类1 //////////////////////////
struct man 
{
	//【重要】在第一个成员位置包含父类对象
	struct person person;

	// 子类对象成员
	int height;
};

///////////////////////// 子类2 //////////////////////////
struct women 
{
	// 【重要】在第一个成员位置包含父类对象
	struct person person;

	// 子类对象成员
	int weight;
};
```

#### 3. 测试

```c
int main()
{
	//1 创建子类对象
	struct man* m1 = (struct man*)malloc(sizeof(struct man));
	struct women* w1 = (struct women*)malloc(sizeof(struct women));

	//2 子类对象1的成员函数指针初始化
	(m1->person).getAge = getAge;
	(m1->person).setAge = setAge;
	(m1->person).getName = getName;
	(m1->person).setName = setName;
	(m1->person).print_age = print_age;
	(m1->person).print_name = print_name;

	//3 子类对象2的成员函数指针初始化
	(w1->person).getAge = getAge;
	(w1->person).setAge = setAge;
	(w1->person).getName = getName;
	(w1->person).setName = setName;
	(w1->person).print_age = print_age;
	(w1->person).print_name = print_name;

	//4 子类对象1成员赋值
	{
		// 执行包含结构体实例的成员函数指针，完对父类结构体实例成员的赋值
		(m1->person).setAge(&(m1->person), 19);
		(m1->person).setName(&(m1->person), "hahahah");
		(m1->person).print_age(&(m1->person));
		(m1->person).print_name(&(m1->person));

		// 对自己成员赋值
		m1->height=199;
	}
  
  	//5 子类对象2成员赋值
	{
		// 同上
		(w1->person).setAge(&(w1->person), 19);
		(w1->person).setName(&(w1->person), "hahahah");
		(w1->person).print_age(&(w1->person));
		(w1->person).print_name(&(w1->person));

		// 对自己成员赋值
		w1->weight=110;
	}
  
  	//6
  	free(m1);
  	free(w1);
}
```



## 8. 版本兼容

### 1. 包含【老版本】struct【成员】

```c
#include <stdio.h>

// 版本1的结构体定义
struct objc_method
{
  char* sel;
  void(*IMP)(void);
};

// 版本2的结构体定义
struct method_t
{
  // 1. 保留版本1结构体的成员
  char* sel;
  void(*IMP)(void);

  // 2. 版本2结构体的新增成员
  char buf[1024];
};

int main()
{
  // 1.【版本2】的结构体实例
  struct method_t m = {
    .sel = "hello",
    .IMP = (void*)0x1111,
    .buf = "worinima"
  };
  printf("sel: %s, IMP: %p\n", m.sel, m.IMP);

  // 2. 使用【版本1】结构体类型的指针，访问【版本2】结构体实例的成员
  struct objc_method* ptr = (struct objc_method*)&m;
  printf("sel: %s, IMP: %p\n", ptr->sel, ptr->IMP);

  // 3.【版本2】结构体新增的成员，只能通过【版本2】的类型访问
  printf("buf = %s\n", m.buf);
}
```

```
➜  main make
gcc main.c
./a.out
sel: hello, IMP: 0x1111
sel: hello, IMP: 0x1111
buf = worinima
➜  main
```

### 2. 包含【老版本】struct【实例】

```c
#include <stdio.h>

// 版本1的结构体定义
struct objc_method
{
  char* sel;
  void(*IMP)(void);
};

// 版本2的结构体定义
struct method_t
{
  // 1. 包含一个版本1结构体的实例
  struct objc_method old;

  // 2. 版本2结构体的新增成员
  char buf[1024];
};

int main()
{
  // 1. 版本2的结构体实例
  struct method_t m = {
    {
      .sel = "hello",
      .IMP = (void*)0x1111
    },
    .buf = "worinima"
  };
  printf("sel: %s, IMP: %p, buf = %s\n", m.old.sel, m.old.IMP, m.buf);
}
```

```
➜  main make
gcc main.c
./a.out
sel: hello, IMP: 0x1111, buf = worinima
➜  main
```
