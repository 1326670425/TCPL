# 第七章 输入与输出  

## 练习7-1  
### 问题描述  
编写一个程序，根据它自身被调用时存放在argv[0]中的名字，实现将大写字母转换为小写字母或将小写字母转换为大写字母的功能。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>

int main(int argc, char *argv[]){
    int c;

    if(strcmp(argv[0], "lower") == 0)
        while((c = getchar()) != EOF)
            putchar(tolower(c));
    else
        while((c = getchar()) != EOF)
            putchar(toupper(c));
    return 0;
}
```  

## 练习7-2  
### 问题描述  
编写一个程序，以合理的方式打印任何输入。该程序至少能够根据用户的习惯以八进制或十六进制打印非图形字符，并截断长文本行。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

#define MAXLINE 100
#define OCTLEN 6

int main(){
    int c, pos;
    int inc(int pos, int n);

    pos = 0;
    while((c = getchar()) != EOF){
        if(iscntrl(c) || c == ' '){// 寻找非显示字符——删除控制符（0177）和普通控制字符（040）
            pos = inc(pos, OCTLEN);// 长字符截断
            printf(" \\%03o ", c);
            if(c == '\n'){
                pos = 0;
                putchar('\n');
            }
        }else{
            pos = inc(pos, 1);
            putchar(c);
        }
    }
    return 0;
}

int inc(int pos, int n){
    if(pos + n < MAXLINE)
        return pos + n;
    else{// 超过每行最大字符数，重新开始一行
        putchar('\n');
        return n;
    }
}
```  

## 练习7-3  
### 问题描述  
改写minprintf函数，使它能完成printf函数的更多功能。  
### 问题解答  
``` C
#include<stdarg.h>
#include<stdio.h>
#include<ctype.h>

#define LOCALFMT 100


void minprintf(char *fmt, ...){
    va_list ap;
    char *p, *sval;
    char localfmt[LOCALFMT];
    int i, ival;
    unsigned uval;
    double dval;

    va_start(ap, fmt);
    for(p = fmt; *p; ++p){
        if(*p != '%'){
            putchar(*p);
            continue;
        }
        // *p为'%'，则解析转换字符
        i = 0;
        localfmt[i++] = '%';
        while(*(p + 1) && !isalpha(*(p + 1)))// 读取转换格式
            localfmt[i++] = *++p;
        localfmt[i++] = *(p + 1);
        localfmt[i] = '\0';

        switch(*++p){
        case 'd':
        case 'i':
            ival = va_arg(ap, int);
            printf(localfmt, ival);
            break;
        case 'x':
        case 'X':
        case 'u':
        case 'o':
            uval = va_arg(ap, unsigned);
            printf(localfmt, uval);
            break;
        case 'f':
            dval = va_arg(ap, double);
            printf(localfmt, dval);
            break;
        case 's':
            sval = va_arg(ap, char *);
            printf(localfmt, sval);
            break;
        default:
            putchar(localfmt);
            break;
        }
    }
    va_end(ap);
}
```  

## 练习7-4  
### 问题描述  
类似于上一节中的函数minprintf，编写scanf函数的一个简化版本。  
### 问题解答  
``` C
#include<stdarg.h>
#include<stdio.h>
#include<ctype.h>

#define LOCALFMT 100


void minscanf(char *fmt, ...){
    va_list ap;
    char *p, *sval;
    char localfmt[LOCALFMT];
    int c, i, *ival;
    unsigned *uval;
    double *dval;

    i = 0;
    va_start(ap, fmt);
    for(p = fmt; *p; ++p){
        if(*p != '%'){
            localfmt[i++] = *p;
            continue;
        }
        localfmt[i++] = '%';// 转换格式开始

        while(*(p + 1) && !isalpha(*(p + 1)))// 读取转换格式
            localfmt[i++] = *++p;
        localfmt[i++] = *(p + 1);
        localfmt[i] = '\0';

        switch(*++p){
        case 'd':
        case 'i':
            ival = va_arg(ap, int *);
            scanf(localfmt, ival);
            break;
        case 'x':
        case 'X':
        case 'u':
        case 'o':
            uval = va_arg(ap, unsigned *);
            scanf(localfmt, uval);
            break;
        case 'f':
            dval = va_arg(ap, double *);
            scanf(localfmt, dval);
            break;
        case 's':
            sval = va_arg(ap, char *);
            scanf(localfmt, sval);
            break;
        default:
            scanf(localfmt);
            break;
        }
        i = 0;
    }
    va_end(ap);
}
```  

注意区别 printf，scanf输入为指针。  

## 练习7-5  
### 问题描述  
改写第4章中的后缀计算器程序，用scanf函数或sscanf函数实现输入以及数的转换。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

#define NUMBER '0'

int getop(char s[]){
    int c, i, rc;
    static char lastc[] = " ";// 保存上次的字符

    sscanf(lastc, "%c", &c);
    lastc[0] = ' ';// 清空最后一个字符
    while((s[0] = c) == ' ' || c == '\t')
        if(scanf("%c", &c) == EOF)
            c = EOF;
    s[1] = '\0';
    if(!isdigit(c) && c != '.')//不是数字或小数点
        return c;
    i = 0;
    // 读整数部分
    if(isdigit(c)){
        do{
            rc = scanf("%c", &c);
            if(!isdigit(s[++i] = c))
                break;
        }while(rc != EOF);
    }
    // 读小数部分
    if(c == '.'){
        do{
            rc = scanf("%c", &c);
            if(!isdigit(s[++i] = c))
                break;
        }while(rc != EOF);
    }
    s[i] = '\0';
    if(rc != EOF)
        lastc[0] = c;
    return NUMBER;
}
```  

解法二：  

``` C
#include<stdio.h>
#include<ctype.h>

#define NUMBER '0'

int getop(char s[]){
    int c, i, rc;
    float f;

    while((rc = scanf("%c", &c)) != EOF)
        if((s[0] = c) != ' ' || c != '\t')
            break;
    s[1] = '\0';
    if(rc == EOF)
        return EOF;
    else if(!isdigit(c) && c != '.')
        return c;
    // 如果读到数字或小数点，就先放回
    // 然后用scanf获取整个数字
    ungetc(c, stdin);
    scanf("%f", &f);
    // 将读到的浮点数转换为字符串形式
    sprintf(s, "%f", f);
    return NUMBER;
}
```  

## 练习7-6  
### 问题描述  
编写一个程序，比较两个文件并打印它们第一个不相同的行。  
### 问题解答  
``` C
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

#define MAXLINE 100

int main(int argc, char *argv[]){
    FILE *fp1, *fp2;
    void filecomp(FILE *fp1, FILE *fp2);
    // 三个参数，一个程序名，两个文件名
    if(argc == 3){
        fprintf(stderr, "comp: need two file names\n");
        exit(1);
    }else if((fp2 = fopen(*++argv, "r")) == NULL){
        fprintf(stderr, "comp: can't open %s\n". *argv);
        exit(1);
    }else{
        filecomp(fp1, fp2);
        fclose(fp1);
        fclose(fp2);
        exit(0);
    }
    return 0;
}

void filecomp(FILE *fp1, FILE *fp2){
    char line1[MAXLINE], line2[MAXLINE];
    char *lp1, *p2;

    do{
        lp1 = fgets(line1, MAXLINE, fp1);
        lp2 = fgets(line2, MAXLINE, fp2);
        if(lp1 == line1 && lp2 == line2){
            if(strcmp(line1, line2) != 0){
                printf("first difference in line\n%s\n", line1);
                lp1 = lp2 = NULL;
            }
        }else if(lp1 != line1 && lp2 == line2)
            printf("end of first file at line\n%s\n", line1);
        else if(lp1 == line1 && lp2 != line2)
            printf("end of first file at line\n%s\n", line2);
    }while(lp1 == line1 && lp2 == line2);
}
```  

## 练习7-7  
### 问题描述  
修改第5章的模式查找程序，使它从一个命名文件的集合中读取输入（有文件名参数时），如果没有文件名参数，则从标准输入中读取输入。当发现一个匹配行时，是否应该将相应的文件名打印出来？  
### 问题解答  
``` C
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

#define MAXLINE 1000

int main(int argc, char *argv[]){
    char pattern[MAXLINE];
    // except决定是选择匹配的还是选择不匹配的
    // number决定是否打印行号
    int c, except = 0, number = 0;
    FILE *fp;
    void fpat(FILE *fp, char *fname, char *pattern);
    // 解析命令参数
    while(--argc > 0 && (*++argv)[0] == '-'){
        while(c = *++argv[0]){
            switch(c){
            case 'x':
                except = 1;
                break;
            case 'n':
                number = 1;
                break;
            default:
                printf("find: illegal option %c\n", c);
                argc = 0;
                break;
            }
        }
    }
    if(argc >= 1)
        strcpy(pattern, *argv);
    else{
        printf("Usage: find [-x] [-n] pattern [file ...]\n");
        exit(1);
    }
    if(argc == 1)//没有文件名参数
        fpat(stdin, "", pattern, except, number);
    else{
        while(--argc > 0){ // 找文件名
            if((fp = fopen(*++argv, "r")) == NULL){
                fprintf(stderr, "find: can't open %s\n", *argv);
                exit(1);
            }else{
                fpat(fp, *argv, pattern, except, number);
                fclose(fp);
            }
        }
    }
    return 0;
}

void fpat(FILE *fp, char *fname, char *pattern, int except, int number){
    char line[MAXLINE];
    long lineno = 0;

    while(fgets(line, MAXLINE, fp) != NULL){
        ++lineno;
        if((strstr(line, pattern) != NULL) != except){
            if(*fname)
                printf("%s - ", fname);
            if(number)
                printf("%ld: ", lineno);
            printf("%s", line);
        }
    }
}
```  

## 练习7-8  
### 问题描述  
编写一个程序，以打印一个文件集合，每个文件从新的一页开始打印，并且打印每个文件相关的标题和页数。  
### 问题解答  
``` C
#include<stdio.h>
#include<stdlib.h>

#define MAXBOT 3 // 页底留白行数
#define MAXHDR 5 // 页头留白行数
#define MAXLINE 100 // 每行字符个数
#define MAXPAGE 66 // 一页最大行数

int main(){
    FILE *fp;
    void fileprint(FILE *fp, char *fname);

    if(argc == 1)// 没有输入参数，则打印标准输入
        fileprint(stdin, " ");
    else{
        while(--argc > 0){
            if((fp = fopen(*++argv, "r")) == NULL){
                fprintf(stderr, "print: can't open %s\n", *argv);
                exit(1);
            }else{
                fileprint(*fp, *argv);
            }
        }
    }
    return 0;
}

void fileprint(FILE *fp, char *fname){
    int lineno;// 记录该页已经输出多少行
    int pageno = 1;
    char line[MAXLINE];
    int heading(char *fname, int pageno);

    lineno = heading(fname, pageno++);
    while(fgets(line, MAXLINE, fp) != NULL){
        if(lineno == 1){
            // \f为换页符
            fprintf(stdout, "\f");
            lineno = heading(fname, pageno++);
        }
        fputs(line, stdout);
        // 达到一页的行数，则行数置1，下次循环换页
        if(++lineno > MAXPAGE - MAXBOT)
            lineno = 1;
    }
    fprintf(stdout, "\f");
}

int heading(char *fname, int pageno){
    int ln = 3;
    fprintf(stdout, "\n\n");
    fprintf(stdout, "%s page %d\n", fname, pageno);
    // 打印上空行
    while(ln++ < MAXHDR)
        fprintf(stdout, "\n");
    return ln;
}
```  

## 练习7-9  
### 问题描述  
类似于isupper这样的函数可以通过某种方式实现以达到节省空间或时间的目的。考虑节省空间或时间的实现方式。  
### 问题解答  

较高空间利用率：
``` C
int isupper(char c){
    if(c >= 'A' && c <= 'Z')
        return 1;
    else
        return 0;
}
```  
但是时间效率相对较低，有函数调用的开销。  

较高时间利用率：  
``` C
#define isupper(c) ((c) >= 'A' && (c) <= 'Z') ? 1 : 0
```  
节约时间，因为没有函数调用的开销，但是占用较多空间，因为每次宏调用都要宏展开。  

**此外**，使用宏版本的isupper需要注意由于参数可能会被求值两次带来的潜在问题，比如：  
``` C
char *p = "This is a string";
if(isupper(*p++))
    ...
```  
这里将被展开为：  
```C
((*p++) >= 'A' && (*p++) <= 'Z') ? 1 : 0 
```