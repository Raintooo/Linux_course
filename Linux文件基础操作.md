- [Linux文件基础操作](#linux文件基础操作)
  - [文件](#文件)
  - [Linux文件编程](#linux文件编程)
  - [ASCII C文件编程](#ascii-c文件编程)
    - [深入ASCII C文件编程](#深入ascii-c文件编程)
    - [ASCII C接口](#ascii-c接口)
    - [ASCII C文件缓冲类型](#ascii-c文件缓冲类型)

# Linux文件基础操作

Linux的设计哲学: 一切接文件

## 文件

什么是文件
* 文件具有永久性存储，按特定字节顺序组成的有命名的数据集
* 文件一般可分为:二进制文件，文本文件
  * 文本文件：每个字节存放ASCII码
    * 存储量大，速度慢，便于对字符操作
  * 二进制文件：数据按照内存中形式原样存放
    * 存储量小，速度快，便于存放中间结果

## Linux文件编程

Linux中，除了常规文件，目录，设备，管道等也属于文件

![linux文件基础操作](./pic/linux文件基础操作1.png)

## ASCII C文件编程

标准C文件接口建立于Linux原生接口之上，利用缓冲机制提高效率  
缓冲区是一片特殊的内存空间，用来暂存文件数据：
  * 读：可一次性读取大量数据到缓冲区，方便后续快速从缓冲区读取
  * 写：可以先写入缓冲区，写满缓冲区后再写入文件
缓冲区的引入旨在避免频繁操作磁盘，提高文件读写效率

### 深入ASCII C文件编程
由于加入缓冲区概念，ASCII C文件编程是一种数据流的编程

![linux文件基础操作](./pic/linux文件基础操作2.png)

### ASCII C接口

```C
/* Open a file and create a new stream for it.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern FILE *fopen (const char *__restrict __filename,
		    const char *__restrict __modes) __wur;

/* Close STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern int fclose (FILE *__stream);

/* Read chunks of generic data from STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern size_t fread (void *__restrict __ptr, size_t __size,
		     size_t __n, FILE *__restrict __stream) __wur;
/* Write chunks of generic data to STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern size_t fwrite (const void *__restrict __ptr, size_t __size,
		      size_t __n, FILE *__restrict __s);

/* Write a string to STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern int fputs (const char *__restrict __s, FILE *__restrict __stream);

/* Get a newline-terminated string of finite length from STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern char *fgets (char *__restrict __s, int __n, FILE *__restrict __stream)
     __wur;
```

文件打开模式  
![linux文件基础操作](./pic/linux文件基础操作3.png)

文件读写指针移动：  

```C
/* Seek to a certain position on STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern int fseek (FILE *__stream, long int __off, int __whence);
/* Return the current position of STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern long int ftell (FILE *__stream) __wur;
// 获取当前文件指针位置

/* Rewind to the beginning of STREAM.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern void rewind (FILE *__stream);
// 将文件指针复位到起始位置
```

### ASCII C文件缓冲类型

* 全缓冲类型：默认缓冲区大小为```BUFSIZ```(与系统有关)
  * 缓冲区满或调用```flush```后才写入磁盘
* 行缓冲类型：默认缓冲区为128字节(与系统有关)
  * 遇见换行符或者缓冲区满或调用```flush```后才写入磁盘
* 无缓冲类型：不对数据进行缓冲

```C
/* If BUF is NULL, make STREAM unbuffered.
   Else make it use buffer BUF, of size BUFSIZ.  */
extern void setbuf (FILE *__restrict __stream, char *__restrict __buf) __THROW;
// 设置大小为BUFSIZ的缓冲区

#ifdef	__USE_MISC
/* If BUF is NULL, make STREAM unbuffered.
   Else make it use SIZE bytes of BUF for buffering.  */
extern void setbuffer (FILE *__restrict __stream, char *__restrict __buf,
		       size_t __size) __THROW;
// 设置大小为size的buf为缓冲区

/* Make STREAM line-buffered.  */
extern void setlinebuf (FILE *__stream) __THROW;
// 设置为行缓冲区

/* Make STREAM use buffering mode MODE.
   If BUF is not NULL, use N bytes of it for buffering;
   else allocate an internal buffer N bytes long.  */
extern int setvbuf (FILE *__restrict __stream, char *__restrict __buf,
		    int __modes, size_t __n) __THROW;

//设置 缓冲区模式为mode, 大小为size的buf的缓冲区
// #define _IOFBF 0		/* Fully buffered.  */
// #define _IOLBF 1		/* Line buffered.  */
// #define _IONBF 2		/* No buffering.  */
#endif
```