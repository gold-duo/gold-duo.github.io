---
title: Improve error message
date: 2024-07-03
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---


对上一节的**error**作一些改进，**tokenize**时输出错误位置

## 知识点
### 输出空格

```c
fprintf(stderr, "%*s", n, ""); 
```

### 如何定位到出错处

在Token构造时直接将 tok->str指向相应字符位置，通过这个指针 - 分析符串指针就得到位移，而分析要的字符串是不换行的所以可以通过`%*s`输空格来定位到指定的错误。

## 核心代码

```c
// reports an error location and exit.
void error_at(char *loc,char *fmt,...){
    va_list ap;
    va_start(ap,fmt);
    int pos=loc - user_input;
    fprintf(stderr,"%s\n",user_input);
    fprintf(stderr, "%*s", pos, ""); // print pos spaces.
    vprintf(stderr,"^ ");
    vfprintf(stderr,fmt,ap);
    fprintf(stderr,"\n");
    va_end(ap);
    exit(1);
}

```