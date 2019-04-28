# 第二章 类型、运算符与表达式
## 练习2-1
### 问题描述
编写一个程序以确定分别由 `signed` 及 `unsigned` 限定的 `char` 、 `short` 、 `int` 与 `long` 类型变量的取值范围。采用打印标准头文件中的相应值以及直接计算两种方式实现。
### 问题解答
利用标准头文件来确定
```C
#include<stdio.h>
#include<limits.h>

int main(){
    printf("signed char min = %d\n", SCHAR_MIN);
    printf("signed char max = %d\n", SCHAR_MAX);
    printf("signed short min = %d\n", SHRT_MIN);
    printf("signed short max = %d\n", SHRT_MAX);
    printf("signed int min = %d\n", INT_MIN);
    printf("signed int max = %d\n", INT_MAX);
    printf("signed long min = %ld\n", LONG_MIN);
    printf("signed long max = %ld\n", LONG_MAX);

    printf("unsigned char max = %u\n", UCHAR_MAX);
    printf("unsigned short max = %u\n", USHRT_MAX);
    printf("unsigned int max = %u\n", UINT_MAX);
    printf("unsigned long max = %lu\n", ULONG_MAX);
    return 0;
}
```
ANSI C标准规定：各种类型的取值范围必须在头文件<limits.h>中定义。

直接计算
```C
#include<stdio.h>
int main(){
    printf("signed char min = %d\n", ~(char)((unsigned char) ~0 >> 1));
    printf("signed char max = %d\n", (char)((unsigned char) ~0 >> 1));
    printf("signed short min = %d\n", ~(short)((unsigned short) ~0 >> 1));
    printf("signed short max = %d\n", (short)((unsigned short) ~0 >> 1));
    printf("signed int min = %d\n", ~(int)((unsigned int) ~0 >> 1));
    printf("signed int max = %d\n", (int)((unsigned int) ~0 >> 1));
    printf("signed long min = %ld\n", ~(long)((unsigned long) ~0 >> 1));
    printf("signed long max = %ld\n", (long)((unsigned long) ~0 >> 1));

    printf("unsigned char max = %u\n", (unsigned char) ~0);
    printf("unsigned short max = %u\n", (unsigned short) ~0);
    printf("unsigned int max = %u\n", (unsigned int) ~0);
    printf("unsigned long max = %u\n", (unsigned long) ~0);
    return 0;
}
```
该解决方案采用按位运算符来计算。比如
```C
-(char)((unsigned char) ~0 >> 1)
```
先将数字0取反，将其各个二进制位置1，然后转换为 `unsigned char` 类型，再将其右移一位以清除符号位，然后将其转换为 `char` 类型，最后再取反即为 `signed char` 类型最小值。
## 练习2-2
### 问题描述
在不使用 `&&` 或 `||` 的条件下编写一个与上面的 `for` 循环语句等价的循环语句。
### 问题解答
原来的 `for` 循环语句：
```C
for(i = 0; i < lim - 1 && (c = getchar()) != '\n' && c != EOF; ++i)
```
与之等价的循环语句：
```C
enum loop {NO, YES};
enum loop okloop = YES;
i = 0;
while(okloop == YES){
    if(i >= lim - 1)
        okloop = NO;
    else if((c = getchar()) == '\n')
        okloop = NO;
    else if(c == EOF)
        okloop = NO;
    else{
        s[i] = c;
        ++i;
    }
}
```
## 练习2-3
### 问题描述
编写函数htoi(s)，把由十六进制数字组成的字符串（包括可选的前缀0x或0X）转换为与之等价的整型值。字符串允许包含的数字包括：0~9、a~f和A~F。
### 问题解答
```C
#define YES 1
#define NO 0

int htoi(char s[]){
    int hexdigit, i, inhex, n;
    i = 0;
    if(s[i] == '0'){
        ++i;
        if(s[i] == 'x' || s[i] == 'X')
            ++i;
    }
    n = 0;
    inhex = YES;
    for(; inhex == YES; ++i){
        if(s[i] >= '0' && s[i] <= '9')
            hexdigit = s[i] - '0';
        else if(s[i] >= 'a' && s[i] <= 'f')
            hexdigit = s[i] - 'a' + 10;
        else if(s[i] >= 'A' && s[i] <= 'F')
            hexdigit = s[i] - 'A' + 10;
        else
            inhex = NO;
        if(inhex == YES)
            n = 16 * n + hexdigit;
    }
    return n;
}
```
## 练习2-4
### 问题描述
重新编写函数squeeze(s1, s2)，将字符串s1中任何与字符串s2中字符匹配的字符都删除。
### 问题解答
```C
void squeeze(char s1[], char s2[]){
    int i, j, k;
    for(i = k = 0; s1[i] != '\0'; ++i){
        // 遍历s2，执行到字符串结束或者找到一个匹配字符
        for(j = 0; s2[j] != '\0' && s2[j] != s1[i]; ++j)
            ;
        // 如果没有找到匹配的，就添加，如果找到匹配的，就跳过
        if(s2[j] == '\0')
            s1[k++] = s1[i];
    }
    s1[k] = '\0';
}
```
## 练习2-5
### 问题描述
编写函数any(s1, s2)，将字符串s2中的任一字符在字符串s1中第一次出现的位置作为结果返回。如果s1中不包含s2中的字符，则返回-1。（标准库函数strpbrk具有相同的功能，但是它返回的是指向该位置的指针。）
### 问题解答
暴力匹配
```C
int any(char s1[], char s2[]){
    int i, j;
    for(i = 0; s1[i] != '\0'; ++i){
        for(j = 0; s2[j] != '\0'; ++j){
            if(s1[i] == s2[j])
                return i;
        }
    }
    return -1;
}
```
## 练习2-6
### 问题描述
编写一个函数setbits(x, p, n, y)，该函数返回对x执行下列操作后的结果值：将x从第p位开始的n个（二进制）位设置为y中最右边n位的值，x的其余各位保持不变。
### 问题解答
```C
unsigned setbits(unsigned x, int p, int n, unsigned y){
    return x & ~(~(~0 << n) << (p + 1 - n)) |
        (y & ~(~0 << n)) << (p + 1 - n);
}
```
问题描述如下所示：

&emsp;&emsp;xxx...xnn..nnnx...xxx   x
&emsp;&emsp;yyy........yyyyn..nnn   y

需要将x中从第p为开始的n位置0，把y中最右边n位移动与x的对应位对齐，将y的其他位置0，然后对结果值进行OR操作。即：

&emsp;&emsp;xxx...x00..000x...xxx   x
&emsp;&emsp;000...0nn..nnn0...000   y

为了对x的n个位置0，需要生成个屏蔽码与x进行AND操作。该屏蔽码其他位是1，与x的n个位对应的位是0。

首先制造一个右边n位是1，左边全为0的屏蔽码：

&emsp;&emsp;~0 << n

然后对屏蔽码取反，右边n为设为1，左边为0：

&emsp;&emsp;~(~0 << n)

然后将屏蔽码左移到指定位置，即左移p+1-n位，再次取反：

&emsp;&emsp;~(~(~0 << n) << (p + 1 - n))

最后将该屏蔽码与x做AND操作，就完成了对x的对应位的清零工作：

&emsp;&emsp;x & ~(~(~0 << n) << (p + 1 - n))

对y的处理与之类似，只是最后少了个取反操作，因为y要保留对应位，而将其他位置0。

最后将两个结果进行OR操作，即完成题目要求。

## 练习2-7
### 问题描述
编写一个函数invert(x, p, n)，该函数返回对x执行下列操作后的结果值：将x中的第p位开始的n个（二进制）位取反，x的其他位不变。
### 问题解答
```C
unsigned invert(unsigned x, int p, int n){
    return x ^ (~(~0 << n) << (p + 1 - n));
}
```
与练习2-6类似，生成对应位是1，其他位是0的屏蔽码， 然后与x进行异或操作即可。

**总结：** 若要对一个数指定位进行取反，只需要将该数 与 指定位是1，其它位是0的屏蔽码进行异或操作即可。（因为0与1异或，结果为1，实现取反；1与1异或，结果为0，实现异或。而0与0异或，结果为0，保持不变；1与0异或，结果为1，保持不变。）

## 练习2-8
### 问题描述
编写一个函数rightrot(x, n)，该函数返回将x循环右移n位后所得的值。
### 问题解答
解法一：
```C
unsigned rightrol(unsigned x, int n){
    int rbit;
    while(n-- > 0){
        rbit = (x & 1) << (wordlength() - 1);
        x >>= 1;
        x = x | rbit;
    }
    return x;
}
int wordlength(void){
    int i;
    unsigned v = (unsigned) ~0;
    for(i = 1; (v >>= 1) > 0; ++i)
        ;
    return i;
}
```
使用变量 `rbit` 来将x最右边的位移动到最左边，然后把x右移一位，再通过OR操作，把rbit中保存的原x最右位赋给x最左位，即实现一次循环右移。然后循环n次即可。

函数wordlength()用来计算具体机器的字长。

解法二：
```C
unsigned rightrol(unsigned x, int n){
    unsigned rbits;
    if((n %= wordlength()) > 0){
        rbits = ~(~0 << n) & x;// 得到x的右边n位
        rbits = rbits << (wordlength() - n);// 将所得n位右移(字长-n）位，即到达x的高位
        x >>= n;// 将x右移n位，空出n个高位
        x |= rbits;// x与rbits进行OR运算。实现循环右移
    }
    return x;
}
```
首先对n进行处理，如果n为当前字长的整数倍，则x保持不变，不需处理。如果n大于字长，则循环右移次数等于n对字长求模的移动次数。

解法二不需要循环，且避免了没必要的移动，要快捷些。

## 练习2-9
### 问题描述
在求对二的补码时，表达式 `x &= (x - 1)` 可以删除x中最右边值为1的一个二进制位。请解释这样做的道理。用这一方法重写bitcount函数，以加快其执行速度。
### 问题解答
```C
int bitcount(unsigned x){
    int b;
    for(b = 0; x != 0; x &= x - 1)
        ++b;
    return b;
}
```
因为 (x - 1) + 1 = x，即(x - 1)的最右端0在x中的变为了1。也就是说，x最右端1的位与(x - 1)在同一位置上的0是对应的，所以两者做AND操作会清除x最右端1的位。可自行举例测试此结论。

应用此结论来统计x中1的个数：当x不为0的时候，每进行一次 `x &= (x - 1)` ，即消除了x最右边的一个1，统计结果b加1。当x为0，统计结束，返回b。

最坏的情况是x的所有位全是1，此时解法二和解法一要进行同样的位操作（即字长），但是绝大多数情况下，解法二只需要进行1的个数次位操作，解法一每次都是字长次操作，解法二要执行的快一些。

## 练习2-10
### 问题描述
重新编写大写字母转小写字母的函数lower，并用条件表达式代替其中的if-else结构。
### 问题解答
```C
int lower(int c){
    return c >= 'A' && c <= 'Z' ? c + 'a' - 'A' : c;
}
```