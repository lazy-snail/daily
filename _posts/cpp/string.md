---
title: string
date: 2018-08-16 14:40:56
categories: cpp
tags: cpp
---
[toc]
# 构成
C++ 里 string 包括：
* C++标准库中定义的 string、wstring、u16string、u32string；
* C-style 的 C-string：char *、const char *。

# 拷贝方式
## str.copy()
C++ 标准库 string 提供的函数。完整声明和示例：
```c++
copy(_CharT* __s, size_type __n, size_type __pos) const

//
char buffer[20];
string str("Test string...");
size_t length = str.copy(buffer, 6, 5);
buffer[length] = '\0';
cout << "buffer contains: " << buffer << '\n';

// output：
buffer contains: string
```

## C 标准库函数
### strcpy strncpy
C 标准库 <cstring\> 提供的函数：strcpy()、strncpy()。其中，strcpy 直到遇到 NULL('\n')才会结束，否则一直拷贝；strncpy 则提供额外参数，如果拷贝 n 个字节之前遇到 NULL，则停止拷贝，否则，只拷贝 n 个字节后停止。
不踩内存则意味着没有结束符，因此最好为 dst 多分配一个字节并置为 NULL 作为结束符。
```c++
string src ="abcdabcd";
char* dst = new char[8];

strcpy(dst, src.c_str()); // 踩内存，src 多出一个 NULL，而 dst 分配的内存不够
strncpy(dst, src.c_str(), 8);// 不踩内存
memcpy(dst, src.c_str(), 8); // 不踩内存
```

### 内存拷贝
C 标准库 <cstring\> 同时提供了内存拷贝函数：memcpy、memccpy、memmove。三者功能相同，参数上、对结束符处理、内存区域覆盖的处理有所区别。
**memcpy()**
函数原型：
```c++
extern void *memcpy(void *dest, void *src, unsigned int count)
```
参数说明：dest 为目的字符串，src 为源字符串，count 为要拷贝的字节数。
函数功能：将字符串 src 中的前 n 个字节拷贝到 dest 中。
返回说明：src 和 dest 所指内存区域不能重叠，函数返回 void* 指针。

**memccpy()**
函数原型：
```c++
extern void *memccpy(void *dest, void *src, unsigned char ch, unsigned int count)
```
参数说明：dest 为目的字符串，src 为源字符串，ch 为终止复制的字符(即复制过程中遇到 ch 就停止复制)，count 为要拷贝的字节数。
函数功能：将字符串 src 中的前 n 个字节拷贝到 dest 中，直到遇到字符 ch 便停止复制。
返回说明：src 和 dest 所指内存区域不能重叠，函数返回 void* 类型指针。

**memmove()**
函数原型：
```c++
extern void *memmove(void *dest, const void *src, unsigned int count)
```
参数说明：dest 为目的字符串，src 为源字符串，count 为要拷贝的字节数。
函数功能：将字符串 src 中的前 n 个字节拷贝到 dest 中。
返回说明：src 和 dest 所指内存区域可以重叠，重叠可能导致 src 内容发生变化。函数返回 void* 类型指针。