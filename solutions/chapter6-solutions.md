# 第六章 结构  

## 练习6-1  
### 问题描述  
上述getword函数还不能正确处理下划线、字符串常数、注释以及预编译器控制指令。请编写一个更完善的getword函数。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

int getword(char *word, int lim){
    int c, d, comment(void), getch(void);
    void ungetch(int c);
    char *w = word;

    while(isspace(c = getch()))
        ;
    if(c != EOF)
        *w++ = c;
    // 为处理 _ 和 # ，现将它们和其后的字母数字看做单词的一部分
    if(isalpha(c) || c == '_' || c == '#'){
        for(; --lim > 0; ++w){
            if(!isalnum(*w = getch()) && *w != '_'){
                ungetch(*w);
                break;
            }
        }
    // 遇到引号
    }else if(c == '\'' || c == '"'){
        for(; --lim > 0; ++w){
            // 字符串中 \' 或 \“ 表示的是引号本身，并不表示字符串的结束
            // 遇到反斜杠，其后可能是引号，为防止判断错误提前终止字符串的读取，
            // 此处预先把下个字符也读取了。
            if((*w = getch()) == '\\')
                *++w = getch();
            else if(*w == c){
                w++;
                break;
            }else if(*w = EOF)
                break;
        }
    }else if(c == '/'){
        // 注释
        if((d = getch()) == '*')
            c = comment();
        else
            ungetch(d);
    }
    *w = '\0';
    return c;
}

int comment(void){
    int c;
    while((c = getch()) != EOF){
        if(c == '*'){
            if((c = getch()) == '/')//注释结束
                break;
            else
                ungetch(c);
          }
    }
    return c;
}
```  

## 练习6-2  
### 问题描述  
编写一个程序，用以读入一个C语言程序，并按字母表顺序分组打印变量名，要求每一组内各变量名的前6个字符相同，其余字符不同。字符串和注释中的单词不予考虑。请将6作为一个可在命令行中设定的参数。  
### 问题解答  
``` C  
#include<stdio.h>
#include<ctype.h>
#include<string.h>
#include<stdlib.h>

struct tnode{
    char *word;
    int match;
    struct tnode *left;
    struct tnode *right;
};

#define MAXWORD 100
#define YES 1
#define NO 0

struct tnode *addtree(struct tnode *p, char *w, int num, int *found);
void treeprint(struct tnode *root);
int getword(char *w, int lim);// 同练习6-1

int main(int argc, char *argv[]){
    struct tnode *root;
    char word[MAXWORD];
    int found = NO;
    int num;
    // 参数设置
    num = (--argc && (*++argv)[0] == '-' ? atoi(argv[0] + 1) : 6);
    root = NULL;
    while(getword(word, MAXWORD) != EOF){
        if(isalpha(word[0]) && strlen(word) >= num)
            root = addtree(root, word, num, &found);
        found = NO;
    }
    treeprint(root);
    return 0;
}

struct tnode *talloc(void);
int compare(char *w, struct tnode *p, int num, int *found);
// 向树中添加单词
struct tnode *addtree(struct tnode *p, char *w, int num, int *found){
    int cond;
    if(p == NULL){
        p = talloc();
        p->word = strdup(w);
        p->match =  *found;
        p->left =  p->right = NULL;
    }else if((cond = compare(w, p, num, found)) < 0)
        p->left = addtree(p->left, w, num, found);
    else if(cond > 0)
        p->right = addtree(p->right, w, num, found);
    return p;
}

int compare(char *s, struct tnode *p, int num, int *found){
    int i;
    char *t = p->word;
    for(i = 0; *s == *t; ++i, ++s, ++t)
        if(*s == '\0')// 相等
            return 0;
    if(i >= num){// 匹配长度大于等于给定的num
        *found = YES;
        p->match = YES;
    }
    return *s - *t;
}

void treeprint(struct tnode *p){
    if(p != NULL){
        treeprint(p->left);
        if(p->match)
            printf("%s\n", p->word);
        treeprint(p->right);
    }
}
```  
这个程序把前num个字符相同的变量名打印出来。变量 `found` 是一个布尔量，如果新识别出来的单词与变量树上的某个单词前num个字符相同，我们就把 `found` 设置为 `YES`，否则设为 `NO`。  

函数compare将把被添加到变量树中的单词和已经在变量树中的单词比较，如果在前num个字符中找到一个匹配，那么，与变量树中的单词相对于的 `*found` 以及匹配成员 `p->match` 将被设置为 `YES`。  

treeprint函数打印变量树，前num个字符相同的那些单词将被集中打印在一起。  

## 练习6-3  
### 问题描述  
编写一个交叉引用程序，打印文档中所有单词的列表，并且每个单词还有一个列表，记录出现过该单词的行号。对the、and等非实义单词不予考虑。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<ctype.h>
#include<stdlib.h>

#define MAXWORD 100
// 记录行号
struct linklist{
    int lnum;
    struct linklist *ptr;
};

struct tnode{
    char *word;
    struct linklist *lines;// 保存该单词出现的行号链表
    struct tnode *left;
    struct tnode *right;
};

struct tnode *addtree(struct tnode *p, char *w, int linenum);
int getword(char *w, int lim);
int noiseword(char *w);
void treeprint(struct tnode *p);

int main(){
    struct tnode *root;
    char word[MAXWORD];
    int linenum = 1;

    root = NULL;
    while(getword(word, MAXWORD) != EOF){
        if(isalpha(word[0]) && noiseword(word) == -1)
            root = addtree(root, word, linenum);
        else if(word[0] == '\n')
            linenum++;
    }
    treeprint(root);
    return 0;
}

struct tnode *talloc(void);
struct linklist *lalloc(void);
void addln(struct tnode *p, int linenum);

struct tnode *addtree(struct tnode *p, char *w, int linenum){
    int cond;
    if(p == NULL){
        p = talloc();
        p->word = strdup(w);
        p->lines = lalloc();
        p->lines->lnum = linenum;
        p->left =  p->right = NULL;
    }else if((cond = strcmp(w, p->word)) == 0)//单词匹配，则添加行号
        addln(p, linenum);
    else if(cond > 0)
        p->right = addtree(p->right, w, linenum);
    else
        p->left = addtree(p->left, w, linenum);
    return p;
}

// 给单词添加行号
void addln(struct tnode *p, int linenum){
    struct linklist *temp;
    temp = p->lines;
    // 遍历找到插入点
    while(temp->ptr != NULL && temp->lnum != linenum)
        temp = temp->ptr;
    // 如果没有当前行号的结点，则添加
    if(temp->lnum != linenum){
        temp->ptr = lalloc();
        temp->ptr->lnum = linenum;
        temp->ptr->ptr = NULL;
    }
}

void treeprint(struct tnode *p){
    struct linklist *temp;
    if(p != NULL){
        treeprint(p->left);
        printf("%10s: ", p->word);
        // 打印行号
        for(temp = p->lines; temp != NULL; temp = temp->ptr)
            printf("%4d ", temp->lnum);
        printf("\n");
        treeprint(p->right);
    }
}
// 申请linklist节点
struct linklist *lalloc(void){
    return (struct linklist *)malloc(sizeof(struct linklist));
}
// 设置停用词
int noiseword(char *w){
    // 为了二分搜索，停用词数组必须按字典序升序排列
    static char *nw[] = {
        "a",
        "an",
        "and",
        "are",
        "in",
        // ...
    };
    int cond, mid;
    int low = 0;
    int high = sizeof(nw) / sizeof(char *) - 1;

    while(low <= high){
        mid = (low + high) / 2;
        if((cond = strcmp(w, nw[mid])) < 0)
            high = mid - 1;
        else if(cond > 0)
            low = mid + 1;
        else
            return mid;
    }
    // 不是停用词
    return -1;
}
```  
此外，为了记录行号，还需要对getword函数做修改，使得它能返回换行符：  
``` C
while(isspace(c = getch()) && c != '\n')
    ;
```  

## 练习6-4  
### 问题描述  
编写一个程序，根据单词的出现频率按降序打印输入的各个不同单词，并在每个单词的前面标上它的出现次数。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>

#define MAXWORD 100
#define NDISTINCT 1000// 限制不同单词的最大数目

struct tnode{
    char *word;
    int count;
    struct tnode *left;
    struct tnode *right;
};

struct tnode *addtree(struct tnode *p, char *w);
int getword(char *w, int lim);
void treestore(struct tnode *p);

struct tnode *list[NDISTINCT];
int ntn = 0;// 结点个数

int main(){
    struct tnode *root;
    char word[MAXWORD];
    int i;

    root = NULL;
    while(getword(word, MAXWORD) != EOF){
        if(isalpha(word[0]))
            root = addtree(root, word);
    }
    treestore(root);
    sortlist();
    for(i = 0; i < ntn; ++i)
        printf("%2d:%20s\n", list[i]->count, list[i]->word);
    return 0;
}
// 将树中结点保存在list数组中，为了随后排序
void treestore(struct tnode *p){
    if(p != NULL){
        treestore(p->left);
        if(ntn < NDISTINCT)
            list[ntn++] = p;
        treestore(p->right);
    }
}
// 按照单词出现次数排序，采用希尔排序算法
void sortlist(){
    int gap, i, j;
    struct tnode *temp;
    // gap控制步长
    for(gap = ntn / 2; gap > 0; gap /= 2){
        for(i = gap; i < ntn; ++i){
            for(j = i - gap; j >= 0; j -= gap){
                if((list[j]->count) >= (list[j + gap]->count))
                    break;
                temp = list[j];
                list[j] = list[j + gap];
                list[j + gap] = temp;
            }
        }
    }
}

struct tnode *addtree(struct tnode *p, char *w){
    int cond;
    if(p == NULL){
        p = talloc();
        p->word = strdup(w);
        p->count = 1;
        p->left =  p->right = NULL;
    }else if((cond = strcmp(w, p->word)) == 0)//单词匹配，则出现次数递增
        p->count += 1;
    else if(cond > 0)
        p->right = addtree(p->right, w);
    else
        p->left = addtree(p->left, w);
    return p;
}
```  

## 练习6-5  
### 问题描述  
编写一个函数，它将从由lookup和install维护的表中删除一个变量名及其定义。  
### 问题解答  
``` C
unsigned hash(char *s);

void undef(char *s){
    int h;
    struct nlist *prev, *np;

    prev = NULL;
    h = hash(s);
    for(np = hashtab[h]; np != NULL; np = np->next){
        if(strcmp(s, np->name) == 0)// 找到则退出循环
            break;
        prev = p;
    }
    if(np != NULL){
        if(prev == NULL)// 第一个结点
            hashtab[h] = np->next;
        else
            prev->next = np->next;
        free((void *)np->name);
        free((void *)np->defn);
        free((void *)np);
    }
}
```  

## 练习6-6  
### 问题描述  
以本节介绍的函数为基础，编写一个适合C语言程序使用的 `#define` 处理器的简单版本（即无参数的情况）。你会发现getch和ungetch函数非常有用。  
### 问题解答  
``` C
#include<stdio.h>
#include<ctype.h>
#include<string.h>

#define MAXWORD 100

struct nlist{
    struct nlist *next;
    char *name;
    char *defn;
};

void error(int x, char *s);
int getch(void);
void getdef(void);
int getword(char *w, int lim);
struct nlist *install(char *name, char *defn);
struct nlist *lookup(char *name);
void skipblanks(void);
void undef(char *name);
void ungetch(int c);
void ungets(char *s);

int main(){
    char w[MAXWORD];
    struct nlist *p;

    while(getword(w, MAXWORD) != EOF){
        if(strcmp(w, "#") == 0)
            getdef();
        else if(!isalpha(w[0]))
            printf("%s", w);// 不是字母
        else if((p = lookup(w)) == NULL)
            printf("%s", w);
        else
            ungets(p->defn);
    }
    return 0;
}

void getdef(void){
    int c, i;
    char def[MAXWORD], dir[MAXWORD], name[MAXWORD];

    skipblanks();
    if(!isalpha(getword(dir, MAXWORD)))
        error(dir[0], "getdef: expecting a directive after #");
    else if(strcmp(dir, "define") == 0){
        skipblanks();
        if(!isalpha(getword(name, MAXWORD)))
            error(name[0], "getdef: non-alpha - name expected");
        else{
            skipblanks();
            for(i = 0; i < MAXWORD - 1; --i)
                if((def[i] == getch()) == EOF || def[i] == '\m')// 结束定义
                    break;
            def[i] = '\0';
            if(i < 0)
                error('\n', "getdef: incomplete define");
            else
                install(name, def);
        }
    }else if(strcmp(dir, "undef") == 0){
        skipblanks();
        if(!isalpha(getword(name, MAXWORD)))
            error(name[0], "getdef: non-alpha in undef");
        else
            undef(name);
    }else
        error(dir[0], "getdef: expecting a directive after #");
}
// 报错，且跳过该行后边的字符
void error(int c, char *s){
    printf("errorL %s\n", s);
    while(c != EOF && c != '\n')
        c = getch();
}

void skipblanks(void){
    int c;
    while((c = getch()) == ' ' || c == '\t')
        ;
    ungetch(c);
}
```  
