# 第一章 导言
## 练习1-1
### 问题描述
在你自己的系统中运行"hello，world"程序，再有意去掉程序中的部分内容，看看会得到什么出错信息。
### 问题解答
```C
#include<stdio.h>
int main(){
    printf("hello, world");
    return 0;
}
```
省略换行符（\n），将使光标停留在输出信息的末尾。
```C
#include<stdio.h>
int main(){
    printf("hello, world\n")
    return 0;
}
```
省略printf()后边的分号，编译器给出出错信息。 
其它自测。
## 练习1-2
### 问题描述
做个试验，当printf函数中的参数字符串中包含\c（c是上边的转义字符序列中未曾列出的某个字符）时，观察会出现什么情况？
### 问题解答
```C
#include<stdio.h>
int main(){
    printf("hello, world\y");
    printf("hello, world\7");
    printf("hello, world\?");
    return 0;
}
```
教材第169页中提到：“如果\后面紧跟的字符不在以上指定字符，则其行为是未定义的。”
## 练习1-3 
### 问题描述 
请修改温度转换程序，使之能在转换表的顶部打印一个标题。 
### 问题解答 
在计算转换表操作代码之前，加一个printf()函数，参数为标题字符串即可。 
## 练习1-4 
### 问题描述 
编写一个程序打印摄氏温度转化为相应的华氏温度的转换表。
### 问题解答 
```C
#include<stdio.h>
int main(){
    float fahr, celsius;
    int lower = 0;//温度下限
    int upper = 300;//温度上限
    int step = 20;//步长（跨度）
    printf("摄氏温度转华氏温度\n");
    ceisius = lower;
    while(celsius <= upper){
        fahr = (9.0 * celsius) / 5.0 + 32.0;
        printf("%3.0f %6.1f\n", celsius, fahr);
        celsius += step;
    }
    return 0;
}
```
## 练习1-5 
### 问题描述 
修改温度转换程序，要求以逆序（即按照从300度递减到0度的顺序）打印温度转换表。
### 问题解答 
```C
#include<stdio.h>
int main(){
    int fahr;
    for(fahr = 300; fahr >= 0; fahr -= 20){
        printf("%3d %6.1f\n", fahr, (5.0 / 9.0) * (fahr - 32));
    return 0;
}
```
fahr初始设为上限，然后按照步长递减。
## 练习1-6
### 问题描述
验证布尔表达式getchar() != EOF的取值是0还是1。
### 问题解答
```C
#include<stdio.h>
int main(){
    int c;
    while(c = (getchar() != EOF))
        printf("%d\n", c);
    printf("%d\n", c);
    return 0;
}
```
while循环中将getchar() != EOF的结果赋给c，未到文件结束时，该表达式为真，c被赋值为1。当程序遇到，文件结束符时，表达式取值为假，此时c被赋值为0，程序结束。
## 练习1-7
### 问题描述
编写一个打印EOF的程序。
### 问题解答
```C
#include<stdio.h>
int main(){
    printf("EOF is %d\n", EOF);
    return 0;
}
```
EOF是在头文件<stdio.h>中定义的，一般赋值为-1，不同系统可能会有不同的值。这正是使用EOF等标准符号常量能够增加程序可移植性的原因。
## 练习1-8
### 问题描述
编写一个统计空格、制表符和换行符个数的程序。
### 问题解答
```C
#include<stdio.h>
int main(){
    int c;
    int nb = 0;//空格
    int nt = 0;//制表符
    int nl = 0;//换行
    while((c = getchar()) != EOF){
        if(c == ' ')
            ++nb;
        if(c == '\t')
            ++nt;
        if(c == '\n')
            ++nl;
    }
    printf("%d %d %d\n", nb, nt, nl);
    return 0;
}
```
## 练习1-9
### 问题描述
编写一个将输入复制到输出的程序，并将其中连续的多个空格用一个空格代替。
### 问题解答
```C
#include<stdio.h>

int main(){
    int c, lastc;
    lastc = 'a';
    while((c = getchar()) != EOF){
        if(c != ' ')
            putchar(c);
        if(c == ' ')
            if(lastc != ' ')
                putchar(c);
        lastc = c;
    }
    return 0;
}
```
整形变量c负责记录当前输入字符的ASCII值，lastc记录前一个输入字符的ASCII值，初始设置为'a'（任意非空格字符即可）。

while循环中第一条if输出非空格字符，第二条if处理空格字符，第三条语句检查当前空格是不是第一个空格，如果lastc为空格，则不输出当前字符（即空格）。最后对lastc刷新。
## 练习1-10
### 问题描述
编写一个将输入复制到输出的程序，将其中的制表符替换为\t，回退符替换为\b，反斜杠替换为\\，这样可以将制表符和回退符以可见的方式显示出来。
### 问题解答
```C
#include<stdio.h>
int main(){
    int c;
    while((c = getchar()) != EOF){
        if(c == '\t')
            printf("\\t");
        else if(c == '\b')
            printf("\\b");
        else if(c == '\\')
            printf("\\\\");
        else
            putchar(c);
    }
    return 0;
}
```
在C语言中，使用'\\\\'来表示反斜杠。
## 练习1-11
### 问题描述
你准备如何测试单词计数程序？如果程序中存在某种错误，那么什么样的输入会导致这样的错误？
### 问题解答
考虑边界情况，如：
* 没有输入
* 没有单词（只有换行符）
* 没有单词（只有换行符、制表符和空格）
* 每个单词各占一行
* 单词出现在文本行行首的情况
* 单词出现在一串空格之后的情况

等等
## 练习1-12
### 问题描述
编写一个程序，以每行一个单词的形式打印其输入
### 问题解答
```C
#include<stdio.h>
#define IN 1 //当前在一个单词内部
#define OUT 0 //在单词外
int main(){
    int c, state;
    state = OUT;
    while((c = getchar()) != EOF){
        if(c == ' ' || c == '\n' || c == '\t'){
            //遇到空格或换行或制表符，且状态在单词内
            if(state == IN){
                putchar('\n');
                state = OUT;
            }
        // 非空格等字符，且状态为OUT，则表示是单词的第一个字符，修改状态且输出
        }else if(state == OUT){
            state = IN;
            putchar(c);
        //单词中其它字符，直接输出
        }else
            putchar(c);
    }
    return 0;
}
```
## 练习1-13
### 问题描述
编写一个程序，打印输入中单词长度的直方图。水平方向和竖直方向。
### 问题解答
```C
// 打印水平直方图
#include<stdio.h>
#define MAXHIST 15
#define MAXWORD 11
#define IN 1
#define OUT 0

int main(){
    int c, i, nc, state;
    int len;
    int maxvalue;
    int ovflow;
    int wl[MAXWORD];// 统计指定长度的单词的个数
    state = OUT;
    nc = 0;// 记录单词长度
    ovflow = 0;
    for(i = 0; i < MAXWORD; ++i)
        wl[i] = 0;
    while((c = getchar()) != EOF){
        if(c == ' ' || c == '\n' || c == '\t'){
            state = OUT;
            if(nc > 0){
                if(nc < MAXWORD)
                    ++wl[nc];
                else
                    ++ovflow;
            }
            nc = 0;
        // 单词开头
        }else if(state == OUT){
            state = IN;
            nc = 1;
        }else//单词内部
            ++nc;
    }
    maxvalue = 0;
    // 找最大值
    for(i = 1; i < MAXWORD; ++i)
        if(wl[i] > maxvalue)
            maxvalue = wl[i];
    for(i = 1; i < MAXWORD; ++i){
        printf("%5d - %5d : ", i, wl[i]);
        if(wl[i] > 0){
            // 如果计算得到的len小于0，则至少打印一个*
            if((len = wl[i] * MAXHIST / maxvalue) <= 0)
                len = 1;
        }else
            len = 0;
        while(len > 0){
            putchar('*');
            --len;
        }
        putchar('\n');
    }
    if(ovflow > 0)
        printf("There are %d words >= %d\n", ovflow, MAXWORD);
    return 0;
}
```
```C
// 打印竖直直方图
#include<stdio.h>
#define MAXHIST 15
#define MAXWORD 11
#define IN 1
#define OUT 0

int main(){
    int c, i, j, nc, state;
    int maxvalue;
    int ovflow;
    int wl[MAXWORD];// 统计指定长度的单词的个数
    state = OUT;
    nc = 0;// 记录单词长度
    ovflow = 0;
    for(i = 0; i < MAXWORD; i++)
        wl[i] = 0;
    while((c = getchar()) != EOF){
        if(c == ' ' || c == '\n' || c == '\t'){
            state = OUT;
            if(nc > 0){
                if(nc < MAXWORD)
                    ++wl[nc];
                else
                    ++ovflow;
            }
            nc = 0;
        // 单词开头
        }else if(state == OUT){
            state = IN;
            nc = 1;
        }else//单词内部
            ++nc;
    }
    maxvalue = 0;
    // 找最大值
    for(i = 1; i < MAXWORD; ++i)
        if(wl[i] > maxvalue)
            maxvalue = wl[i];
    for(i = MAXHIST; i > 0; --i){
        for(j = 1; j < MAXWORD; ++j){
            // 逐行打印，达到长度的，打印 * ，不够的打印空格补齐
            if(wl[j] * MAXHIST / maxvalue >= i)
                printf(" * ");
            else
                printf("   ");
        }
        putchar('\n');
    }
    // 输出下标和取值
    for(i = 1; i < MAXWORD ++i)
        printf("%4d ", i);
    putchar('\n');
    for(i = 1; i < MAXWORD; ++i)
        printf("%4d ", wl[i]);
    putchar('\n');

    if(ovflow > 0)
        printf("There are %d words >= %d\n", ovflow, MAXWORD);
    return 0;
}
```
## 练习1-14
### 问题描述
编写一个程序，打印输入中各个字符出现频率的直方图。
### 问题解答
```C
#include<stdio.h>
#include<ctype.h>

#define MAXHIST 15
#define MAXCHAR 128

int main(){
    int c, i;
    int len;
    int maxvalue;
    int cc[MAXCHAR];
    for(i = 0; i < MAXCHAR; ++i)
        cc[i] = 0;
    // 统计字符个数
    while((c = getchar()) != EOF)
        if(c < MAXCHAR)
            ++cc[c];
    maxvalue = 0;
    for(i = 1; i < MAXCHAR; ++i)
        if(cc[i] > maxvalue)
            maxvalue = cc[i];
    for(i = 1; i < MAXCHAR; ++i){
        // 如果是可打印字符
        if(isprint(i))
            // 输出对应的ASCII码，字符值，个数
            printf("%5d - %c - %5d : ", i, i, cc[i]);
        else
            //不可打印字符，就不输出字符值了
            printf("%5d -  - %5d : ", i, cc[i]);
        if(cc[i] > 0){
            if((len = cc[i] * MAXHIST / maxvalue) <= 0)
                len = 1;
        }else
            len = 0;
        while(len > 0){
            putchar('*');
            --len;
        }
        putchar('\n');
    }
    return 0;
}
```
## 练习1-15
### 问题描述
重新编写1.2节中的温度转换程序，使用函数实现温度转换计算。
### 问题解答
```C
#include<stdio.h>

float celsius(float fahr);

int main(){
    float fahr;
    int lower = 0;
    int upper = 300;
    int step = 20;
    fahr = lower;
    while(faht < upper){
        printf("%3.0f %6.1f\n", fahr, celsius(fahr));
        fahr += step;
    }
    return 0;
}
float celsius(floar fahr){
    return (5.0 / 9.0) * (fahr - 32.0);
}
```
## 练习1-16
### 问题描述
修改打印最长文本行的程序，使其可以打印任意长度的输入行的长度，并尽可能多地打印文本。
### 问题解答
```C
#include <stdio.h>
#define MAXLINE 1000

int getline(char line[], int maxline);
void copy(char to[], char from[]);

int main()
{
    int len;
    int max;
    char line[MAXLINE];
    char longest[MAXLINE];
    max = 0;
    while((len = getline(line, MAXLINE)) > 0)){
        printf("%d, %s", len, line);
        if(len > max){
            max = len;
            copy(longest , line);
        }
    }
    if(max > 0)
        printf("%s", longest);
    return 0;
}
int getline(char s[], int lim){
    int c, i, j;
    j = 0;
    for(i = 0; (c = getchar()) != EOF && c != '\n'; ++i){
        // 要在后边预留两个位置，一个'\n'，一个'\0'
        if(i < lim - 2){
            s[j] = c;
            ++j;
        }
        if(c == '\n'){
            s[j] = c;
            ++j;
            ++i;
        }
    }
    s[j] = '\0';
    // 返回字符串长度
    return i;
}
void copy(char to[], char from[]){
    int i;
    i = 0;
    while((to[i] = from[i]) != '\n')
        ++i;
}
```
## 练习1-17
### 问题描述
编写一个程序，打印长度大于80个字符的所有输入行。
### 问题解答
```C
#include <stdio.h>
#define MAXLINE 1000
#define LONGLINE 80
int getline(char line[], int maxline);

int main()
{
    int len;
    char line[MAXLINE];
    while((len = getline(line, MAXLINE)) > 0)){
        if(len > LONGLINE)
            printf("%s", line);
    }
    return 0;
}
int getline(char s[], int lim){
    int c, i, j;
    j = 0;
    for(i = 0; (c = getchar()) != EOF && c != '\n'; ++i){
        if(i < lim - 2){
            s[j] = c;
            ++j;
        }
        if(c == '\n'){
            s[j] = c;
            ++j;
            ++i;
        }
    }
    s[j] = '\0';
    return i;
}
```
## 练习1-18
### 问题描述
编写一个程序，删除每个输入行末尾的空格及制表符，并删除完全是空格的行。
### 问题解答
```C
#include <stdio.h>
#define MAXLINE 1000

int getline(char line[], int maxline);
int remove(char s[]);

int main()
{
    char line[MAXLINE];
    while(getline(line, MAXLINE) > 0)){
        if(remove(line) > 0)
            printf("%s", line);
    }
    return 0;
}
int remove(char s[]){
    int i = 0;
    while(s[i] != '\n')
        ++i;
    --i;//回到换行符之前
    while(i >= 0 && (s[i] == ' ' || s[i] == '\t'))
        --i;
    // 该行非空,等于0表示只有一个字符
    if(i >= 0){
        s[++i] = '\n';
        s[++i] = '\0';
    }
    return i;
}
int getline(char s[], int lim){
    int c, i, j;
    j = 0;
    for(i = 0; (c = getchar()) != EOF && c != '\n'; ++i){
        if(i < lim - 2){
            s[j] = c;
            ++j;
        }
        if(c == '\n'){
            s[j] = c;
            ++j;
            ++i;
        }
    }
    s[j] = '\0';
    return i;
}
```
## 练习1-19
### 问题描述
编写函数reverse(s)将字符串s中的字符顺序颠倒过来。并使用该函数，编写一个程序，每次颠倒一个输入行的字符顺序。
### 问题解答
```C
#include <stdio.h>
#define MAXLINE 1000

int getline(char line[], int maxline);
void reverse(char s[]);

int main()
{
    char line[MAXLINE];
    while(getline(line, MAXLINE) > 0)){
        reverse(line);
        printf("%s", line);
    }
    return 0;
}
void reverse(char s[]){
    int i, j;
    char temp;
    i = 0;
    while(s[i] != '\0')
        ++i;
    --i;
    if(s[i] == '\n')//跳过行末的换行符
        --i;
    j = 0;
    while(j < i){
        temp = s[j];
        s[j] = s[i];
        s[i] = temp;
        --i;
        ++j;
    }
}
int getline(char s[], int lim){
    int c, i, j;
    j = 0;
    for(i = 0; (c = getchar()) != EOF && c != '\n'; ++i){
        if(i < lim - 2){
            s[j] = c;
            ++j;
        }
        if(c == '\n'){
            s[j] = c;
            ++j;
            ++i;
        }
    }
    s[j] = '\0';
    return i;
}
```
## 练习1-20
### 问题描述
请编写程序detab，将输入中的制表符替换为适当数目的空格，使空格充满到下一个制表符终止的地方。假设制表符终止位的位置是固定的，比如每个n列就会出现一个制表符终止位，n应该作为变量还是符号常量呢？
### 问题解答
```C
#include<stdio.h>
#define TABINC 8// 假设每隔8个位置会有个制表符

int main(){
    int c, nb, pos;
    nb = 0;
    pos = 1;
    while((c = getchar()) != EOF){
        if(c == '\t'){
            // 计算下一个制表符的位置
            nb = TABINC - (pos - 1) % TABINC;
            while(nb > 0){
                putchar(' ');
                ++pos;
                --nb;
            }
        // 遇到换行符则重置pos
        }else if(c == '\n'){
            putchar(c);
            pos = 1;
        }else{
            putchar(c);
            ++pos;
        }
    }
    return 0;
}
```
## 练习1-21
### 问题描述
编写程序entab，将空格串替换为最少数量的制表符和空格，但要保持单词之间的间隔不变。假设制表符终止位的位置与练习1-20一样。当使用一个制表符或者一个空格就能到达下个制表符终止位的时候，选择哪种替换字符比较好？
### 问题解答
```C
#include<stdio.h>
#define TABINC 8
int main(){
    int c, nb, nt, pos;
    nb = 0;
    nt = 0;
    for(pos = 1; (c = getchar()) != EOF; ++pos){
        if(c == ' '){
            // 位于TABINC的倍数时，将空格串替换为一个制表符
            if(pos % TABINC != 0)
                ++nb;
            else{
                nb = 0;
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
            // 遇到换行符，则重置pos
            if(c == '\n')
                pos = 0;
            // 跳过制表符所占位
            else if(c == '\t')
                pos += (TABINC - (pos - 1) % TABINC) - 1;
        }
    }
    return 0;
}
```
如果只需一个空格就能到达下个制表位，选择替换为一个制表符，有助于避免特殊情况。
## 练习1-22
### 问题描述
编写一个程序，将较长的输入行“折”成短一些的两行或多行，折行的位置在输入行的第n列之前的最后一个非空格符之后。要保证程序能智能地处理输入行很长以及在指定列前没有空格或制表符的情况。
### 问题解答
```C
#include<stdio.h>
#define MAXCOL 10
#define TABINC 8
char line[MAXCOL];
int exptab(int pos);
int findblnk(int pos);
int newpos(int pos);
void printl(int pos);

int main(){
    int c, pos;
    pos = 0;
    while((c = getchar()) != EOF){
        line[pos] = c;
        if(c == '\t')
            pos = exptab(pos);
        else if(c == '\n'){
            printl(pos);
            pos = 0;
        // 当pos大于MAXCOL，开始处理截断
        }else if(++pos >= MAXCOL){
            pos = findblnk(pos);
            printl(pos);
            pos = newpos(pos);
        }
    }
    return 0;
}
void printl(int pos){
    int i;
    for(i = 0; i < pos; i++)
        putchar(line[i]);
    if(pos > 0)
        putchar('\n');
}
// 制表符扩展为空格
int exptab(int pos){
    line[pos] = ' ';
    for(++pos; pos < MAXCOL && pos % TABLNC != 0; ++pos){
        line[pos] = ' ';
    }
    if(pos < MAXCOL)
        return pos;
    // 若大于MAXCOL，则截断输出，并将pos重置为0
    else{
        printl(pos);
        return 0;
    }
}
int findblnk(int pos){
    // 倒退查找空格，保证折行处单词的完整性
    while(pos > 0 && line[pos] != ' ')
        --pos;
    if(pos == 0)// 没有空格
        return MAXCOL;
    else
        return pos + 1;// 有空格，则返回空格前的位置
}
// 调整输入行
int newpos(int pos){
    int i, j;
    if(pos <= 0 || pos >= MAXCOL)
        return 0;
    else{
        i = 0;
        for(j = pos; j < MAXCOL; ++j){
            line[i++] = line[j];
        }
        return i;
    }
}
```
## 练习1-23
### 问题描述
编写一个删除C语言程序中的所有注释语句。要正确处理带引号的字符串与字符常量。在C语言中，注释不允许嵌套。
### 问题解答
```C
#include<stdio.h>
void rcomment(int c);
void in_comment(void);
void echo_quote(int c);

int main(){
    int c;
    while((c = getchar()) != EOF){
        rcomment(c);
    }
    return 0;
}
void rcomment(int c){
    int d;
    if(c == '/'){
        if((d = getchar()) == '*')// “/*”型注释
            in_comment();
        else if(d == '/'){// “//型”注释
            putchar(c);
            rcomment(d);
        }else{
            putchar(c);
            putchar(d);
        }
    }else if(c == '\'' || c == '"')
        echo_quote(c);
    else
        putchar(c);
}
// 处理多行注释
void in_comment(void){
    int c, d;
    c = getchar();
    d = getchar();
    while(c != '*' || d != '/'){
        c = d;
        d = getchar();
    }
}
// 处理引号内的注释标志
void echo_quote(int c){
    int d;
    putchar(c);
    // 找到下个和c对应的引号之前循环
    while((d = getchar()) != c){
        putchar(d);
        // 如果出现引号跟在反斜杠后边的情况，则不作为结束标志
        // 因为这种情况的引号是转义的，原样输出即可
        if(d == '\\')
            putchar(getchar());
    }
    putchar(d);
}
```
## 练习1-24
### 问题描述
编写一个程序，查找C语言程序中的基本语法错误，如圆括号、方括号以及花括号不匹配等。要正确处理引号（包括单引号、双引号）、转义字符序列与注释。
### 问题解答
```C
#include<stdio.h>
int brace, brack, paren;
void in_quote(int c);
void in_comment(void);
void search(int c);

int main(){
    int c;
    extern int brace, brack, paren;
    while((c = getchar()) != EOF){
        if(c == '/'){
            if((c = getchar()) == '*')
                in_comment();
            else
                search(c);
        }else if(c == '\'' || c == '"')
            in_quote(c);
        else
            search(c);
        if(brace < 0){
            printf("Unbalanced braces\n");
            brace = 0;
        }else if(brack < 0){
            printf("Unbalanced brackets\n");
            brack = 0;
        }else if(paren < 0){
            printf("Unbalanced parentheses\n");
            paren = 0;
        }
    }
    if(brace > 0)
        printf("Unbalanced braces\n");
    if(brack > 0){
        printf("Unbalanced brackets\n");
    if(paren > 0){
        printf("Unbalanced parentheses\n");
    return 0;
}

void search(int c){
    extern int brace, brack, paren;
    switch(c){
    case '{':
        ++brace;
        break;
    case '}':
        --brace;
        break;
    case '[':
        ++brack;
        break;
    case ']':
        --brack;
        break;
    case '(':
        ++paren;
        break;
    case ')':
        --paren;
        break;
    default:
        break;
    }
}

void in_comment(void){
    int c, d;
    c = getchar();
    d = getchar();
    while(c != '*' || d != '/'){
        c = d;
        d = getchar();
    }
}

void in_quote(int c){
    int d;
    while((d = getchar()) != c){
        if(d == '\\')
            getchar();
    }
}
```
简单的处理括号匹配和处理引号。
