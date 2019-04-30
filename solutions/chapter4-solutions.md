# 第四章 函数与程序结构  

## 练习4-1  
### 问题描述  
编写函数strrindex(s, t)，它返回字符串t在s中最右边出现的位置。如果s中不包含t，则返回-1。  
### 问题解答  
解法一：  
``` C
int strrindex(char s[], char t[]){
    int i, j, k, pos;
    pos = -1;
    for(i = 0; s[i] != '\0'; ++i){
        for(j = i, k = 0; t[k] != '\0' && s[j] == t[k]; ++j, ++k){
            ;
        }
        if(k > 0 && t[k] == '\0')
            pos = i;
    }
    return pos;
}
```  
类似于教材P59的strindex函数，只是教材上找到匹配即返回，该函数不返回，而仅仅记录本次匹配的位置，所以退出循环后， `pos` 保存的是最后一次匹配的位置，即最右边的位置。  

解法二：  
``` C
#include<string.h>
int strrindex(char s[], char t[]){
    int i, j, k;
    for(i = strlen(s) - strlen(t); i >= 0; --i){
        for(j = i, k = 0; t[k] != '\0' && s[j] == t[k]; ++j, ++k){
            ;
        }
        if(k > 0 && t[k] == '\0')
            return i;
    }
    return -1;
}
```  
解法二对s从右往左遍历，这里 `i` 只需从 `strlen(s) - strlen(t)` 开始即可，因为往后的位置不可能匹配成功（长度不够）。  

如果找到一个匹配，即返回当前位置，此位置即为所求，如果遍历完没有匹配，则返回-1。  

## 练习4-2  
### 问题描述  
对atof函数进行扩充，使它可以处理形如：
&emsp;&emsp;123.45e-6  
的科学表示法， 其中浮点数后可能会紧跟一个e或E以及一个指数（可能有正负号）。  
### 问题解答  
``` C
#include<ctype.h>
double atof(char s[]){
    double val, power;
    int exp, i, sign;
    // 跳过前导空格
    for(i = 0; isspace(s[i]); ++i)
        ;
    // 记录正负标志
    sign = (s[i] == '-') ? -1 : 1;
    if(s[i] == '+' || s[i] == '-')
        ++i;
    // 整数部分
    for(val = 0.0; isdigit(s[i]); ++i)
        val = 10.0 * val + s[i] - '0';
    if(s[i] == '.')
        i++;
    // 小数部分
    for(power = 1.0; isdigit(s[i]); ++i){
        val = 10.0 * val + s[i] - '0';
        power *= 10;
    }
    val = sign * val / power;
    // 遇到指数
    if(tolower(s[i] == 'e')){
        sign = (s[++i] == '-') ? -1 : 1;
        if(s[i] == '+' || s[i] == '-')
            ++i;
        for(exp = 0; isdigit(s[i]); ++i)
            exp = 10 * exp + s[i] - '0';
        // 处理指数，1指数移动一次小数点
        if(sign == 1)// 指数为正
            while(exp-- > 0)
                val *= 10;
        else// 指数为负
            while(exp-- > 0)
                val /= 10;
    }
    return val;
}
```  

## 练习4-3  
### 问题描述  
在基本框架上对计算器程序进行扩充。在该程序中加入取模 `%` 运算，并注意负数的情况。  
### 问题解答  
主程序修改如下：  
``` C
#include<stdio.h>
#include<stdlib.h>

#define MAXOP 100 // 最大长度
#define NUMBER '0' // 标识找到了一个数

int getop(char s[]);
void push(double x);
double pop(void);

int main(){
    int type;
    double op2;
    char s[MAXOP];
    while((type = getop(s)) != EOF){
        switch(type){
        case NUMBER:
            push(atof(s));
            break;
        case '+':
            push(pop() + pop());
            break;
        case '*':
            push(pop() * pop());
            break;
        case '-':
            op2 = pop();
            push(pop() - op2);
            break;
        case '/':
            op2 = pop();
            if(op2 != 0.0)
                push(pop() / op2);
            else
                printf("error: zero divisor\n");
            break;
        case '%':
            op2 = pop();
            if(op2 != 0.0)
                push(fmod(pop(), op2));
            else
                printf("error: zero divisor\n");
            break;
        case '\n':
            printf("\t%.8g\n", pop());
            break;
        default:
            printf("error: unknown command %s\n", s);
            break;
        }
    }
    return 0;
}
``` 
增加对取模运算的处理分支，调用库函数fmod来计算。  
 
getop函数修改如下：  
``` C
#include<ctype.h>
#include<string.h>
#include<ctype.h>

#define NUMBER '0'

int getch(void);
void ungetch(int);

int getop(char s[]){
    int i, c;
    // 跳过空格和制表符
    while((s[0] = c = getch()) == ' ' || c == '\t')
        ;
    s[1] = '\0';
    // 不是数字也不是小数点
    // 可能是运算符或者非法输入
    if(!isdigit(c) && c != '.' && c != '-')
        return c;
    i = 0;
    // 如果遇到'-'，判断是负数还是减号
    if(c == '-'){
        if(isdigit(c = getch()) || c == '.')
            s[++i] = c;
        else{
            if(c != EOF)
                ungetch(c);
            return '-'; // 是减号
        }
    }
    // 收集整数部分
    if(isdigit(c))
        while(isdigit(s[++i] = c = getch()))
            ;
    // 收集小数部分
    if(c == '.')
        while(isdigit(s[++i] = c = getch()))
            ;
    s[i] = '\0';
    if(c != EOF)
        ungetch(c);
    return NUMBER;
}
```  
增加对负数的判断。  

## 练习4-4  
### 问题描述  
在栈操作中添加几个命令，分币用于在不弹出元素的情况下打印栈顶元素；复制栈顶元素；交换栈顶两个元素的值。另外增加一个命令用于清空栈。  
### 问题解答  
``` C
#include<stdio.h>
#include<stdlib.h>
#include<math.h>

#define MAXOP 100 // 最大长度
#define NUMBER '0' // 标识找到了一个数

int getop(char s[]);
void push(double x);
double pop(void);
void clear(void);

int main(){
    int type;
    double op1, op2;
    char s[MAXOP];
    while((type = getop(s)) != EOF){
        switch(type){
        case NUMBER:
            push(atof(s));
            break;
        case '+':
            push(pop() + pop());
            break;
        case '*':
            push(pop() * pop());
            break;
        case '-':
            op2 = pop();
            push(pop() - op2);
            break;
        case '/':
            op2 = pop();
            if(op2 != 0.0)
                push(pop() / op2);
            else
                printf("error: zero divisor\n");
            break;
        case '?':// 打印栈顶元素
            op2 = pop();
            printf("\t%.8g\n", op2);
            push(op2);
            break;
        case 'c':// 清空栈
            clear();
            break;
        case 'd':// 复制栈顶元素
            op2 = pop();
            push(op2);
            push(op2);
            break;
        case 's':// 交换栈顶两个元素
            op1 = pop();
            op2 = pop();
            push(op1);
            push(op2);
            break;
        case '%':
            op2 = pop();
            if(op2 != 0.0)
                push(fmod(pop(), op2));
            else
                printf("error: zero divisor\n");
            break;
        case '\n':
            printf("\t%.8g\n", pop());
            break;
        default:
            printf("error: unknown command %s\n", s);
            break;
        }
    }
    return 0;
}
// 清空栈函数，只需将sp设为0
void clear(void){
    sp = 0;
}
```  

## 练习4-5  
### 问题描述  
给计算器程序增加访问sin、exp与pow等库函数的操作。  
### 问题解答  
``` C
#include<stdio.h>
#include<stdlib.h>
#include<math.h>
#include<string.h>

#define MAXOP 100 // 最大长度
#define NUMBER '0' // 标识找到了一个数
#define NAME 'n' // 标记库函数名字

int getop(char s[]);
void push(double x);
double pop(void);
void mathfnc(char s[]);

int main(){
    int type;
    double op2;
    char s[MAXOP];
    while((type = getop(s)) != EOF){
        switch(type){
        case NUMBER:
            push(atof(s));
            break;
        case NAME:
            mathfnc(s);
            break;
        case '+':
            push(pop() + pop());
            break;
        case '*':
            push(pop() * pop());
            break;
        case '-':
            op2 = pop();
            push(pop() - op2);
            break;
        case '/':
            op2 = pop();
            if(op2 != 0.0)
                push(pop() / op2);
            else
                printf("error: zero divisor\n");
            break;
        case '%':
            op2 = pop();
            if(op2 != 0.0)
                push(fmod(pop(), op2));
            else
                printf("error: zero divisor\n");
            break;
        case '\n':
            printf("\t%.8g\n", pop());
            break;
        default:
            printf("error: unknown command %s\n", s);
            break;
        }
    }
    return 0;
}
// 处理math相关函数
void mathfnc(char s[]){
    double op2;
    if(strcmp(s, "sin") == 0)
        push(sin(pop()));
    else if(strcmp(s, "cos") == 0)
        push(cos(pop()));
    else if(strcmp(s, "exp") == 0)
        push(exp(pop()));
    else if(strcmp(s, "pow") == 0){
        op2 = pop();
        push(pow(pop(), op2));
    }
    else
        printf("error: %s not supported\n", s);
```  
修改后的getop函数：  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>

#define NUMBER '0' // 标识找到了一个数
#define NAME 'n' // 标记库函数名字

int getch(void);
void ungetch(int);

int getop(char s[]){
    int i, c;
    // 跳过空格和制表符
    while((s[0] = c = getch()) == ' ' || c == '\t')
        ;
    s[1] = '\0';
    i = 0;
    if(islower(c)){
        while(islower(s[++i] = c = getch()))
            ;
        s[i] = '\0';
        if(c != EOF)
            ungetch(c);
        if(strlen(s) > 1)// 长度大于1，是函数
            return NAME;
        else // 长度为1，可能是个命令
            return c;
    }
    // 不是数字也不是小数点
    // 可能是运算符或者非法输入
    if(!isdigit(c) && c != '.')
        return c;

    // 收集整数部分
    if(isdigit(c))
        while(isdigit(s[++i] = c = getch()))
            ;
    // 收集小数部分
    if(c == '.')
        while(isdigit(s[++i] = c = getch()))
            ;
    s[i] = '\0';
    if(c != EOF)
        ungetch(c);
    return NUMBER;
}
```  

## 练习4-6  
### 问题描述  
给计算器程序增加处理变量的命令（提供26个具有单个英文字母变量名的变量很容易）。增加一个变量存放最近打印的值。  
### 问题解答  
``` C
#include<stdio.h>
#include<math.h>

#define MAXOP 100 // 最大长度
#define NUMBER '0' // 标识找到了一个数

int getop(char s[]);
void push(double x);
double pop(void);

int main(){
    int i, type var = 0;
    double op2, v;
    char s[MAXOP];
    double variable[26];

    for(i = 0; i < 26; ++i)
        variable[i] = 0.0;
    
    while((type = getop(s)) != EOF){
        switch(type){
        case NUMBER:
            push(atof(s));
            break;
        case '+':
            push(pop() + pop());
            break;
        case '*':
            push(pop() * pop());
            break;
        case '-':
            op2 = pop();
            push(pop() - op2);
            break;
        case '/':
            op2 = pop();
            if(op2 != 0.0)
                push(pop() / op2);
            else
                printf("error: zero divisor\n");
            break;
        case '=':
            pop;
            if(var >= 'A' && var <= 'Z')
                variable[var - 'A'] = pop();
            else
                printf("error: no variable name\n");
            break;
        case '\n':
            v = pop();
            printf("\t%.8g\n", v);
            break;
        default:
            printf("error: unknown command %s\n", s);
            break;
        }
        var = type;
    }
    return 0;
}
```  
用大写字母A-Z来代表变量，这些字母作为数组变量的索引。字符变量v存放打印的值。  

在遇到一个变量名时，计算器程序将该变量的值压入堆栈。当遇到 `=` 操作符时，将堆栈中，变量下边的数值赋值给该变量。如：  
&emsp;&emsp;3 A =  
将3赋值给变量A。此后，表达式：  
&emsp;&emsp;2 A +  
将把2和3（被赋值给变量A的数值）相加。计算器的换行操作符将输出字符5，同时将这个5赋值给变量v。如果下一个操作是：  
&emsp;&emsp;v 1 +  
则结果将是6，即5+1。  

## 练习4-7  
### 问题描述  
编写一个函数ungets(s)，将整个字符串压回到输入中。ungets函数需要使用buf和bufp吗？它能否仅使用ungetch函数？  
### 问题解答  
``` C
#include<string.h>

void ungets(char s[]){
    int len = strlen(s);
    void ungetch(int c);
    while(len > 0)
        ungetch(s[--len]);
}
```  
ungets函数不需要直接对buf和bufp进行操作，buf，bufp和出错检查将由函数ungetch处理。  

## 练习4-8  
### 问题描述  
假定最多只压回一个字符，请相应的修改getch和ungetch函数。  
### 问题解答  
``` C
#include<stdio.h>
char buf = 0;

int getch(void){
    int c;
    if(buf != 0)
        c = buf;
    else
        c = getchar();
    buf = 0;
    return c;
}

void ungetch(int c){
    if(buf != 0)
        printf("ungetch: too many characters\n");
    else
        buf = c;
}
```  
因为最多只压入一个字符，所以只需要一个字符即可。
## 练习4-9  
### 问题描述  
以上介绍的getch和ungetch函数不能正确处理压回的EOF。考虑压回EOF时应该如何处理？请实现你的设计方案。  
### 问题解答  
``` C
#include<stdio.h>

#define BUFSIZE 100

int buf[BUFSIZE];
int bufp = 0;

int getch(void){
    return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c){
    if(bufp >= BUFSIZE)
        printf("ungetch: too many characters\n");
    else
        buf[bufp++] = c;
}
```  
原来的输入缓冲区被声明为字符数组，但是在C语言中，当 `char` 类型和 `int` 类型相互转化时，由于两种类型长度不同，需要补位或截取，就有可能出错。比如十进制的-1在16位机器上被表示为十六进制的 `0xFFFF` ，当把 `0xFFFF` 保存到一个 `char` 类型变量中，实际被保存的数字是 `0xFF`。而当将其再转换为 `int` 类型时，它可能被转换为 `0x00FF`(高位补0，结果是十进制的255)，就和初始的-1不相等了。所以这里为了正确处理 `EOF` ，需要将缓冲区声明为整形数组。  

## 练习4-10  
### 问题描述  
另一种方法是通过getline函数读入整个输入行，这种情况下可以不使用getch与ungetch函数。请运用这一方法修改计算器程序。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

#define MAXLINE 100
#define NUMBER '0'

static int getline(char line[], int limit);

static int li = 0;// 输入长度
char line[MAXLINE];

int getop(char s[]){
    int c, i;
    if(line[i] == '\0'){
        if(getline(line, MAXLINE) == 0)
            return EOF;
        else
            li = 0;
    }
    while((s[0] = c = line[li++]) == ' ' || c == '\t')
        ;
    s[i] = '\0';
    if(!isdigit(c) && c != '.')
        return c;// 不是数
    i = 0;
    if(isdigit(c))
        while(isdigit(s[++i] = c = line[li++]))
            ;
    if(c == '.')
        while(isdigit(s[++i] = c = line[li++]))
            ;
    s[i] = '\0';
    li--;
    return NUMBER;
}
```  
利用 `li` 来读取 `line` 中的一个字节，然后对 `li` 递增。在函数末尾，不需要调用ungetch函数把一个字符重新压回输入缓冲区，只需要对变量 `li` 减1就能达到后退一个字符的目的。  

## 练习4-11  
### 问题描述  
修改getop函数，使其不必使用ungetch函数。提示：可以使用一个 `static` 类型的内部变量解决该问题。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

#define NUMBER '0'

int getch(void);

int getop(char s[]){
    int c, i;
    static int lastc = 0;// 作为缓存
    if(lastc == 0)
        c = getch();
    else{
        c = lastc;
        lastc = 0;
    }
    while((s[0] = c) == ' ' || c == '\t')
        c = getch();
    s[1] = '\0';
    if(!isdigit(c) && c != '.')
        return c;
    i = 0;
    if(isdigit(c))
        while(isdigit(s[++i] = c = getch()))
            ;
    if(c == '.')
        while(isdigit(s[++i] = c = getch()))
            ;
    s[i] = '\0';
    if(c != EOF)
        lastc = c;
    return NUMBER;
}
```  

## 练习4-12  
### 问题描述  
运用printd函数的设计思想编写一个递归版本的itoa函数，即通过帝国调用把整数转换为字符串。  
### 问题解答  
``` C
#include<math.h>
void itoa(int n, char s[]){
    // 字符串索引，要声明为static，因为每次调用需要共享。
    static int i;
    if(n / 10)
        itoa(n / 10, s);
    else{
        i = 0;
        if(n < 0)
            s[i++] = '-';
    }
    s[i++] = abs(n) % 10 + '0';
    s[i] = '\0';
}
```  

## 练习4-13  
### 问题描述  
编写一个递归版本的reverse(s)函数，以将字符串s倒置。  
### 问题解答  
``` C
#include<string.h>
// 保持用户接口不变
void reverse(char s[]){
    void reverser(char s[], int i, int len);
    reverser(s, 0, strlen(s));
}
// 递归倒置字符串
void reverser(char s[], int i, int len){
    int c, j;
    j = len - (i + 1);
    if(i < j){
        c = s[j];
        s[j] = s[i];
        s[i] = c;
        reverser(s, ++i, len);
    }
}
```  

## 练习4-14  
### 问题描述  
定义宏swap(t, x, y)，以交换t类型的两个参数。  
### 问题解答  
``` C
#define swap(t, x, y) { t _z;   \
                        _z = y; \
                        y = x;  \
                        x = _z; }
```  
使用花括号定义一个程序块，使用 `\` 可以实现换行。这里应保证所声明的 `t` 类型临时变量不与 `x` 、 `y` 命名冲突，所以将其命名为库变量的标准格式（以 `_` 开头）。当然这并不能解决命名冲突问题，在使用时应注意，变量名不要是 `_z`。  
