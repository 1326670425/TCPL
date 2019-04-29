# 第三章 控制流  

## 练习3-1  
### 问题描述  
在上面有关折半查找的例子中，while循环语句内共执行了两次测试，其实只要一次就足够（代价是将更多的测试在循环外进行）。重写该函数，使得在循环内只执行一次测试。比较两个版本函数的运行时间。  
### 问题解答  
```C
int binsearch(int x, int v[], int n){
    int low, high, mid;
    low = 0;
    high = n - 1;
    mid = (low + high) / 2;
    while(low <= high && x != v[mid]){
        if(x < mid)
            high = mid - 1;
        else
            low = mid + 1;
        mid = (low + high) / 2;
    }
    if(x == v[mid])
        return mid;
    else
        return -1;
}
```  
将测试 `x != v[mid]` 移动到了while条件中，运行时间几乎没有差异，并没有提升性能。  
## 练习3-2  
### 问题描述  
编写一个函数escape(s, t)，将字符串t复制到字符串s中，并在复制过程中将换行符、制表符等不可见字符分别转换为 `\n` , `\t` 等相应可见的转义字符序列。要求使用switch语句。再编写一个具有相关功能的函数，在复制过程中将转义字符转换为实际字符。  
### 问题解答  
```C
void excape(char s[], char t[]){
    int i, j;
    for(i = j = 0; t[i] != '\0'; ++i){
        switch(t[i]){
        case '\n':
            s[j++] = '\\';
            s[j++] = 'n';
            break;
        case '\t':
            s[j++] = '\\';
            s[j++] = 't';
            break;
        default:
            s[j++] = t[i];
            break;
        }
    }
    s[j] = '\0';
}  
```  
```C
void unexcape(char s[], char t[]){
    int i, j;
    for(i = j = 0; t[i] != '\0'; ++i){
        switch(t[i]){
        case '\\':
            switch(t[++i]){
            case 'n':
                s[j++] = '\n';
                break;
            case 't':
                s[j++] = '\t';
                break;
            default:// 其它情况
                s[j++] = '\\';
                s[j++] = t[i];
                break;
            }
            break;
        default:
            s[j++] = t[i];
            break;
        }
    }
    s[j] = '\0';
}
```  
## 练习3-3  
### 问题描述  
编写函数expand(s1, s2)，将字符串s1中类似a-z一类的速记符号在字符串s2中扩展为等价的完整列表abc···xyz。该函数能够处理大小写字母和数字，并可以处理a-b-c、a-z0-9与-a-z等类似情况。作为前导和尾随的-字符原样打印。  
### 问题解答  
```C
void expand(char s1[], char s2[]){
    char c;
    int i, j;
    i = j = 0;
    while((c = s1[i++]) != '\0'){
        // 遇到要扩展的片段
        // 只有当'-'左边的字符小于右边的字符时，才扩展
        if(s1[i] == '-' && s1[i + 1] >= c){
            i++;
            while(c < s1[i])
                s2[j++] = c++;
        // 否则直接复制
        }else
            s2[j++] = c;
    }
    s2[j] = '\0';
}
```  
## 练习3-4  
### 问题描述  
在数的对二的补码表示中，我们编写的itoa函数不能处理最大的负数，及n等于-(2<sup>字长-1</sup>)的情况。请解释其原因。修改该函数，使它在任何机器上运行时都能打印出正确的值。  
### 问题解答  
```C
#define abs(x) ((x) < 0 ? -(x) : (x))
void itoa(int n, char s[]){
    int i, sign;
    sign = n;
    i = 0;
    do{
        s[i++] = abs(n % 10) + '0';
    }while((n /= 10) != 0);
    if(sign < 0)
        s[i++] = '-';
    s[i] = '\0';
    reverse(s);
}
```  
由于最大的负数无法通过 `n = -n` 来转为正值，所以只用变量 `sign` 来保存 `n` 的初值，用宏 `abs` 来计算 `n % 10` 的绝对值，将结果转为正数，即可绕过最大负数无法转为正数的问题。  

此外，将do-while语句条件表达式改为 `(n /= 10) != 0` ，避免因为n是一个负数而使函数陷入无限循环。  

## 练习3-5  
### 问题描述  
编写函数itob(n, s, b)，将整数n转换为以b为底的数，并将转换结果以字符的形式保存到字符串s中。例如，itob(n, s, 16)将整数n格式化为十六进制数保存在s中。  
### 问题解答  
```C
void itob(int n, char s[], int b){
    int i, j, sign;
    if((sign = n) < 0)
        n = -n;
    i = 0;
    do{
        j = n % b;
        // 如果j大于9，则转换为字母表示
        s[i++] = (j <= 9) ? j + '0' : j + 'a' - 10;
    }while((n /= b) > 0);
    if(sign < 0)
        s[i++] = '-';
    s[i] = '\0';
    reverse(s);
}
```  
## 练习3-6  
### 问题描述  
修改itoa函数，使得该函数可以接收三个参数。其中，第三个参数为最小字段宽度。为了保证转换后所得的结果至少具有第三个参数指定的最小宽度，在必要时应在所得结果的左边填充一定空格。  
### 问题解答  
```C
#define abs(x) ((x) < 0 ? -(x) : (x))
void itoa(int n, char s[], int w){
    int i, sign;
    sign = n;
    i = 0;
    do{
        s[i++] = abs(n % 10) + '0';
    }while((n /= 10) != 0);
    if(sign < 0)
        s[i++] = '-';
    // 补齐宽度
    while(i < w)
        s[i++] = ' ';
    s[i] = '\0';
    reverse(s);
}
```  
