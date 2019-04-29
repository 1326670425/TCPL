# 第三章 控制流  

* switch语句中，case只是个标号，因此某个分支中的代码执行完后，程序将进入下一个分支继续执行，除非显式跳转，一般使用break语句或return语句跳出。除了一个计算需要多个标号的情况外，应尽量减少从一个分支直接进入下一个分支执行这种用法。  

* 逗号运算符 `,` 也是C语言优先级最低的运算符，在for循环中经常用到。被逗号分隔的一对表达式将按照从左到右的顺序进行求值，各表达式右边的操作数的类型和值即为其结果的类型的和值。如reverse(s):  
  ```C
  #include<string.h>
  void reverse(char s[]){
      int c, i, j;
      for(i = 0, j = strlen(s) - 1; i < j; ++i, --j){
          c = s[i];
          s[i] = s[j];
          s[j] = c;
      }   
  }
  ```  
  逗号表达式还适用于reverse函数中元素的交换，这样，元素的交换过程便可以看作是一个单步操作。  
  ```C
  for(i = 0, j = strlen(s) - 1; i < j; ++i, --j){
      c = s[i], s[i] = s[j], s[j] = c;
  } 
  ```  

* 注意do-while语句与while语句的区别：do-while语句先执行一次循环体在做判断，while先判断，条件满足才执行循环体。  
  ```C
  // itoa函数：将数字n转换为字符串并保存到s中
  void itoa(int n, char s[]){
      int i, sign;
      if((sign = n) < 0)
          n = -n;
      i = 0;
      do{
          s[i++] = n % 10 + '0';
      }while((n /= 10) > 0);
      if(sign < 0)
          s[i++] = '-';
      s[i] = '\0';
      reverse(s);
  }
  ```  
  这里使用do-while语句，因为即使n的值为0，也需要在s里放一个字符'0'。

* `goto` 语句常用于终止在某些深度嵌套的结构中的处理过程，例如一次跳出多层循环。不过还是要尽可能少用goto语句。  
