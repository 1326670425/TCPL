# 第五章 指针与数组  

- 地址运算符 `&` 只能应用于内存中的对象，即变量和数组元素。它不能作用于表达式、常量或 `register` 类型的变量。  

- 数组名和指针的不同之处：指针是一个变量，因此，在C语言中，假设 `pa` 是 `int` 型的指针， `a` 是 `int` 型的数组名。语句 `pa = a` 和 `pa++` 都是合法的。但是数组名不是变量，所以对数组名的操作是非法的，比如 `a = pa` 或 `a++` 。  

- 指向不同数组的元素的指针之间的算术或者比较运算没有定义。（特例：指针的算术运算可以使用数组最后一个元素的下一个元素的地址。）  

- 区别如下两种定义：  
  
  &emsp;&emsp; `char amessage[] = "now is the time";`  
  &emsp;&emsp; `char *pamessage = "now is the time";`  

  `amessage`是一个仅仅能够存放初始化字符串以及空字符 `\0` 的一维数组。数组中的单个字符可以进行修改，但 `amessage` 始终指向同一个存储位置。另一方面， `pmessage` 是一个指针，其初值指向一个字符串常量，之后它可以被修改以指向其它地址，但试图修改字符串的内容，结果是没有定义的。  

- 如果将二维数组作为参数传递给函数，那么在函数的参数声明中必须指明数组的列数，而行数可以不指明。一般地，数组第一维可以不指明，其它各维都要明确指定大小。  

- **二维数组和指针数组的区别：**  
  
  对于如下两个定义：  
  ``` C
  int a[10][20];
  int *b[10];
  ```  
  `a` 是一个真正的二维数组，它分配了200个 `int` 类型长度的存储空间，并且通过常规的矩阵下标计算公式来计算得到的元素位置。但是`b` 仅仅是分配了10个指针，并且没有对它们初始化，它们的初始化必须以显式的方式进行，比如静态初始化或通过代码初始化。假如`b`的每个元素都指向一个具有20个`int`类型元素的数组，那么编译器就要为它分配200个 `int` 类型长度的存储空间以及10个指针的存储空间。  

- 指针数组的一个优点是：数组的每一行长度可以不同。  

- `argc` 表示命令行参数个数，`argv`表示指向字符串数组的指针，每个字符串对应一个参数。`argv[0]` 是程序名，ANSI规定要求 `argv[argc]` 的值必须是一个空指针。  

- 指向函数的指针：

  ``` C
    #include<stdio.h>
    #include<string.h>
    
    #define MAXLINES 5000
    char *lineptr[MAXLINES];
    
    int numcmp(char s[], char t[]);
    int readlines(char *lineptr[], int maxlines);
    void qsort(char *v[], int left, int right,
               int (*comp)(void *, void *));
    void writelines(char *lineptr[], int nlines, int decr);
    
    int main(int argc, char *argv[]){
    
        int nlines;
        int num;
    
        if--argc > 0 && strcmp(argv[1], "-n") == 0)
            num = 1;
        if((nlines = readline(lineptr, LINES)) >= 0){
    
            qsort((void *)lineptr, 0, nlines - 1, (int (*)(void *, void *))(num ? numcmp : strcmp));
            writelines(lineptr, nlines, option & DECR);
            return 0;
        }else{
            printf("input too big to sort\n");
            return 0;
        }
    }
    void qsort(void *v[], int left, int right, int (*comp)(void *, void *)){
        int i, last;
        void swap(void *v[], int left, int right);
        if(left >= right)
            return ;
        swap(v, left, (left + right) / 2);
        last = left;
        for(i = left + 1; i <= right ++i)
            if((*comp)(v[i], v[left]) < 0)
            swap(v, ++last, i);
        swap(v, left, right);
        qsort(v, left, last - 1, comp);
        qsort(v, last + 1, right, comp);
    }
  ```  
