---
title: Add + and - operators
date: 2024-07-01
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---


添加+、-操作符。这节他巧妙的用strtol将一串字符里的数字分解并转换成数字。


## 知识点

### strtol函数
- 入参：
    - 1.输入字符
    - 2.成功后指向下一个要转换的字符
    - 3.输入字符的进制
- 出参：long int
- 实例

```c
printf("    mov rax, %ld\n",strtol(*p,&p,10));
```

## 汇编知识

### RISCV-V指令的加减

- 在x86中立即数和寄存器相加后的结果返回到寄存器中.比如`add rax,66`。但是RISCV-V的的立即数加法除了要有加法寄存器还有返回值寄存器

```nasm
addi a0,a0,66 ;第一个a0=第二个a0+66
```

- x86有sub的减法指令，RISCV-V的减法则在addi的基础上自行将立即数取负数

```nasm
addi a0,a0,-66 ;a0-66
```