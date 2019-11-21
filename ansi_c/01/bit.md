[TOC]



## 1、所有位运算的操作符

| 运算符 | 运算   | 作用                                                         |
| :----- | :----- | :----------------------------------------------------------- |
| &      | 按位与 | 双方都是1，结果才为1，否则结果为0。注意：**&&**表示**逻辑与**。 |
| \|     | 按位或 | 只要1个为1，则结果为1。其实就是两个数的对应**二进制位**相加。注意：**\|\|**表示**逻辑或**。 |
| ^      | 异或   | 不相同=1，相同=0                                             |
| ~      | 取反   | 0变1，1变0                                                   |
| `<<`   | 左移   | 各二进位全部`左移`若干位，高位丢弃，**低位补0** 。每左移1位，相当于**乘以2** |
| `>>`   | 右移   | 各二进位全部`右移`若干位。对**无符号**数，**高位补0**。**有符号**数，各编译器处理方法不一样，有的**补符号位**（算术右移），有的**补0**（逻辑右移）。每右移1位，相当于**除以2** |

注意`&` , `|` , `^` 这三个位运算符具备**交换律**和**结合律**

- 交换律

```
a ^ b ^ c ==> a ^ c ^ b
```

- 结合律

```
(a & b) & c ==> a & (b & c)
```



## 2、位运算规律总结

### 1. bit 取反: ~[0|1] => [1|0]

```c
#include <stdio.h>
#include <stdlib.h> 

int main(int argc, const char * argv[]) 
{
    printf("~0      = %d\n", ~0);
    printf("~1      = %d\n", ~1);
    printf("~(-1)   = %d\n", ~(-1));
    printf("~8      = %d\n", ~8);
    printf("~(-8)   = %d\n", ~(-8));
}
```

```
~0      = -1
~1      = -2
~(-1)   = 0
~8      = -9
~(-8)   = 7
```

### 2. 获取 bit 值: [0|1] `& 1`  => [0|1] => 不变

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
  printf("1 & 1 = %d\n", 1 & 1);
  printf("0 & 1 = %d\n", 0 & 1);
}
```

```
->  gcc main.c
->  ./a.out
1 & 1 = 1
0 & 1 = 0
->
```

### 3. bit 清零: [0|1] `& 0`  => 0

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
  printf("1 & 0 = %d\n", 1 & 0);
  printf("0 & 0 = %d\n", 0 & 0);
}
```

```
->  gcc main.c
->  ./a.out
1 & 0 = 0
0 & 0 = 0
->
```

### 4. bit 置 1: [0|1] `| 1` => 1

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
  // 按位或：一真则全真
  printf("0 | 1 = %d\n", 0 | 1);
  printf("1 | 1 = %d\n", 1 | 1);
}
```

```
➜  main gcc main.c
➜  main ./a.out
0 | 1 = 1
1 | 1 = 1
➜  main
```

### 5. bit 取反 : [0|1] `^ 1` => [1|0]

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
  printf("0 ^ 1 = %d\n", 0 ^ 1);
  printf("1 ^ 1 = %d\n", 1 ^ 1);
}
```

```
->  gcc main.c
->  ./a.out
0 ^ 1 = 1
1 ^ 1 = 0
->
```

### 6. bit 清零: `X ^ X` = 0

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
  printf("1 ^ 1 = %d\n", 1 ^ 1);
  printf("0 ^ 0 = %d\n", 0 ^ 0);
}
```

```
->  gcc main.c
->  ./a.out
1 ^ 1 = 0
0 ^ 0 = 0
->
```

### 7. bit 不变: [0|1] `^ 0` => [0|1]

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
  printf("1 ^ 0 = %d\n", 1 ^ 0);
  printf("0 ^ 0 = %d\n", 0 ^ 0);
}
```

```
->  gcc main.c
->  ./a.out
1 ^ 0 = 1
0 ^ 0 = 0
->
```

### 8. bit 不变 : X `| 0` => X

..



## 3、使用十六进制数填充二进制位

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argv, char* argr[])
{
  // 两个十六进制数，占用1个字节
  // int 占用4个字节
  // 所以每两个十六进制数填充1个字节
  int a = 0x11223344;
  printf("a = %x\n", a);
}
```

```
->  gcc bit.c
->  ./a.out
a = 11223344
->
```



## 4、二进制位【清零】 => bit `& 0`

将【第5】位清零：

```c
#include <stdio.h>
#include <stdlib.h>

#if 1
int main()
{
  // 1个字节 ==> 8个二进制位 => 1111,1111
  int x = 0xFF;
  
  // 第一步、左移4位，然后全部按位取反 => 让4位的二进制位变成0
  int mask = ~(1 << 4);

  // 第二步、1 & 0 => 0
  x = x & mask; 		//1110,1111 >>> 0xEF

  printf("x = %X\n", x);
}
#else
int main()
{
  int x = 0xFF;
  x &= ~(1 << 4);
  printf("x = %X\n", x);
}
#endif
```

```
➜  main make
gcc main.c
./a.out
x = EF
➜  main
```

ef = 0x1110,1111



## 5、二进制位【置1】 => bit `| 1`

### 1. 将【第5位】位置1

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
  // 二进制位为：1010,0000
  int x = 0xA0;

#if 1
  // 第一步、
  int mask = 1 << 4;

  // 第二步、
  x |= mask;		// 1011,0000 >>> 0xB0

#else
  x |= (1 << 4);
#endif

  printf("x = %X\n", x);
}
```

```
➜  main make
gcc main.c
./a.out
x = B0
➜  main
```

### 2. 将【下标值=6】位置1

```c
#include <stdio.h>

int main()
{
  int x = 0xA0;   // 1010,0000
  x |= (1 << 6);  // 1110,0000 => 0xE0
  printf("x = %X\n", x);
}
```

```
➜  main make
gcc main.c
./a.out
x = E0
➜  main
```

### 3. 先清零，再置1

```c
#include <stdio.h>

int main()
{
  int x = 0xA0;   // 1010,0000

  x &= ~(1 << 6); // 先将下标值=6的bit位【清零】
  printf("x = %X\n", x);
  
  x |= (1 << 6);  // 1110,0000 => 0xE0
  printf("x = %X\n", x);
}
```

```
➜  main make
gcc main.c
./a.out
x = A0
x = E0
➜  main
```



## 6、【获取】二进制位的值 => bit `& 1`


```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
  // 二进制位为：1010,1011
  int x = 0xAB;

  // 获取【第5位】的0|1
  int val1 = x & (1<<4); // 1010,1011 & 0001,0000 = 0000,0000 = 0x00
  printf("val1 = %X\n", val1);

  // 获取【第6位】的0|1，但是得到的包含后续所有二进制位的值
  int val2 = x & (1<<5); // 1010,1011 & 0010,0000 = 0010,0000 = 0x20
  printf("val2 = 0x%X\n", val2);

  // 获取【第6位】的0|1，只得到0或1
  int val3 = (x & (1<<5)) >> 5;
  printf("val3 = %d\n", val3);
}

```

```
➜  main make
gcc main.c
./a.out
val1 = 0
val2 = 0x20
val3 = 1
➜  main
```



## 7、位移枚举

### 1. 将32位（4字节）切割成多个区域

| 掩码序列  | 32位掩码十六进制数   | 32位掩码二进制数 | 获取32位目标二进制串中的部分位 | mask值左移位数 |
|:---- |:----------- |:--------------| ---------------- | --------- |
| mask1 | 0xFF  | 0000,0000,0000,0000,0000,0000,1111,1111 | 0 ~ 7 位   | 0位        |
| mask2 | 0xFF00  | 0000,0000,0000,0000,1111,1111,0000,0000 | 8 ~ 15 位 | 0 ~ 8位    |
| mask3 | 0xFF, 0000 | 0000,0000,1111,1111,0000,0000,0000,0000 | 16 ~ 23 位 | 0 ~ 16位   |
| mask4 | 0xFF00, 0000 | 1111,1111,0000,0000,0000,0000,0000,0000 | 24 ~ 31 位 | 0 ~ 24位   |

### 2. 获取对应部分二进制位的数值

那么给定一个**包如上4个区间数值之和**的数值**value**时，获取对应部分二进制位的数值：

```c
1. 获取x的[0~7]位的值 	=> value & mask1;
2. 获取x的[8~15]位的值 	=> value & mask2;
3. 获取x的[16~23]位的值 	=> value & mask3;
4. 获取x的[24~31]位的值 	=> value & mask4;
```

### 3. 具体示例

```c
#include <stdio.h>
#include <stdlib.h>

// sizeof(enum option) = 4字节 = 32位
enum option
{
  option_none = 0,

//////////////0到7位（单选）//////////////
  option_0_7_mask = 0xFF, 			//1111,1111
  option1_1 = 1,
  option1_2 = 2,
  option1_3 = 3,

//////////////8到15位（多选）//////////////
  option_8_15_mask = 0xFF00,			//1111,1111,0000,0000
  option2_1 = 1 << 8,
  option2_2 = 1 << 9,
  option2_3 = 1 << 10,

//////////////16到23位（多选）//////////////
  option_16_23_mask = 0xFF0000,		//1111,1111,0000,0000,0000,0000
  option3_1 = 1 << 16,
  option3_2 = 1 << 17,
  option3_3 = 1 << 18,

//////////////8到15位（多选）//////////////
  option_24_31_mask = 0xFF000000,		//1111,1111,0000,0000,0000,0000,0000,0000
  option4_1 = 1 << 24,
  option4_2 = 1 << 25,
  option4_3 = 1 << 26,
};

void show(enum option opt)
{
  enum option tmp;

  printf("\n");

  //1. 解析[0,7]单选
  {
    printf("解析 option1\n");
    tmp = opt & option_0_7_mask;
    switch (tmp)
    {
      case option1_1:
        printf("=> option1_1\n");
        break;
      case option1_2:
        printf("=> option1_2\n");
        break;
      case option1_3:
        printf("=> option1_3\n");
        break;
      default:
        printf("non options\n");
        break;
    }
    printf("\n");
  }

  //2. 解析[8,15]多选
  {
    printf("解析 option2\n");
    tmp = opt & option_8_15_mask;
  
    if (option2_1 == (tmp & option2_1)) /* 注意： == 优先级高于 & */
      printf("=> option2_1\n");
    
    if (option2_2 == (tmp & option2_2))
      printf("=> option2_2\n");
    
    if (option2_3 == (tmp & option2_3))
      printf("=> option2_3\n");

    printf("\n");
  }

  //2. 解析[16,23]多选
  {
    printf("解析 option3\n");
    tmp = opt & option_16_23_mask;
    
    if (option3_1 == (tmp & option3_1))
      printf("=> option3_1\n");
    
    if (option3_2 == (tmp & option3_2))
      printf("=> option3_2\n");
    
    if (option3_3 == (tmp & option3_3))
      printf("=> option3_3\n");

    printf("\n");
  }

  //2. 解析[24,31]多选
  {
    printf("解析 option4\n");
    tmp = opt & option_24_31_mask;
    
    if (option4_1 == (tmp & option4_1))
      printf("=> option4_1\n");
    
    if (option4_2 == (tmp & option4_2))
      printf("=> option4_2\n");
    
    if (option4_3 == (tmp & option4_3))
      printf("=> option4_3\n");
  }
}

int main(int argv, char* argr[])
{
  printf("sizeof(enum option) = %lu\n\n", sizeof(enum option));

  //1.
  enum option opt = option_none;

  //2. 
  opt |= option1_2;
  printf("选择了 option1_2\n");

  //3.
  opt |= option2_1;
  opt |= option2_3;
  printf("选择了 option2_1\n");
  printf("选择了 option2_3\n");

  //3.
  opt |= option3_2;
  opt |= option3_3;
  printf("选择了 option3_2\n");
  printf("选择了 option3_3\n");

  //4.
  opt |= option4_1;
  opt |= option4_3;
  printf("选择了 option4_1\n");
  printf("选择了 option4_3\n");

  //5.
  show(opt);
}
```

```
->  gcc main.c
->  ./a.out
sizeof(enum option) = 4

选择了 option1_2
选择了 option2_1
选择了 option2_3
选择了 option3_2
选择了 option3_3
选择了 option4_1
选择了 option4_3

解析 option1
=> option1_2

解析 option2
=> option2_1
=> option2_3

解析 option3
=> option3_2
=> option3_3

解析 option4
=> option4_1
=> option4_3
->
```



## 8、10进制转2进制

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * 不断的左移获取bit位，
 * bit位%2得到0或1
 */

// 二进制1010 => 十进制1010
void func1(int n)
{
  int result=0,k=1,i,temp;
  temp = n;
  while(temp)
  {
    i = temp%2;
    result = k * i + result;
    k = k*10;
    temp = temp/2; // temp>>1
  }
  printf("%d\n", result);
}

// 递归打印二进制的每一个bit位
void func2(int n)
{
  // 1. 递归结束
  if (n <= 0)
    return;

  // 2. 先递归
#if 1
  func2(n/2);
#else
  func2(n>>1);
#endif

  // 3. 后打印
  printf("%d", n%2);
}

#if 1
int main()
{
  func1(10);
  func2(10);
  printf("\n");
}
#endif

// 1000 => 8
// 1001 => 9
// 1010 => 10
```

```
➜  main make
gcc main.c
./a.out
1010
1010
➜  main
```



## 8、【交换】两个变量的值

### 1. 不借助【中间变量】2种方法

![](Snip20190422_1.png)

### 2. 使用【位或】

#### c 版本

```c
#include <stdio.h>
#include <stdlib.h>

void swap(int* a, int* b)  
{  
  *a ^= *b;
  *b ^= *a; 
  *a ^= *b;
}

int main()
{
  int a = 3;
  int b = 4;

  printf("a = %d, b = %d\n", a, b);
  swap(&a, &b);
  printf("a = %d, b = %d\n", a, b);
}
```

运行输出

```
->  gcc main.c
->  ./a.out
a = 3, b = 4
a = 4, b = 3
```

#### c++ 版本

```c
#include <iostream>
#include <cstdio>

void swap(int & a, int & b)  
{  
  a ^= b;  
  b ^= a;  
  a ^= b;  
}

int main(int argc, const char * argv[]) 
{

  int a = 3;
  int b = 4;

  printf("a = %d, b = %d\n", a, b);

  swap(a, b);

  printf("a = %d, b = %d\n", a, b);

  return 0;
}
```

运行输出

```
->  g++ main.cpp
->  ./a.out
a = 3, b = 4
a = 4, b = 3
```



## 9、获得 int 最大值

```cql
#include <stdio.h>

#include <stdio.h>

int main()
{
  // 1. 100000.....000 (31个0)
  int a = 1 << ((sizeof(int) * 8) - 1);
  printf("%d\n", a);

  // 2. 100000.....000 (31个0) - 1 = 01111...1111(31个1) => 4字节最大正数
  a = a - 1;
  printf("%d\n", a);
}
```

```
->  gcc main.c
->  ./a.out
-2147483648
2147483647
->
```



## 10、左移与右移

###  1. << 左移，高位丢弃，低位补0，相当于乘以2

```
001（第一个0是符号位，表示正数） << 1 
==> 010 
==> 2
```

### 2. >> 右移，高位补符号位，低位丢弃，相当于除以2

```
01000(正数) >> 3
==> 00001 
==> 1
```

```
11000(负数) >> 3
==> 11111
```

### 3. 正数右移补0，负数右移补符号位1

```c
#include <stdlib.h>
#include <stdio.h>

int main(int argv, char* argr[])
{
  //1. 正的int数值，左移31位 => 0000..0000(32个0) => 0
    int a = 19;
    printf("a>>31 = %d\n", a>>31);

  //2. 负的int数值，左移31位 => 111.....111(32个1) => -1的补码
    int b = -19;
    printf("b>>31 = %d\n", b>>31);
}
```

输出

```
a>>31 = 0
b>>31 = -1
```

- **正数**右移最**小**值 => **0**
- **负数**右移最**大**值 => **-1**


### 4. 获取符号位

```c
#include <stdlib.h>
#include <stdio.h>

/**
 *	判断符号位
 *	
 *	- return 0 	正数
 *	- return <0 负数
 */
int cmp(int x)
{	
    /**
     * - 1) 正数>>31 => 0
     * - 2) 负数>>31 => -1
     */
  if ((x>>31) < 0)
    return -1;
  else 
    return 0;
}

int main(int argv, char* argr[])
{
  //1. 正的int数值，左移31位
    int a = 19;
    printf("%d\n", cmp(a));

  //2. 负的int数值，左移31位
    int b = -19;
    printf("%d\n", cmp(b));
}
```

编译链接运行

```
->  gcc bit.c
->  ./a.out
0
-1
```



## 11、取绝对值

### 1. 负数变正数的计算公式

- (1) 对负数的二进制位全部**取反**
- (2) 然后二进制位**最低位 + 1**
- (3) 如果本来就是**正数**，则**不会发生变化**

比如对 `-6` 取绝对值的过程：

```
1. -6的原码: 1111 1010 (二进制)
2. 取反： 0000 0101 (二进制)
3. 加1：0000 0110 ==> 1*2^2 + 1*2^1 = 6
```

对应取反再加1的代码如下

```c
int SignReversal(int a)  
{  
  /**
   *	- (1) 全部二进制位进行取反
   * 	- (2) 低位二进制位再加1
   */
    return ~a + 1;  
}
```

### 2. 获取符号位，只对负数进行取反加1

```c
int abs_1(int a)  
{  
  //1. 先向右移动31位，来取最高位的符号位。
  // i == -1, 负数
  // i == 0, 正数
    int i = a >> 31;
    
    //2. 负数，则对其取反，再加1
    return i == 0 ? a : (~a + 1);  
} 
```

如上代码关键点：

- (1) int类型，占用4个字节，即32个二进制位
- (2) `负数`最高位是`符号`位，用来表示正数还是负数，不参与数值运算
- (3) 可以将`负数`的二进制码，`向右移动31位`，就能够获得最高位的`符号位`
  - `n >> 31` == `-1`，则表示n是`负数`
  - `n >> 31` == `0`，则表示n是`正数`

### 3. 优化后的版本

```c
#include <stdio.h>
#include <stdlib.h>

// 默认编译器会给int加上signed，即区分正负数的
int myfabs(int n) 
{
    //1. 左移31位，全部用符号位填充
    //- 如果n是负数，则二进制位全部都是【1】，十进制【真值】是【-1】
    //- 如果n是正数，则二进制位全部都是【0】，十进制【真值】是【0】
    int mask = n >> ((sizeof(int) * 8) - 1);
    
    //2. 按位取反，然后 + 1
    //- 如果此时mask=-1，则mask的二进制位为1111....111(32个1)
    //- 任意数 ^ 1 => 取反
    //- (n ^ 11111...11111) - (-1) ==> 取反后再加1 ==> 得到正数
    //- 如果n是正数，就不变
    return (n ^ mask) - mask;
}

int main(int argv, char* argr[])
{
  int x = -299;
  printf("myfabs(-299) = %d\n", myfabs(x));
}
```

编译运行

```
->  gcc main.c
->  ./a.out
myfabs(-299) = 299
```



## 12、加解密

### 1. 注释版

对一个字节的数据，通过**移位、位或、位与**等运算，完成加密/解密。

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * 对传入的【1字节】数据进行【加密】
 * 
 * @param val 传入需要加密的【1个字节】数据
 * @return 返回存储加密后的【2个字节】的内存
 */ 
int16_t encry_byte(int8_t val)
{
  /**
  	1、先将传入的待加密的【1字节】数据，写入到局部栈帧上的【2字节】内存中
    2、再将【2字节】内存中的【低1字节】数据【左移4位】，则空出来的【低4位】用来做混淆
    3、再空出来的【低4位】中，随机分配 0000 ~ 1111 之间的数值，再解密时会直接【右移4位】去掉【低4位】的随机值
   */
  
  // 1. 将【1字节数】据写入到【2字节】长度的内存中
  int16_t encryted = val;

  // 2. 将2字节内存中数据左移4位
  encryted <<= 4;

  // 3. 将2字节（0~15位）的最高位【赋值1】，即变为【负数】
  encryted |= (1<<(sizeof(int16_t) * 8 - 1));

  // 4. 低4个二进制位，随机分配【1（0001）~15（1111）】之间的数值
  encryted += rand()%15 + 1;

  // 5. 返回的已加密的2个字节长度的数据
  return encryted;
}

/**
 * 对传入的【1字节】数据进行【解密】
 * 
 * @param val 要解密的2个字节数据。即 encry_byte() 的返回值。
 * @return 返回1个字节的解密后的数据
 */ 
int8_t decry_byte(int16_t val)
{
  // 1. 
  int16_t decryted = val;

  // 2. 左移1位，去掉最高位的符号位
  decryted <<= 1;

  // 3. 再向右移动5位，去掉在低4二进制位添加的0~15随机值
  decryted >>= 5;

  // 4. 返回解密后的值
  return decryted;
}

int main()
{
  //1. 原始数据：0010,0011（1个字节）
  // char ch = 0x23;
  int8_t ch = 0x23;
  printf("原始数据 = %x\n", ch);

  //2. 加密1个字节的数据
  // => 一定要使用short接收返回值
  // => encry_byte()返回的是2个字节数据
  // => 如果使用char接收返回值，则会截掉高位的1字节数据，造成后续无法解密
  // short encryted = encry_byte(ch);
  int16_t encryted = encry_byte(ch);
  printf("加密后 = %x\n", encryted);

  //3. 解密1个字节的数据
  // => 此时只需要使用char接收返回值
  // => decry_byte()只返回解密后的1个字节数据
  int8_t decryted = decry_byte(encryted);
  printf("解密后 = %x\n", decryted);
}
```

```
->  gcc main.c
->  ./a.out
原始数据 = 23
加密后 = ffff8238
解密后 = 23
->
```

### 2. 无注释版

```c
#include <stdio.h>
#include <stdlib.h>

int16_t encry_byte(int8_t val)
{
  int16_t encryted = val; 
  encryted <<= 4; 
  encryted |= (1<<(sizeof(int16_t) * 8 - 1));
  encryted += rand()%15 + 1;
  return encryted;
}
 
int8_t decry_byte(int16_t val)
{
  int16_t decryted = val;
  decryted <<= 1;
  decryted >>= 5;
  return decryted;
}

int main()
{
  int8_t ch = 0x23;
  printf("ch = 0x%02X\n", ch);
  int16_t encryted = encry_byte(ch);
  printf("encryted = %x\n", encryted);
  int8_t decryted = decry_byte(encryted);
  printf("decryted = 0x%02X\n", decryted);
}
```


```
➜  main make
gcc main.c
./a.out
ch = 0x23
encryted = ffff8238
decryted = 0x23
➜  main
```


