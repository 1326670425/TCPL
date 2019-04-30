# 第四章 函数与程序结构  

### 本章计算器程序所涉及的框架程序  

逆波兰计算器主体  
```C
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

栈操作函数  
```C
#define MAXVAL 100
int sp = 0;
double val[MAXVAL];
// 压栈
void push(double f){
    if(sp < MAXVAL)
        val[sp++] = f;
    else
        printf("error: stack full, can't push %g\n", f);
}
// 出栈
double pop(void){
    if(sp > 0)
        return val[--sp];
    else{
        printf("error: stack empty\n");
        return 0.0;
    }
}
```  
获取操作符和操作数的getop函数  
```C
#include<ctype.h>

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
    if(!isdigit(c) && c != '.')
        return c;
    i = 0;
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
获取字符getch函数和回退字符ungetch函数  
```C
#define BUFSIZE 100

char buf[BUFSIZE];
int bufp = 0;

int getch(void){
    // 如果缓冲区非空，则返回缓冲区内字符。
    // 如果缓冲区为空，则调用getchar函数获取。
    return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c){
    if(bufp >= BUFSIZE)
        printf("ungetch: too many characters\n");
    else
        buf[bufp++] = c;
}
```  

- 使用 `static` 声明限定外部变量和函数，可以将其后声明的对象的作用域限定为被编译源文件的剩余部分。通过 `static` 限定外部对象，可以达到隐藏外部对象的目的。
  
  `static` 类型的内部变量是一种只能在某个特定函数内使用但一直占据存储空间的变量。  

- `register` 声明告诉编译器，它所声明的变量在程序中的使用频率较高。其思想是：将register变量放在机器的寄存器中，这样可以使程序更小、执行速度更快。但编译器可以忽略此项。  
  
  `register` 声明适用于自动变量以及函数的形式参数。  

- 对于外部变量和静态变量，初始化表达式必须是常量表达式，且只初始化一次。对于自动变量和寄存器变量，每次进入函数或者程序块都将被初始化。  
  
  不显式初始化的情况下，外部变量和静态变量都被初始化为0，自动变量和寄存器变量的初值则没有定义。  

- 宏定义时注意加括号， 不然在宏扩展时可能出错。  

- 使用 `#undef` 指令取消名字的宏定义。  

- 形式参数不能用带引号的字符串替换。但是，如果在替换文本中，参数名以 `#` 作为前缀则结果将被扩展为由实际参数替换该参数的带引号的字符串。例如，可以将它与字符串连接运算结合起来编写一个调试打印宏：  

  &emsp;&emsp; `#define dprint(expr) printf(#expr " = %g\n", expr)`  

  使用语句
  
  &emsp;&emsp; `dprint(x/y);`  
  
  调用该宏时，该宏被扩展为：
  
  &emsp;&emsp; `printf("x/y" " = %g\n", x/y);`  

- 预处理运算符 `##` 为宏扩展提供了一种连接实际参数的手段。如果替换文本中的参数与 `##` 相邻，则该参数将被实际参数替换， `##` 与前后的空白符将被删除，并对替换后的结果重新扫描。如，下列定义的宏paste用于连接两个参数：  

  &emsp;&emsp; `#define paste(front, back) front ## back`  

  则宏调用 `paste(name, 1)` 被扩展为 `name1` 。

- 条件包含。可以避免多次重复包含同一文件。  
