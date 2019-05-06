# 第五章 指针与数组  

## 练习5-1  
### 问题描述  
在上面的例子中，如果符号`+`或`-`的后面紧跟的不是数字，getint函数将把符号看做为数字0的有效表达式。修改该函数，将这种形式的`+`或`-`重新压回输入缓冲区。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

int getch(void);
void ungetch(int);

int getint(int *pn){
    int c, d, sign;
    // 跳过前导空格
    while(isspace(c = getch()))
        ;
    if(!isdigit(c) && c != EOF && c != '+' && c != '-'){
        ungetch(c);
        return 0;
    }
    sign = (c == '-') ? -1 : 1;
    if(c == '+' || c == '-'){
        d = c;
        // 正负号后边不是数字
        if(!isdigit(c = getch())){
            if(c != EOF)
                ungetch(c);// 压回预读取的字符
            ungetch(d);// 压回符号字符
            return d;
        }
    }
    for(*pn = 0; isdigit(c); c = getch())
        *pn = 10 * *pn + c - '0';
    *pn *= sign;
    if(c != EOF)
        ungetch(c);
    return c;
}
```  

## 练习5-2  
### 问题描述  
模仿函数getint的实现方法，编写一个读取浮点数的getfloat函数。getfloat函数的返回值应该是什么类型？  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

int getch(void);
void ungetch(int);

int getfloat(float *pn){
    int c, sign;
    float power;
    // 跳过前导空格
    while(isspace(c = getch()))
        ;
    if(!isdigit(c) && c != EOF && c != '+' && c != '-'){
        ungetch(c);
        return 0;
    }
    sign = (c == '-') ? -1 : 1;
    // 整数部分
    for(*pn = 0.0; isdigit(c); c = getch())
        *pn = 10.0 * *pn + c - '0';
    
    if(c == '.')
        c = getch();
    
    for(power = 1.0; isdigit(c); c = getch()){
        *pn = 10.0 * *pn + c - '0';
        power *= 10.0;
    }
    *pn *= sign / power;
    if(c != EOF)
        ungetch(c);
    return c;
}
```  
getfloat函数返回EOF或者紧跟在浮点数后边的那个字符的ASCII值。因此函数返回值应该是`int`。  

## 练习5-3  
### 问题描述  
用指针方式实现第2章中的函数strcat。函数strcat(s, t)将t指向的字符串复制到s指向的字符串的尾部。  
### 问题解答  
``` C
void strcat(char *s, char *t){
    // 找到s的末尾
    while(*s)
        s++;
    while(*s++ = *t++)
        ;
}
```  

## 练习5-4  
### 问题描述  
编写函数strend(s, t)。如果字符串t出现在字符串s的尾部，该函数返回1；否则返回0。  
### 问题解答  
``` C
int strend(char *s, char *t){
    char *bs = s;
    char *bt = t;
    // 遍历找到s、t的末尾
    for(; *s; ++s)
        ;
    for(; *t; ++t)
        ;

    for(; *s == *t; --s, --t){
        if(t == bt || s == bs)
            break;
    }
    if(*s == *t && t == bt && *s != '\0')
        return 1;
    else
        return 0;
}
```  

## 练习5-5  
### 问题描述  
实现库函数strncpy、strncat和strncmp，它们最多对参数字符串中的前n个字符进行操作。  
### 问题解答  
``` C
void strncpy(char *s, char *t, int n){
    while(*t && n-- > 0)
        *s++ = *t++;
    // t的长度小于n，给s尾部填充'\0'字符
    while(n-- > 0)
        *s++ = '\0';
}

void strncat(char *s, char *t, int n){
    void strncpy(char *s, char *t, int n);
    int strlen(char *s);
    strncpy(s + strlen(s), t, n);
}

int strncmp(char *s, char *t, int n){
    for(; *s == *t; ++s, ++t){
        if(*s == '\0' || --n <= 0)
            return 0;
    }
    return *s - *t;
}
```  

## 练习5-6  
### 问题描述  
采用指针而非数组索引方式改写前面章节和练习中的某些程序，例如getline、atoi、itoa以及它们的变体形式，reverse、strindex、getop等等。  
### 问题解答  
不复杂，懒得写了...  

## 练习5-7  
### 问题描述  
重写函数readlines，将输入的文本行存储到由main函数提供的一个数组中，而不是存储到调用alloc分配的存储空间中。该函数的运行速度比改写前快多少？  
### 问题解答  
``` C
#include<string.h>

#define MAXLEN 1000
#define MAXSTOR 5000

int getline(char *s, int n);

int readlines(char *lineptr[], char *linestor, int maxlines){
    int len, nlines;
    char line[MAXLEN];
    char *p = linestor;
    char *linestop = linestor + MAXSTOR;
    nlines = 0;
    while((len = getline(line, MAXLEN)) > 0){
        if(nlines >= maxlines || p + len > linestop)
            return -1;
        else{
            line[len - 1] = '\0'; // 删除换行符
            strcpy(p, line);
            lineptr[nlines++] = p;
            p += len;
        }
    }
    return nlines;
}
```  
readlines将读取的输入行保存在主函数提供的数组 `linestor` 中。字符指针 `p` 的初值指向 `linestor` 的第一个元素。  

这个版本的readlines函数因为不去频繁申请新的存储空间，速度会比原始版本稍快。  

## 练习5-8  
### 问题描述  
函数day_of_year和month_data中没有进行错误检查，请解决该问题。  
### 问题解答  
``` C
static char daytab[2][13] = {
    {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},
    {0, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
};

int day_of_year(int year, int month, int day){
    int i, leap;
    leap = year % 4 == 0 && year % 100 != 0 || year % 400 == 0;
    if(month < 1 || month > 12)
        return -1;
    if(day < 1 || day > daytab[leap][month])
        return -1;
    for(i = 1; i < month; ++i)
        day += daytab[leap][i];
    return day;
}

void month_day(int year, int yearday, int *month, int *day){
    int i, leap;
    if(year < 1){
        *month = -1;
        *day = -1;
        return;
    }
    leap = year % 4 == 0 && year % 100 != 0 || year % 400 == 0;
    for(i = 1; i <= 12 && yearday > daytab[leap][i]; ++i)
        yearday -= daytab[leap][i];
    if(i > 12 && yearday > daytab[leap][12]){
        *month = -1;
        *day = -1;
    }else{
        *month = i;
        *day = yearday;
    }
}
```  

## 练习5-9  
### 问题描述  
用指针方式代替数组下标方式，改写函数day_of_year和month_day。  
### 问题解答  
``` C
static char daytab[2][13] = {
    {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},
    {0, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
};

int day_of_year(int year, int month, int day){
    int leap;
    char *p;
    leap = year % 4 == 0 && year % 100 != 0 || year % 400 == 0;
    p = daytab[leap];
    if(month < 1 || month > 12)
        return -1;
    if(day < 1 || day > *(p+month))
        return -1;
    while(--month)
        day += *++p;// 使用前缀自增，跳过下标0
    return day;
}

void month_day(int year, int yearday, int *month, int *day){
    int leap;
    char *p;
    if(year < 1){
        *month = -1;
        *day = -1;
        return;
    }
    leap = year % 4 == 0 && year % 100 != 0 || year % 400 == 0;
    p = daytab[leap];
    while(yearday > *++p)
        yearday -= *p;
    if(i > 12 && yearday > *(p + 12)){
        *month = -1;
        *day = -1;
    }else{
        *month = p - *(daytab + leap);
        *day = yearday;
    }
}
```  

## 练习5-10  
### 问题描述  
编写程序expr，以计算从命令行输入的逆波兰表达式的值，其中每个运算符或操作数用一个单独的参数表示。例如，命令  
&emsp;&emsp; `expr 2 3 4 + *`  
将计算表达式 `2*(3+4)` 的值。  
### 问题解答  
``` C
#include<stdio.h>
#include<math.h>

#define MAXOP 100
#define NUMBER '0'

int getop(char s[]);
void ungets(char s[]);
void push(double x);
double pop(void);

int main(int argc, char *argv[]){
    char s[MAXOP];
    double op2;
    while(--argc > 0){
        ungets(" ");// 将结束标志放入缓冲区
        ungets(*++argv);// 将参数放入缓冲区
        switch(getop(s)){
        case NUMBER:
            push(atof(s));
            break;
        case '+':
            push(pop() + pop());
            break;
        case '-':
            op2 = pop();
            push(pop() - op2);
            break;
        case '*':
            push(pop() * pop());
            break;
        case '/':
            op2 = pop();
            if(op2 != 0.0)
                push(pop() / op2);
            else
                printf("error: zero divisor\n");
            break;
        default:
            printf("error: unknown command %s\n", s);
            // 循环的时候，自减操作使得条件为假退出循环
            argc = 1;
            break;
        }
    }
    printf("\t%.8g\n", pop());
    return 0;
}
```  

## 练习5-11  
### 问题描述  
修改程序entab和detab，使它们接受一组作为参数的制表符停止位，如果启动程序时不带参数，则使用默认的制表符停止位设置。  
### 问题解答  
``` C
#include<stdio.h>

#define MAXLINE 100
#define TABINC 8
#define YES 1
#define NO 0

void settab(int argc, char *argv[], char *tab);
void entab(char *tab);
int tabpos(int pos, char *tab);

int main(int argc, char *argv[]){
    char tab[MAXLINE + 1];

    settab(argc, argv, tab);// 初始化制表符停止位设置
    entab(tab);
    return 0;
}

void entab(char *tab){
    int c, pos;
    int nb = 0;
    int nt = 0;
    for(pos = 1; (c = getchar()) != EOF; ++pos){
        if(c == ' '){
            if(tabpos(pos, tab) == NO)
                ++nb;
            else{
                nb = 0
                ++nt;
            }
        }else{
            for(; nt > 0; --nt)
                putchar('\t');
            if(c == '\t')
                nb = 0;
            else
                for(; nb > 0; --nb)
                    putchar(' ');
            putchar(c);
            if(c == '\n')
                pos = 0;
            else if(c == '\t')
                while(tabpos(pos, tab) != YES)
                    ++pos;
        }
    }
}
```  
settab.c代码：  
``` C
#include<stdlib.h>

#define MAXLINE 100
#define TABINC 8
#define YES 1
#define NO 0

void settab(int argc, char *argv[], char *tab){
    int i, pos;
    if(argc <= 1){// 默认制表符停止位设置
        for(i = 1; i <= MAXLINE; ++i){
            if(i % TABINC == 0)
                tab[i] = YES;
            else
                tab[i] = NO;
        }
    }else{// 自设定停止位
        for(i = 1; i <= MAXLINE; ++i)
            tab[i] = NO;
        while(--argc > 0){
            pos = atoi(*++argv);
            if(pos > 0 && pos <= MAXLINE)
                tab[pos] = YES;
        }
    }
}
```  
tabpos.c代码：
``` C
#define MAXLINE 100
#define YES 1
// 查看当前位是不是停止位
int tabpos(int pos, char *tab){
    if(pos > MAXLINE)
        return YES;
    return tab[pos];
}
```  

数组 `tab` 每个元素对应文本行中的一个位置，如果某个位置处有一个制表符停止位，与之对应的 `tab[i]` 就取值为 `YES`，否则为 `NO`。  

detab函数相关：  
```C
#include<stdio.h>

#define MAXLINE 100
#define TABINC 8
#define YES 1
#define NO 0

void settab(int argc, char *argv[], char *tab);
void detab(char *tab);
int tabpos(int pos, char *tab);

int main(int argc, char *argv[]){
    char tab[MAXLINE + 1];

    settab(argc, argv, tab);// 初始化制表符停止位设置
    detab(tab);
    return 0;
}
// 将制表符替换为空格
void detab(char *tab){
    int c, pos = 1;
    while((c = getchar()) != EOF){
        if(c == '\t'){
            do{
                putchar(' ');
            }while(tabpos(pos++, tab) != YES);
        }else if(c == '\n'){
            putchar(c);
            pos = 1;
        }else{
            putchar(c);
            ++pos;
        }
    }
}
```  

## 练习5-12  
### 问题描述  
对程序entab和detab做一些扩充，以接受下列缩写的命令：

&emsp;&emsp; entab -m +n

表示制表符从第m列开始，每隔n列停止。选择（对使用者而言）比较方便的默认行为。  
### 问题解答  
``` C
#include<stdio.h>

#define MAXLINE 100
#define TABINC 8
#define YES 1
#define NO 0

void esettab(int argc, char *argv[], char *tab);
void entab(char *tab);
int tabpos(int pos, char *tab);

int main(int argc, char *argv[]){
    char tab[MAXLINE + 1];

    esettab(argc, argv, tab);// 初始化制表符停止位设置
    entab(tab);
    return 0;
}
```  
esettab.c代码：  
``` C
#include<stdlib.h>

#define MAXLINE 100
#define TABINC 8
#define YES 1
#define NO 0

void esettab(int argc, char *argv[], char *tab){
    int i, inc, pos;
    if(argc <= 1){// 默认制表符停止位设置
        for(i = 1; i <= MAXLINE; ++i){
            if(i % TABINC == 0)
                tab[i] = YES;
            else
                tab[i] = NO;
        }
    }else if(argc == 3 && *argv[1] == '-' && *argv[2] == '+'){// 使用参数设定
        pos = atoi(&(*++argv)[1]);// 选择输入的m
        inc = atoi(&(*++argv)[1]);// 选择输入的n
        for(i = 1; i <= MAXLINE; ++i){
            if(i != pos)
                tab[i] = NO;
            else{
                tab[i] = YES;
                pos += inc;// pos递增inc，实现每隔inc停止
            }
        }
    }else{
        for(i = 1; i <= MAXLINE; ++i)
            tab[i] = NO;
        while(--argc > 0){
            pos = atoi(*++argv);
            if(pos > 0 && pos <= MAXLINE)
                tab[pos] = YES;
        }
    }
}
```  
detab相关：  
``` C
#include<stdio.h>

#define MAXLINE 100
#define TABINC 8
#define YES 1
#define NO 0

void settab(int argc, char *argv[], char *tab);
void detab(char *tab);
int tabpos(int pos, char *tab);

int main(int argc, char *argv[]){
    char tab[MAXLINE + 1];

    settab(argc, argv, tab);// 初始化制表符停止位设置
    detab(tab);
    return 0;
}
```  

## 练习5-13  
### 问题描述  
编写程序tail，将其输入中的最后n行打印出来。默认情况下，n的值为10，但可以通过一个可选的参数改变n的值，因此，命令  

&emsp;&emsp; tail -n  

将打印其输入的最后n行。无论输入或n的值是否合理，该程序都应该能正常运行。编写的程序要充分利用存储空间。  
### 问题解答  
``` C
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

#define DEFLINES  10
#define LINES 100//最大输入行数
#define MAXLEN 100

void error(char *p);
int getline(char *line, int n);

int main(int argc, char *argv[]){
    char *p, *buf, *bufend;
    char line[MAXLEN];//  当前输入行
    char *lineptr[LINES]; // 记录每行输入
    int first, i, last, len, n, nlens;

    if(argc == 1)
        n = DEFLINES;// 默认n
    else if(argc == 2 && (*++argv)[0] == '-')
        n = atoi(argv[0] + 1);// 因为判断条件中，argv已自增，所以这里为argv[0]
    else// 如果参数不是两个，或者第二个参数不以'-'开头，则报错
        error("usage: tail [-n]");
    // 不合法n的处理
    if(n < 1 || n > LINES)
        n = LINES;
    // 初始化指针数组
    for(i = 0; i < LINES; ++i)
        lineptr[i] = NULL;
    if((p = buf = malloc(LINES * MAXLEN)) == NULL)
        error("tail: can't allocate buf");
    bufend = buf + LINES * MAXLEN;
    last = 0;
    nlines = 0;
    while((len = getline(line, MAXLEN) > 0)){
        // 如果缓冲区空间不够，则重置p为缓冲区开头
        if(p + len + 1 >= bufend)
            p = buf;
        lineptr[last] = p;
        strcpy(lineptr[last], line);
        // 如果last达到了LINES，则重置last
        if(++last >= LINES)
            last = 0;
        p += len + 1;
        nlines++;// 总行数递增
    }
    if(n > nlines)
        n = nlines;
    first = last - n;
    if(first < 0)
        first += LINES;
    // 取模运算使得i保持在0到LINES-1之间
    for(i = first; n-- > 0; i = (i + 1) & LINES)
        printf("%s", lineptr[i]);
    return 0;
}

void error(char *s){
    printf("%s\n", s);
    exit(1);
}
```  

## 练习5-14  
### 问题描述  
修改排序程序，使它能够处理`-r`标记。该标记表明，以逆序（递减）方式排序。要保证`-r`和`-n`能够组合在一起使用。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>

#define NUMBERIC 1
#define DECR 2
#define LINES 100

int numcmp(char s[], char t[]);
int readlines(char *lineptr[], int maxlines);
void qsort(char *v[], int left, int right,
           int (*comp)(void *, void *));
void writelines(char *lineptr[], int nlines, int decr);

static char option = 0;

int main(int argc, char *argv[]){
    char *lineptr[LINES];
    int nlines;
    int c;

    while(--argc > 0 && (*++argv)[0] == '-'){
        // 遍历命令列表
        while(c = *++argv[0]){
            switch(c){
            case 'n':// 按数字排序
                option |= NUMBERIC;
                break;
            case 'r':
                option |= DECR;
                break;
            default:
                printf("sort: illegal option %c\n", c);
                break;
            }
        }
    }
    if(argc)
        printf("Usage: sort -nr \n");
    else{
        if((nlines = readline(lineptr, LINES)) > 0){
            if(option & NUMBERIC)
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))numcmp);
            else
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))strcmp);
            writelines(lineptr, nlines, option & DECR);
        }else{
            printf("input too bit to sort\n");
            return -1;
        }
    }
    return 0;
}

void writelines(char *lineptr[], int nlines, int decr){
    int i;
    if(decr){
        for(i = nlines - 1; i >= 0; --i){
            printf("%s\n", lineptr[i]);
        }
    }else
        for(i = 0; i < nlines; ++i)
            printf("%s\n", lineptr[i]);
}
```  

## 练习5-15  
### 问题描述  
增加一个选项 `-f` ，使得排序过程不考虑字母大小写之间的区别。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>

#define NUMBERIC 1
#define DECR 2
#define LINES 100
#define FOLD 4

int charcmp(char s[], char t[]);
int numcmp(char s[], char t[]);
int readlines(char *lineptr[], int maxlines);
void qsort(char *v[], int left, int right,
           int (*comp)(void *, void *));
void writelines(char *lineptr[], int nlines, int decr);

static char option = 0;

int main(int argc, char *argv[]){
    char *lineptr[LINES];
    int nlines;
    int c;

    while(--argc > 0 && (*++argv)[0] == '-'){
        // 遍历命令列表
        while(c = *++argv[0]){
            switch(c){
            case 'f':
                option |= FOLD;
                break;
            case 'n':// 按数字排序
                option |= NUMBERIC;
                break;
            case 'r':
                option |= DECR;
                break;
            default:
                printf("sort: illegal option %c\n", c);
                break;
            }
        }
    }
    if(argc)
        printf("Usage: sort -fnr \n");
    else{
        if((nlines = readline(lineptr, LINES)) > 0){
            if(option & NUMBERIC)
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))numcmp);
            else if(option & FOLD)
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))charcmp);
            else
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))strcmp);
            writelines(lineptr, nlines, option & DECR);
        }else{
            printf("input too bit to sort\n");
            return -1;
        }
    }
    return 0;
}
// 忽略大小写
int charcmp(char s[], char t[]){
    for(; tolower(*s) == tolower(*t); ++s, ++t)
        if(*s == '\0')
            return 0;
    return tolower(*s) - tolower(*t);
}
```  

## 练习5-16  
### 问题描述  
增加选项 `-d` （代表目录顺序）。该选项表明，只对字母、数字和空格进行比较。要保证该选项可以与 `-f` 组合在一起使用。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>

#define NUMBERIC 1
#define DECR 2
#define LINES 100
#define FOLD 4
#define DIR 8

int charcmp(char s[], char t[]);
int numcmp(char s[], char t[]);
int readlines(char *lineptr[], int maxlines);
void qsort(char *v[], int left, int right,
           int (*comp)(void *, void *));
void writelines(char *lineptr[], int nlines, int decr);

static char option = 0;

int main(int argc, char *argv[]){
    char *lineptr[LINES];
    int nlines;
    int c;

    while(--argc > 0 && (*++argv)[0] == '-'){
        // 遍历命令列表
        while(c = *++argv[0]){
            switch(c){
            case 'd':
                option |= DIR;
                break;
            case 'f':
                option |= FOLD;
                break;
            case 'n':// 按数字排序
                option |= NUMBERIC;
                break;
            case 'r':
                option |= DECR;
                break;
            default:
                printf("sort: illegal option %c\n", c);
                break;
            }
        }
    }
    if(argc)
        printf("Usage: sort -dfnr \n");
    else{
        if((nlines = readline(lineptr, LINES)) > 0){
            if(option & NUMBERIC)
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))numcmp);
            else
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))charcmp);
            writelines(lineptr, nlines, option & DECR);
        }else{
            printf("input too bit to sort\n");
            return -1;
        }
    }
    return 0;
}

int charcmp(char s[], char t[]){
    char a, b;
    int fold = (option & FOLD) ? 1 : 0;
    int dir = (option & DIR) ? 1 : 0;

    do{
        if(dir){
            while(!isalnum(*s) && *s != ' ' && *s != '\0')
                s++;
            while(!isalnum(*t) && *t != ' ' && *t != '\0')
                t++;
        }
        a = fold ? tolower(*s) : *s;
        s++;
        b = fold ? tolower(*t) : *t;
        t++;
        if(a == b && a == '\0')
            return 0;
    }while(a == b);
    return a - b;
}
```  

## 练习5-17  
### 问题描述  
增加字段处理功能，以使得排序程序可以根据行内的不同字段进行排序，每个字段按照一个单独的选项集合进行排序。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>

#define NUMBERIC 1
#define DECR 2
#define FOLD 4
#define DIR 8
#define LINES 100

int charcmp(char s[], char t[]);
void error(char s[]);
int numcmp(char s[], char t[]);
void readargs(int argc, char *argv[]);
int readlines(char *lineptr[], int maxlines);
void qsort(char *v[], int left, int right,
           int (*comp)(void *, void *));
void writelines(char *lineptr[], int nlines, int decr);

char option = 0;
int pos1 = 0;// 定位字段开始位置
int pos2 = 0;// 定位字段结束位置


int main(int argc, char *argv[]){
    char *lineptr[LINES];
    int nlines;
    int c;
    
    readargs(argc, argv);
    
    if(argc)
        printf("Usage: sort -dfnr \n");
    else{
        if((nlines = readline(lineptr, LINES)) > 0){
            if(option & NUMBERIC)
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))numcmp);
            else
                qsort((void *)lineptr, 0, nlines - 1,
                      (int (*)(void *, void *))charcmp);
            writelines(lineptr, nlines, option & DECR);
        }else{
            printf("input too bit to sort\n");
            return -1;
        }
    }
    return 0;
}

void readargs(int argc, char *argv[]){
    int c;
    int atoi(char *);
    while(--argc > 0 && (*++argv)[0] == '-' || c == '+'){
        if(c == '-' && !isdigit(*(argv[0] + 1))){
            // 遍历命令列表
            while(c = *++argv[0]){
                switch(c){
                case 'd':
                    option |= DIR;
                    break;
                case 'f':
                    option |= FOLD;
                    break;
                case 'n':// 按数字排序
                    option |= NUMBERIC;
                    break;
                case 'r':
                    option |= DECR;
                    break;
                default:
                    printf("sort: illegal option %c\n", c);
                    error("Usage: sort -dfnr [+pos1] [-pos2]");
                    break;
                }
            }
        }else if(c == '-')// 遇到可选的字段排序选项
            pos2 = atoi(argv[0] +  1);
        else if((pos1 = atoi(argv[0] + 1)) < 0)
            error("Usage: sort -dfnr [+pos1] [-pos2]");
    }
}
```  
numcmp.c代码：  
``` C
#include<math.h>
#include<ctype.h>
#include<string.h>

#define MAXSTR 100

void substr(char s[], char t[], int maxstr);

int numcmp(char s1[], char s2[]){
    double v1, v2;
    char str[MAXSTR];
    
    substr(s1, str);
    v1 = atof(str);
    substr(s2, str);
    v2 = atof(str);
    if(v1 < v2)
        return -1;
    else if(v1 > v2)
        return 1;
    else
        return 0;
}


int charcmp(char s[], char t[]){
    char a, b;
    int fold = (option & FOLD) ? 1 : 0;
    int dir = (option & DIR) ? 1 : 0;
    int i, j, endpos;
    extern char option;
    extern int pos1, pos2;
    i = j = pos1;
    if(pos2 > 0)
        endpos = pos2;
    else if((endpos = strlen(s)) > strlen(t))
        endpos = strlen(t);
    do{
        if(dir){
            while(i < endpos && !isalnum(s[i]) && s[i] != ' ' && s[i] != '\0')
                i++;
            while(j < endpos && !isalnum(t[j]) && t[j] != ' ' && t[j] != '\0')
                j++;
        }
        if(i < endpos && j < endpos){
            a = fold ? tolower(s[i]) : s[i];
            i++;
            b = fold ? tolower(t[j]) : t[j];
            j++;
            if(a == b && a == '\0')
                return 0;
        }
    }while(a == b && i < endpos && j < endpos);
    return a - b;
}
```  
substr.c代码：  
``` C
#include<string.h>
void error(char s[]);

void substr(char s[], char str[]){
    int i, j, len;
    extern int pos1, pos2;
    len = strlen(s);
    if(pos2 > 0){
        if(len >= pos2)
            len = pos2;
        else
            error("substr: string to short");
    }
    for(j = 0, i = pos1; i < len; ++i, ++j)
        str[j] = s[i];
    str[j] = '\0';
}
```  

## 练习5-18  
### 问题描述  
修改dcl程序，使它能够处理输入中的错误。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>

enum {NAME, PARENS, BRACKETS};
enum {NO, YES};

void dcl(void);
void dirdcl(void);
void errmsg(char *s);
int gettoken(void);

extern int tokentype;// 最新读取的字符串的类型
extern char token[];//最新读取的字符串
extern char name[];
extern char out[];
extern int prevtoken;

void dcl(void){
    int ns;
    // 统计'*'
    for(ns = 0; gettoken() == '*'; ){
        ns++;
    }
    dirdcl();
    while(ns-- > 0)
        strcat(out, " pointer to");
}

void dirdcl(void){
    int type;
    
    if(tokentype == '('){
        dcl();
        if(tokentype != ')')
            errmsg("error: missing )\n");
    }else if(tokentype == NAME)
        strcpy(name, token);
    else
        errmsg("error: expected name or (dcl)\n");
    while((type = gettoken()) == PARENS || type == BRACKETS){
        if(type == PARENS)
            strcat(out, " function returning");
        else{
            strcat(out, " array");
            strcat(out, token);
            strcat(out, " of");
        }
    }
}

void errmsg(char *msg){
    printf("%s", msg);
    // 通知gettoken已经读入一个记号了
    prevtoken = YES;
}
```  
gettoken.c代码：  
``` C
#include<string.h>
#include<ctype.h>

enum {NAME, PARENS, BRACKETS};
enum {NO, YES};


extern int tokentype;// 最新读取的字符串的类型
extern char token[];//最新读取的字符串
int prevtoken = NO;// 是否有上个token

int gettoken(void){
    int c, getch(void);
    void ungetch(int c);
    char *p = token;
    if(prevtoken == YES){
        prevtoken = NO;
        return tokentype;
    }
    while((c = getchar()) == ' ' || c == '\t')
        ;
    if(c == '('){
        if((c = getch()) == ')'){
            strcpy(token, "()");
            return tokentype = PARENS;
        }else{
            ungetch(c);
            return tokentype = '(';
        }
    }else if(c == '['){
        for(*p++ = c; (*p++ = getch()) != ']'; )
            ;
        *p = '\0';
        return tokentype = BRACKETS;
    }else if(isalpha(c)){
        for(*p++ = c; isalnum(c = getchar()); )
            *p++ = c;
        *p = '\0';
        ungetch(c);// 要把最后取到的，不是字母或数字的字符回退到输入缓冲区
        return tokentype = NAME;
    }else
        return tokentype = c;
}
```  

## 练习5-19  
### 问题描述  
修改undcl程序，使它在把文字描述转为声明的过程中不会生成多余的圆括号。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>
#include<string.h>

#define MAXTOKEN 100

enum { NAME, PARENS, BRACKETS};

void dcl(void);
void dirdcl(void);
int gettoken(void);
int nexttoken(void);

int tokentype;
char token[MAXTOKEN];
char out[1000];

int undcl(){
    int type;
    char temp[MAXTOKEN];
    
    while(gettoken() != EOF){
        strcpy(out, token);
        while((type = gettoken) != '\n'){
            if(type == PARENS || type == BRACKETS)
                strcat(out, token);
            else if(type == '*'){
                if((type = nexttoken()) == PARENS || type == BRACKETS)
                    sprintf(temp, "(*%s)", out);
                else
                    sprintf(temp, "*%s", out);
                strcpy(out, temp);
            }else if(type == NAME){
                sprintf(temp, "%s %s", token, out);
                strcpy(out, temp);
            }else
                printf("invalid input at %s\n", token);
        }
        printf("5s\n", out);
    }
    return 0;
}

enum {NO, YES};

int gettoken(void);

int nexttoken(void){
    int type;
    extern int prevtoken;
    type = gettoken();
    prevtoken = YES;
    return type;
}
```  
只有当下个记号是 `()` 或 `[]` 时，undcl程序才有必要在自己的输出结果中使用括号。  

## 练习5-20  
### 问题描述  
扩展dcl程序的功能，使它能够处理包含其它成分的声明，例如带有函数参数类型的声明、带有类似const限定符的声明等。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>

enum {NAME, PARENS, BRACKETS};
enum {NO, YES};

void dcl(void);
void dirdcl(void);
void errmsg(char *s);
int gettoken(void);

extern int tokentype;// 最新读取的字符串的类型
extern char token[];//最新读取的字符串
extern char name[];
extern char datatype[];// 数据类型，char、int等
extern char out[];
extern int prevtoken;

void dcl(void){
    int ns;
    // 统计'*'
    for(ns = 0; gettoken() == '*'; ){
        ns++;
    }
    dirdcl();
    while(ns-- > 0)
        strcat(out, " pointer to");
}

void dirdcl(void){
    int type;
    void parmdcl(void);
    if(tokentype == '('){
        dcl();
        if(tokentype != ')')
            errmsg("error: missing )\n");
    }else if(tokentype == NAME)
        if(name[0] == '\0')
            strcpy(name, token);
    else
        errmsg("error: expected name or (dcl)\n");
    while((type = gettoken()) == PARENS || type == BRACKETS || type == '('){
        if(type == PARENS)
            strcat(out, " function returning");
        else if(type == '('){
            strcat(out, " function expecting");
            // 分析参数
            parmdcl();
            strcat(out, " and returning");
        }else{
            strcat(out, " array");
            strcat(out, token);
            strcat(out, " of");
        }
    }
}

void errmsg(char *msg){
    printf("%s", msg);
    // 通知gettoken已经读入一个记号了
    prevtoken = YES;
}
```  
parmdcl.c代码：  
``` C
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<ctype.h>

#define MAXTOKEN 100

enum {NAME, PARENS, BRACKETS};
enum {NO, YES};

void dcl(void);
void dirdcl(void);
void errmsg(char *s);
void dclspec(void);
int typespec(void);
int typequal(void);
int compare(char **s. char **t);
int gettoken(void);

extern int tokentype;// 最新读取的字符串的类型
extern char token[];//最新读取的字符串
extern char name[];
extern char datatype[];// 数据类型，char、int等
extern char out[];
extern int prevtoken;

void parmdcl(void){
    do{
        dclspec();
    }while(tokentype == ',');
    if(tokentype != ')')
        errmsg("missing ) in parameter declaration\n");
}
// declaration specification
void dclspec(void){
    char temp[MAXTOKEN];
    temp[0] = '\0';
    gettoken();
    do{
        if(tokentype != NAME){
            prevtoken = YES;
            dcl();
        }else if(typespec() == YES){
            strcat(temp, " ");
            strcat(temp, token);
            gettoken();
        }else if(typequal() == YES){
            strcat(temp, " ");
            strcat(temp, token);
            gettoken();
        }else
            errmsg("unknown type in parameter list\n");
    }while(tokentype != ',' && tokentype != ')');
    strcat(out, temp);
    if(tokentype == ',')
        strcat(out, ",");
}
// return YES if token is a type-specifier
int typespec(void){
    static char *types[] = {
        "char",
        "int",
        "void"
    };
    char *pt = token;
    // 折半查找
    if(bsearch(&pt, types, sizeof(types) / sizeof(char *), sizeof(char *), compare) == NULL)
        return NO;
    else
        return YES;
}
// return YES if token is a type-qualifier
int typequal(void){
    static char *typeq[] = {
        "const",
        "volatile",
        "extern"
    };
    char *pt = token;
    if(bsearch(&pt, typeq, sizeof(typeq) / sizeof(char *), sizeof(char *), compare) == NULL)
        return NO;
    else
        return YES;
}

int compare(char **s, char **t){
    return strcmp(*s, *t);
}
```  
