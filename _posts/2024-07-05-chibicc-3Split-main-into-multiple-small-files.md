---
title: Split main.c into multiple small files
date: 2024-07-05
categories: [compiler,chibicc]
tags: [chibicc,chibicc] 
---


将main源文件里的词法、语法、代码生成分离成对应的tokenize、parse、codegen文件，而chibi.h头文件存储公共部分。

## 知识点

主要涉及的知识点在Makefile的修改。

### Makefile

Makefile的注释来此[rvcc](https://github.com/sunshaoce/rvcc/blob/d056e53f05c412aadacb7d8427f5cf95ce8a477b/Makefile)

```makefile
# C编译器参数：使用C11标准，生成debug信息，禁止将未初始化的全局变量放入到common段
CFLAGS=-std=c11 -g -fno-common

# C源代码文件，所有的.c结尾的文件
SRCS=$(wildcard *.c)
# C文件编译生成可重定位文件，将所有.c文件替换为同名的.o结尾的文件名
OBJS=$(SRCS:.c=.o)

# chibicc标签，表示如何构建最终的二进制文件，依赖于所有的.o文件
# $@表示目标文件，此处为chibicc，$^表示依赖文件，此处为$(OBJS)
chibicc: $(OBJS)
# 将多个*.o文件编译为chibicc
	$(CC) $(CFLAGS) -o $@ $^ $(LDFLAGS)

# 所有的可重定位文件依赖于chibi.h的头文件
$(OBJS): chibi.h

# 测试标签，运行测试脚本
test: chibicc
	./test.sh

# 清理标签，清理所有非源代码文件
clean:
	rm -f chibicc *.o *.s tmp* a.out

# 伪目标，没有实际的依赖文件
.PHONY: test clean
```