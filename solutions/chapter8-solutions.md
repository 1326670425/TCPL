# UNIX系统接口  

## 练习8-1  
### 问题描述  
用read、write、open和close系统调用代替标准库中功能等价的函数，重写第7章的cat程序，并通过实验比较两个版本的相对执行速度。  
### 问题解答  
``` C
#include<stdio.h>
#include<fcntl.h>
#include "syscalls.h"

void error(char *fmt, ...);

int main(){
    int fd;
    void filecopy(int ifd, int ofd);

    if(argc == 1)// 没有参数，将标准输入复制到标准输出
        filecopy(0, 1);
    else{
        while(--argc > 0){
            // 尝试以只读方式打开文件并返回文件描述符
            if((fd = open(*++argv, O_RDONLY)) == -1)
                error("cat: can't open %s\n", *argv);
            else{
                filecopy(fd, 1);
                close(fd);
            }
        }
    }
    return 0;
}

void filecopy(int ifd, int ofd){
    int n;
    char buf[BUFSIZ];
    // 等于0表示读到文件尾，等于-1表示出错
    while((n = read(ifd, buf, BUFSIZ)) > 0)
        // 实际写入不等于n则出错
        if(write(ofd, buf, n) != n)
            error("cat: write error\n");
}
```  

这个版本由于使用系统调用，要比之前的版本快些。  

## 练习8-2  
### 问题描述  
用字段代替显式的按位操作，重写fopen和_filebuf。比较相应代码的长度和执行速度。  
### 问题解答  
``` C
#include<fcntl.h>
#include "syscalls.h."

#define PERMS 0666 // 访问权限
#define OPEN_MAX 20

typedef struct _iobuf{
    int cnt;    //剩余字符数
    char *ptr;  // 下一个字符的位置
    char *base; // 缓冲区的位置
    int flag;   // 文件访问模式
    int fd;     // 文件描述符
}FILE;
extern FILE _iob[OPEN_MAX];
// 位字段方式重新定义flag
struct flag_field{
    unsigned is_read    : 1;
    unsigned is_write   : 1;
    unsigned is_unbuf   : 1;
    unsigned is_buf     : 1;
    unsigned is_eof     : 1;
    unsigned is_er      : 1;
};

FILE *fopen(char *name, char *mode){
    int fd;
    FILE *fp;
    // 访问模式错误
    if(*mode != 'r' && *mode != 'w' && *mode != 'a')
        return NULL;
    // 找空位置
    for(fp = _iob; fp < _iob + OPEN_MAX; fp++){
        if(fp->flag.is_read == 0 && fp->flag.is_write == 0)
            break;
    }
    // 没找到空位置
    if(fp >= _iob + OPEN_MAX)
        return NULL;
    if(*mode == 'w')
        fd = creat(name, PERMS);
    else if(*mode == 'a'){
        // 文件不存在，则创建
        if((fd = open(name, O_WRONLY, 0)) == -1)
            fd = creat(name, PERMS);
        lseek(fd, 0L, 2);// 定位到文件末尾
    }else
        fd = open(name, O_RDONLY, 0);
    if(fd == -1)// 没有得到文件描述符，权限不够或访问出错
        return NULL;
    fp->fd = fd;
    fp->cnt = 0;
    fp->base = NULL;
    fp->flag.is_unbuf = 0;
    fp->flag.is_buf = 1;
    fp->flag.is_eof = 0;
    fp->flag.is_err = 0;
    if(*mode == 'r'){
        fp->flag.is_read = 1;
        fp->flag.is_write = 0;
    }else{
        fp->flag.is_read = 0;
        fp->flag.is_write = 1;
    }
    return fp;
}

int _filebuf(FILE *fp){
    int bufsize;
    // 验证权限
    if(fp->flag.is_read == 0 ||
       fp->flag.is_eof  == 1 ||
       fp->flag.is_err  == 1)
        return EOF;
    bufsize = (fp->flag.is_unbuf == 1) ? 1 : BUFSIZ;
    if(fp->base == NULL)
        if((fp->base = (char *)malloc(bufsize)) == NULL)
            return EOF;
    fp->ptr = fp->base;
    fp->cnt = read(fp->fd, fp->ptr, bufsize);
    if(--fp->cnt < 0){
        if(fp->cnt == -1)
            fp->flag.is_eof = 1;
        else
            fp->flag.is_err = 1;
        fp->cnt = 0;
        return EOF;
    }
    return (unsigned char) *fp->ptr++;
}
```  
新方案具有更高的可读性，但是增加了代码规模，执行速度也变慢了。位字段不仅要依赖于计算机硬件，还降低执行速度。  

## 练习8-3  
### 问题描述  
请设计并编写函数_flushbuf、fflush和fclose。  
### 问题解答  
``` C
#include "syscalls.h"

int _flushbuf(int x, FILE *fp){
    unsigned nc;
    int bufsize;
    // 验证指针合法性
    if(fp < _iob || fp >= _iob + OPEN_MAX)
        return EOF;
    // 不是以写操作打开，或发生错误，返回EOF
    if((fp->flag & (_WRITE | _ERR)) != _WRITE)
        return EOF;
    bufsize = (fp->flag & _UNBUF) ? 1 : BUFSIZ;
    // 缓冲区不存在
    if(fp->base == NULL){
        if((fp->base = (char *)malloc(bufsize)) == NULL){
            fp->flag |= _ERR;
            return EOF;
        }
    }else {
        nc = fp->ptr - fp->base;
        // 把缓冲区内容写入文件
        if(write(fp->fd, fp->base, nc) != nc){
            fp->flag |= _ERR;
            return EOF;
        }
    }
    fp->ptr = fp->base;// 初始化缓冲区
    *fp->ptr++ = (char)x;// 保存当前字符
    // 因为放了一个 x 字符
    fp->cnt = bufsize - 1;
    return x;
}

int fclose(FILE *fp){
    int rc;
    if((rc = fflush(fp)) != EOF){
        free(fp->base);
        fp->ptr = NULL;
        fp->cnt = 0;
        fp->base = NULL;
        fp->flag &= ~(_READ | _WRITE);
    }
    return rc;
}


int fflush(FILE *fp){
    int rc = 0;
    if(fp < _iob || fp >= _iob + OPEN_MAX)
        return EOF;
    if(fp->flag & _WRITE)
        rc = _flushbuf(0, fp);
    fp->ptr = fp->base;
    fp->cnt = (fp->flag & _UNBUF) ? 1 : BUFSIZ;
    return rc;
}
```  

## 练习8-4  
### 问题描述  
标准库函数  

&emsp;&emsp;`int fseek(FILE *fp, long offset, int origin);`

类似于函数lseek，所不同的是，该函数中的 `fp` 是一个文件指针而不是文件描述符，且返回值是一个 `int` 类型的状态而非位置值。编写函数fseek，并确保该函数与库中其它函数使用的缓冲能够协同工作。  
### 问题解答  
``` C
#include "syscalls.h"

int fseek(FILE *fp, long offset, int origin){
    unsigned nc;
    long rc = 0;

    if(fp->flag & _READ){
        if(origin == 1)
            offset -= fp->cnt;// 找到文件当前位置
        rc = lseek(fp->fd, offset, origin);
        fp->cnt = 0;
    }else if(fp->flag & _WRITE){
        // 缓冲区有内容，先把缓冲区内容写入文件
        if((nc = fp->ptr - fp->base) > 0)
            if(write(fp->fd, fp->base, nc) != nc)
                rc = -1;
        if(rc != -1)
            rc = lseek(fp->fd, offset, origin);
    }
    return (rc == -1) ? -1 : 0;
}
```  

## 练习8-5  
### 问题描述  
修改fsize程序，打印 i 结点项中包含的其他信息。  
### 问题解答  
``` C
#include<stdio.h>
#include<string.h>
#include<fcntl.h>
#include<sys/sys/types.h>
#include<sys/stat.h>
#include "dirent.h"

int stat(char *s, struct stat *buf);
void dirwalk(char *name, void (*fnc)(char *nm));

void fsize(char *name){
    struct stat stbuf;
    if(stat(name, &stbuf) == -1){
        fprintf(stderr, "fsize: can't access %s\n", name);
        return;
    }
    // 如果是目录，则遍历目录中的所有文件
    if((stbuf.st_mode & S_IFMT) == S_IFDIR)
        dirwalk(name, fsize);
    printf("%5u %6o %3u %8ld %s\n", stbuf.st_ino, stbuf.st_mode, stbuf.st_nlink, stbuf.st_size, name);
}
```  

## 练习8-6  
### 问题描述  
标准库函数calloc(n, size)返回一个指针，它指向n个长度为size的对象，且所有分配的存储空间都被初始化为0,。通过调用或修改malloc函数来实现calloc函数。  
### 问题解答  
``` C
#include "syscalls.h"

void *calloc(unsigned n, unsigned size){
    unsigned i, nb;
    char *p, *q;
    nb = n * size;// 字节总数
    if((p = q = malloc(nb)) != NULL)
        // 初始化
        for(i = 0; i < nb; ++i)
            *p++ = 0;
    return q;
}
```  

## 练习8-7  
### 问题描述  
malloc接收对存储空间的请求时，并不检查请求长度的合理性。而free则认为被释放的块包含一个有效的长度字段。改进这些函数，使它们具有错误检查功能。  
### 问题解答  
``` C
#include "syscalls.h"

#define MAXBYTES (unsigned) 10240 // 限制最大申请空间

static unsigned maxalloc; // 截至当时的最大内存块长度
static Header base;// 从空链表开始
static Header *freep = NULL; // 空闲链表初始指针

void *malloc(unsigned nbytes){
    Header *p, *prevp;
    static Header *morecore(unsigned nunits);
    unsigned nunits;

    if(nbytes > MAXBYTES){
        fprintf(stderr, "alloc: can't allocate more than %u bytes\n", MAXBYTES);
        return NULL;
    }
    // 对齐分配空间大小
    nunits = (nbytes + sizeof(Header) - 1) / sizeof(Header) + 1;
    // 没有空闲链表
    if((prevp = freep) == NULL){
        base.s.ptr = freep = prevp = &base;
        base.s.size = 0;
    }
    for(p = prevp->s.ptr; ; prevp = p, p = p->s.str){
        // 找到一个够大的块
        if(p->s.size >= nunits){
            if(p->s.size == nunits)// 正好等于
                prevp->s.ptr = p->s.ptr;
            else{// 该块比较大，则将尾部切出去，只修改该块大小即可
                p->s.size -= nunits;
                p += p->s.size;
                p->s.size = nunits;
            }
            freep = prevp;
            return (void *)(p + 1);// 跳过头部
        }
        if(p == prevp)// 闭环，没有空闲块，则重新申请
            if((p = morecore(nunits)) == NULL)
                return NULL;
    }
}

#define NALLOC 1024 // 申请的最小单元，为了防止频繁申请，所以限制最小值

static Header *morecore(unsigned nu){
    // UNIX系统调用sbrk(n)返回一个指针，指向n个字节的存储空间
    char *cp, *sbrk(int);
    Header *up;
    // 限定最小值
    if(nu < NALLOC)
        nu = NALLOC;
    cp = sbrk(nu * sizeof(Header));
    if(cp == (char *)-1)// sbrk函数失败返回-1，所以要强制类型转换
        return NULL;
    up = (Header *) cp;
    up->s.size = nu;
    maxalloc = (up->s.size > maxalloc) ? up->s.size : maxalloc;
    free((void *)(up + 1));
    return freep;
}

void free(void *ap){
    Header *bp, *p;
    bp = (Header *)ap - 1;// 找到头部
    // 因为maxalloc记录了所申请过的最大块的大小，所以如果该块大于该值，则报错返回
    if(bp->s.size == 0 || bp->s.size > maxalloc){
        fprintf(stderr, "free: can't free %u units\n", bp->s.size);
        return;
    }
    // 寻找插入位置
    for(p = freep; !(bp > p && bp < p->s.ptr); p = p->s.ptr){
        if(p >= p->s.ptr && (bp > p || bp < p->s.ptr))
            break;
    }
    // 若相邻，则合并
    if(bp + bp->s.size == p->s.ptr){
        bp->s.size += p->s.ptr->s.size;
        bp->s.ptr = p->s.ptr->s.ptr;
    }else
        bp->s.ptr = p->s.ptr;
    // 若相邻，则合并
    if(p + p->s.size == bp){
        p->s.size += bp->s.size;
        p->s.ptr = bp->s.ptr;
    }else
        p->s.ptr = bp;
    freep = p;
}
```  

## 练习8-8  
### 问题描述  
编写函数bfree(p, n)，释放一个包含n个字符的任意块p，并将它放入由malloc和free维护的空闲块链表中。通过使用bfree，用户可以在任意时刻向空闲块中添加一个静态或外部数组。  
### 问题解答  
``` C
#include "syscalls.h"

unsigned bfree(char *p, unsigned n){
    Header *hp;
    // 块太小
    if(n < sizeof(Header))
        return 0;
    hp = (Header *)p;// 指针转化为Header*
    hp->s.size = n / sizeof(Header);// 计算块大小
    free((void *)(hp + 1));
    return hp->s.size;
}
```  
