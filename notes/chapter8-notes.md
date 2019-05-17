# UNIX系统接口  

- 在UNIX系统中，所有的外设都被看作是文件系统中的文件，因此，所有的输入/输出都要通过读/写文件来完成。通过一个单一的接口就可以处理外围设备和程序之间的通信。  

- **文件描述符**是一个非负整数，操作系统用它来标识文件。UNIX下运行程序时，默认打开三个文件：标准输入（0）、标准输出（1）和标准错误（2）。  

- 在UNIX文件系统中，每个文件对应一个9比特的权限信息，分别控制文件的所有者、所有者组合其他成员对文件的读、写和执行访问权限。  

- vprintf函数用一个参数取代了变长参数表，且此参数通过调用va_start宏进行初始化。  
  ``` C
  #include<stdio.h>
  #include<stdarg.h>
  void error(char *fmt, ...){
      va_list args;
      va_start(args, fmt);
      fprintf(stderr, "error: ");
      vfprintf(stderr, fmt, args);
      fprintf(stderr, "\n");
      va_end(arg);
      exit(1);
  }
  ```  
- 使用lseek系统调用可以随机访问文件，格式：  

  `long lseek(int fd, long offset, int origin);`

  将文件描述符为 `fd` 的文件的当前位置设置为 `offset`，其中 `offset` 是相对于 `origin` 指定的位置而言的。随后的读写操作将从此位置开始。 `origin` 的值可以是0、1和2，分别指定 `offset` 从文件开始，当前位置或文件结束处开始算起。该系统调用返回 `long` 类型的值，表示文件的新位置，若发生错误则返回-1。标准库函数 `fseek` 与其类似，不过它的第一个参数是 `FILE*` 类型，且发生错误时返回非0值。  

- 在UNIX系统中，目录就是文件，它包含了一个文件名列表和一些指示文件位置的信息。“位置”是一个指向其他表（即 i 结点表）的索引。文件的 i 结点 是存放除文件名意外的所有文件信息的地方。目录项一般包含两个条目：文件名和 i 结点编号。  

- malloc管理的空间不是连续的，每个块包含一个长度、一个指向下一块的指针以及一个指向自身存储空间的指针。这些块按照存储空间的升序组织，最后一块指向第一块。

- 空闲块开始处的控制信息称为“头部”，为了简化块的对齐，所以块的大小必须是头部大小的整数倍，且头部已正确地对齐。

  ``` C
  typedef long Align;// 按照long类型边界对齐
  union header{
      struct{
          union header *ptr;
          unsigned size;
      }s;
      Align x;// 不被使用，仅用于强制对齐
  };
  typedef union header Header;
  ```  

- 在malloc函数中，请求的长度将被舍入，以保证它是头部大小的整数倍。实际分配的块将多包含一个头部单元。malloc返回的指针指向空闲空间而不是头部。
