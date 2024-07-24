---
title: Compile an integer to an exectuable that exits with the given number
date: 2024-06-28
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---

开篇通.个人学习rui314大佬的[chibicc](https://github.com/rui314/chibicc/commits/main/)笔记。

这一节是通过c语言用printf打印出汇编代码。可能是为了简化chibicc没有后端，生成的中间代码是汇编，再通过汇编器(as)生成可执行文件。

> 这个系列同时参考了[rvcc](https://github.com/sunshaoce/rvcc),每节同时附带上RISC-V 64的汇编。由于我的环境是mac不保证运行正确😂

## 知识点
### 一.clang命令行参数
- 编译c文件：`clang file -o output`
- 编译汇编文件：`clang file -o output`

> 在mac下需要将段 main 修改成 _main,不然会出现【Undefined symbols for architecture x86_64: "_main"】错误

### 二.C知识点
- 1.main函数的入参

```c
int main(int argc,char **argv){
    //argc：入参个数
    //argv：参数.而argv[0]为程序名称
}
```
- 2.atoi函数：将字符串转换为数字

### 三.汇编知识

```nasm
.intel_syntax noprefix ; Intel 语法的指令
.global main           ; 声明全局段，程序入口
main:                  ; 对应global声明的main段标签
;...
```

- 1.x86汇编

以r开头的是64位寄存器，如 rax 、 rbx 等。

以e开头的是32位寄存器，如 eax 、 ebx（还是十几年前用过）。

在32位中eax默认当作函数的返回值寄存器，相同的64位是rax。

- 2.RISC-V 汇编

```nasm
li a0,999 ;li指令位加载立即数刀寄存器a0,相当于x86的move
;a0 默认当作函数的返回值寄存器
```
### 四.cnmake

- 好吧mac环境没有cnmake得安装
  - (下载)[https://links.jianshu.com/go?to=https%3A%2F%2Fcmake.org%2Fdownload%2F]
  - 添加命令行

    打开cmake，点击菜单栏的 Tools -> How to Install For Command Line Use
    拷贝"sudo "/Applications/CMake.app/Contents/bin/cmake-gui" --install"

    终端中执行拷贝的命令

- 使用

```sh
#1.创建一个编译文件目录
cmake -Bbuild 

#2.编译
cd build/
make

#3.执行编译程序
./chibicc 66

#4.清理编译的文件
rm -r build
```

## 成果

通过c程序输入一个数字参数x，产生一个汇编文件，这个汇编编译后程序运行输出的就是x.

下面是汇编代码：

### 1.x86

```nasm
.intel_syntax noprefix
.global _main       ;mac下main要加下划线
_main:
  mov rax, 66
  ret
```

### 2.RISC-V 

```nasm
.global main      
main:
  li a0, 66
  ret
```