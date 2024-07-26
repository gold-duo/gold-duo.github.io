---
title: Add the notion of the expression statement
date: 2024-07-23
categories: [compiler,chibicc]
tags: [chibicc,chibicc] 
---

直接翻译过来是“添加表达式概念语句”，没搞明白他写这个commit的作用是什么。


## 知识点

从他的代码修改看是为了在codegen中移除遍历`Node`时每次调用`gen`后再生成附加的`pop`指令


### 一.parse部分修改

添加了一个`ND_EXPR_STMT`节点类型用于在构造纯表达式语句。

```c
static Node *stmt(void) {
  //...
  Node *node = new_unary(ND_EXPR_STMT, expr());
  expect(";");
  return node;
}
```

### 二.codegen部分修改

codegen部分除了上面提到的移除遍历`Node`时每次调用`gen`后再生成附加的`pop`指令，其次是gen按照`ND_EXPR_STMT`节点类型生成相应的指令。

```c
switch (node->kind) {
//..
case ND_EXPR_STMT:
    gen(node->lhs);
    printf("  add rsp, 8\n");
    return;
//...
}
```

这里`ND_EXPR_STMT`节点为什么生成的是一条` add rsp, 8`指令。这需要搞明白在x86中 push、pop实际是cpu提供的简化指令。

#### `push x`等价于

```nasm
sub rsp,8
mov rsp,x
;这跟RISC-V中有点像
;   addi sp, sp, -8
;   sd x, 0(sp)

```

#### `pop x`等价于

```nasm
mov x,rsp
add rsp,8
;这跟RISC-V中有点像
;   addi sp, sp, 8
;   ld x,0(sp)
```

#### ND_EXPR_STMT节点生成代码的目的

再回来看看一个语句(stmt)定义：除了可能是`return`语句就或者是纯表达式`expr`。而`return`语句生成的指令直接弹出返回值并退出

```c
  switch (node->kind) {
  case ND_RETURN:
    gen(node->lhs);
    printf("  pop rax\n");
    printf("  ret\n");
    return;
  }
```

除了`ND_NUM`节点(这次修改如果输入测试数据是一个纯数字生成的代码就是错误的了)其他`ND_ADD`、`ND_SUB`、`ND_MUL`、`ND_DIV`等的计算结果都是存储到`rax`并且在`gen`函数的结尾为了方便和下一个节点计算采用栈操作(`push rax`)。所以`ND_EXPR_STMT`节点生成的指令`add rsp, 8`说白了就是平衡栈，目的是为了抵消`gen`函数中生成的最后一条`push rax`。

#### 为什么需要平衡栈呢？

- 先来了解下`rip/eip`

  `rip/eip`指令指针寄存器(ip=instruction pointer)，如它的名字作用也很明显保存着cpu要执行的下一条指令的内存地址.

- 再来看看`call`指令做了什么

  调用`call`指令实际等价于

  ```nasm
  push eip
  jmp x_fun_addr
  ```

- 最后来回忆下前一节`ret`指令做了什么

  ret会将**返回地址弹出堆栈**，并跳转到该地址。
  
  可见这里弹出**返回地址**实际就是`call`指令压入的`eip`.

至此解释了为什么要平衡栈。之所以这么绕是因为他的codegen至今为止还是比较简单，生成的main函数的汇编代码缺少了典型的函数汇编代码模版。比如一个函数的汇编代码通常模版是:

```nasm
  push rbp
  mov rbp, rsp
  sub rsp,x*8     ;局部变量空间。这两句可以用enter指令替代，但enter比较慢少有编译器用
  ;...
  mov rsp,rbp     ;恢复原来的栈.这句和下一句intel提供了leave指令
  pop rbp
  ret
```

从这个汇编模版可见：无论一个函数里面有多少本地变量，无论里面你怎么捣腾堆栈(rsp)，只要在退出时恢复rsp就不会影响后续代码的执行。后续作者应该还是要对`ND_EXPR_STMT`节点生成的代码做修改。