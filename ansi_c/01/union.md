[TOC]



## 1、union 结构体长度 = 最大成员变量的长度

```c
#include <stdlib.h>
#include <stdio.h>

union Person
{
	char a;		//1字节
	int  b;		//4字节
	char *c;	//8字节
	double d;	//8字节
};

int main()
{
	printf("%lu\n", sizeof(union Person));
}
```

```
8
```

- 所以可能造成**数据覆盖**的问题
- 只适用于成员变量的数据，肯定**不会同时**使用的时候，进行数据的**压缩**



## 2、union 存在【覆盖】的问题

```c
#include <stdio.h>
#include <string.h>
 
union Data
{
  int 	age;
  float score;
  char  name[20];
};
 
int main( )
{ 
	// 1、
  union Data data;        

	// 2、修改union的age成员
  data.age = 10;
  printf("------------------------------\n");
  printf("data.age : %d\n", data.age);
  printf("data.score : %f\n", data.score);
  printf("data.name : %s", data.name);

	// 3、修改union的score成员
  data.score = 220.5;
  printf("------------------------------\n");
  printf("data.age : %d\n", data.age);
  printf("data.score : %f\n", data.score);
  printf("data.name : %s\n", data.name);
	
	// 4、修改union的name成员
  strcpy(data.name, "C Programming");
  printf("------------------------------\n");
  printf("data.age : %d\n", data.age);
  printf("data.score : %f\n", data.score);
  printf("data.name : %s\n", data.name);
}
```

```
➜  main make
gcc main.c
./a.out
------------------------------
data.age : 10
data.score : 0.000000
data.name :
------------------------------
data.age : 1130135552
data.score : 220.500000
data.name :
------------------------------
data.age : 1917853763
data.score : 4122360580327794860452759994368.000000
data.name : C Programming
➜  main
```

- 1、因为union内部所有的成员，都使用【同一块内存】，所以会覆盖其他成员的数据
- 2、当修改unin其中一个成员时，就会【覆盖】掉之前成员的数据值
- 总结：**最后赋值**的成员变量的内存空间一定是完整的




## 3、每次使用union成员之前，必须对成员【重新赋值】

```c
#include <stdio.h>
#include <string.h>
 
union Data
{
  int 	age;
  float score;
  char  name[20];
};
 
int main( )
{ 
  // 1. 类似于创建一个struct实例
  union Data data;        

  // 2. 重新赋值age成员，再使用age成员
  data.age = 10;
  printf( "data.age : %d\n", data.age);

  // 3. 重新赋值score成员，再使用score成员
  data.score = 220.5;
  printf( "data.score : %f\n", data.score);

  // 4.重新赋值name成员，再使用name成员
  strcpy( data.name, "C Programming");
  printf( "data.name : %s\n", data.name);
}
```

```
->  gcc main.c
->  ./a.out
data.age : 10
data.score : 220.500000
data.name : C Programming
->
```



## 4、union 成员, ==共享== 同一块内存

```c
#include <stdlib.h>
#include <stdio.h>

// union占用长度为长度最大的成员占用的长度，所以为4字节
union Flag
{
  char bits[4]; // 4字节
  int  flags;   // 4字节
}cc;

int main()
{
  //1.
  printf("sizeof(union Flag) = %lu\n", sizeof(union Flag));

  //2. 填充char a[4]数组的4个字节，每个字节使用二个十六进制数填充
  cc.bits[0] = 0x31; // 填充第一个字节(最低位)		 |
  cc.bits[1] = 0x32; // 填充第二个字节						|
  cc.bits[2] = 0x33; // 填充第三个字节					\	| /
  cc.bits[3] = 0x34; // 填充第四个字节(最高位)		\|/

  //3.
  printf("cc.bits = ");
  printf("%X", cc.bits[0]);
  printf("%X", cc.bits[1]);
  printf("%X", cc.bits[2]);
  printf("%X", cc.bits[3]);
  printf("\n");
  printf("cc.flags = 0x%x\n", cc.flags);
}
```

```
➜  main make
gcc main.c
./a.out
sizeof(union Flag) = 4
cc.bits = 31323334
cc.flags = 0x34333231
➜  main
```


## 5、union 压缩, 使用同一份内存

```c
union mem
{
  //1. 类型1包的内存缓存区
  char buff[1024];
  
  //2. 类型2包的内存缓存区
  char buff[1024*8];
};
```

这两种数据包读取到缓冲区时，可以共享 **union mem buf** 实例的内存空间。



## 6、测试系统是 大端存储 or 小端存储

### 1. 大端存储or小端存储

存储**0x11223344**：

- 1、大端存储：[0x44][0x33][0x22][0x11] => 低存高
- 2、小端存储：[0x11][0x2][0x33][0x44] => 低存低

### 2.【指针变量】判断

```c
#include <stdio.h>

int main()
{
	int a = 0;
	printf("-------------------------------\n");

	// 改变指针步长为1个字节
	char* p = (char*)&a;

	// 打印p指向地址的变化趋势
	printf("p+0 = %p\n", p);
	printf("p+1 = %p\n", p+1);
	printf("p+2 = %p\n", p+2);
	printf("p+3 = %p\n", p+3);
	printf("-------------------------------\n");

	// 使用 0x十六进制 填充变量a对应4个字节
	// => 1个十六进制数，占用4个二进制位
	// => 2个十六进制数，占用8个二进制位 => 1个字节
	*p = 0x11; // 最低位1个字节，存储0x11
	*(p+1) = 0x22;
	*(p+2) = 0x33;
	*(p+3) = 0x44;// 最高位1个字节，存储0x44

	// 依次打印每一个字节的数值
	printf("p+0 = %p, *(p+0) = 0x%x\n", p, *p);
	printf("p+1 = %p, *(p+1) = 0x%x\n", p+1, *(p+1));
	printf("p+2 = %p, *(p+2) = 0x%x\n", p+2, *(p+2));
	printf("p+3 = %p, *(p+3) = 0x%x\n", p+3, *(p+3));
	printf("-------------------------------\n");

	// 直接打印变量a内存值
	printf("a = 0x%x\n", a);
	printf("-------------------------------\n");

	// 判断大小端对齐方式
	if (0x44332211 == a)
		printf("小端对齐\n");
	else
		printf("大端对齐\n");
}
```

```
->  gcc main.c
->  ./a.out
-------------------------------
p+0 = 0xbfafd914
p+1 = 0xbfafd915
p+2 = 0xbfafd916
p+3 = 0xbfafd917
-------------------------------
p+0 = 0xbfafd914, *(p+0) = 0x11
p+1 = 0xbfafd915, *(p+1) = 0x22
p+2 = 0xbfafd916, *(p+2) = 0x33
p+3 = 0xbfafd917, *(p+3) = 0x44
-------------------------------
a = 0x44332211
-------------------------------
小端对齐
->
```

- 1、内存是从【小】到【大】变化
- 2、依次按照内存从**小**到**大**，挨个设置4个字节的数值
- 最终 **a = 0x44332211** 
  - 最高位 **0x44**  => 最存储 **p+3** 内存最**高**地址
  - 最低位 **0x11**  => 最存储 **p** 内存最**低**地址
  - 也就是高存高、低存低 => 是**小端存储**

### 3.【union】判断

```c
#include <stdlib.h>
#include <stdio.h>

// union占用长度为长度最大的成员占用的长度，所以为4字节
union Flag
{
  char bits[4]; // 4字节
  int  flags;   // 4字节
}cc;

int main()
{
  //1.
  printf("sizeof(union Flag) = %lu\n", sizeof(union Flag));

  //2. 填充char a[4]数组的4个字节，每个字节使用二个十六进制数填充
  // 内存是由 a[0]低 -> a[3]高
  cc.bits[0] = 0x31; // 填充第一个字节(最低位)
  cc.bits[1] = 0x32; // 填充第二个字节
  cc.bits[2] = 0x33; // 填充第三个字节
  cc.bits[3] = 0x34; // 填充第四个字节(最高位)

  //3.
  if (0x34333231 == cc.flags)
    printf("小端对齐\n");
	else
		printf("大端对齐\n");
}
```

```
➜  main make
gcc main.c
./a.out
sizeof(union Flag) = 4
小端对齐
➜  main
```


### 4. readelf 直接查看elf文件的对齐方式 

```
->  gcc main.c
->  readelf -h a.out | grep Magic
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
->
```

查看Magic魔数的【第6个字节】即可知道大端or小段

- 1、0x00 => 无效格式
- 2、0x01 => **小**端格式
- 3、0x02 => **大**端格式

所以确实是**小端**存储。



## 7、union 优化改进【struct 位域 结构体】

### 1. struct 位域结构体

```c
#include <stdlib.h>
#include <stdio.h>

 /**
  * 位域结构体实例，划分1个字节的8个二进制位，表示不同的含义
  */
struct
{
#if 1
  uint8_t sleep:2;  // 占用0、1两个二进制位， mask=11 => 3
  uint8_t run:1;    // 占用2一个二进制位，    mask=100 => 1<<2
  uint8_t eat:2;    // 占用3、4两个二进制位， mask=11000 => 3<<3
  uint8_t sing:1;	  // 占用5一个二进制位，    mask=100000 => 1<<5
#else
  /** uint8_t => unsigned char */
  unsigned char sleep:2;
  unsigned char run:1;
  unsigned char eat:2;
  unsigned char sing:1;
#endif
}flag;

int main()
{
  // 逐个对位域进行赋值 => 0x0011,1110
  flag.sleep = 0x2; // 10
  flag.run = 0x1;		// 1
  flag.eat = 0x3;		// 11
  flag.sing = 0x1;	// 1

  // 打印每一个位域值
  printf("sleep\t = %d\n", flag.sleep);
	printf("run\t = %d\n", flag.run);
	printf("eat\t = %d\n", flag.eat);
  printf("sing\t = %d\n", flag.sing);
}
```

```
➜  main make
gcc main.c
./a.out
sleep	 = 2
run	 = 1
eat	 = 3
sing	 = 1
➜  main
```

对位域结构体中的成员读写，需要使用`实例.成员 = 值;`。

### 2. union 提供便捷方式, 读写位域结构体

```c
#include <stdlib.h>
#include <stdio.h>

union Bits
{
  /**
   * 1、位域结构体实例，划分1个字节的8个二进制位，表示不同的含义
   */
  struct
  {
  #if 1
    uint8_t sleep:2;  // 占用0、1两个二进制位， mask=11 => 3
    uint8_t run:1;    // 占用2一个二进制位，    mask=100 => 1<<2
    uint8_t eat:2;    // 占用3、4两个二进制位， mask=11000 => 3<<3
    uint8_t sing:1;	  // 占用5一个二进制位，    mask=100000 => 1<<5
  #else
    /** uint8_t => unsigned char */
    unsigned char sleep:2;
    unsigned char run:1;
    unsigned char eat:2;
    unsigned char sing:1;
  #endif
  }flag;

  /**
   * 2、直接获取union整个1字节的内存
   * 注意：bits的内存长度 >= 位域结构体的总长度
   */
  #if 1
    uint8_t bits;
  #else
    /** uint8_t => unsigned char */
    unsigned char bits;
  #endif
}bitfield;

int main()
{
  // 1、清零
  bitfield.bits = 0;    

  // 2、直接赋1字节的bits内存
  bitfield.bits = 0x3e; // 二进制：0011,1110

  // 3、通过【位域结构体】打印每一个位域值
  printf("--------------------------\n");
  printf("sleep\t = %d\n",  bitfield.flag.sleep);
  printf("run\t = %d\n",    bitfield.flag.run);
  printf("eat\t = %d\n",    bitfield.flag.eat);
  printf("sing\t = %d\n",   bitfield.flag.sing);
  

  // 4、通过【bits】打印每一个位域值
  printf("--------------------------\n");
  printf("sleep\t = %d\n",    bitfield.bits & 0x3);
  printf("run\t = %d\n",      (bitfield.bits & (0x1<<2)) >> 2);
  printf("eat\t = %d\n",      (bitfield.bits & (0x3<<3)) >> 3);
  printf("sing\t = %d\n",     (bitfield.bits & (0x3<<5)) >> 5);
}
```

```
➜  main make
gcc main.c
./a.out
--------------------------
sleep	 = 2
run	 = 1
eat	 = 3
sing	 = 1
--------------------------
sleep	 = 2
run	 = 1
eat	 = 3
sing	 = 1
➜  main
```