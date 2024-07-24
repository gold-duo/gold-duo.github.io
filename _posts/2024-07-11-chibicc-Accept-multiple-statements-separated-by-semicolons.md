---
title: Accept multiple statements separated by semicolons
date: 2024-07-11
categories: [compiler,chibicc]
tags: [chibicc,chibicc] 
---


通过分号分割多个语句。

## 知识点

分号(`;`)属于c函数`ispunct`处理的符号，所以tokenize不需要修改。codegen有小修改，稍微大点的修改是parse.

### 一.parse修改

之前处理的输入都是单语句，现在可以处理多语句。来看他的语法分析部分

```c
// program = stmt*
// stmt = expr ";"   
```

一个语句(stmt)由表达式(expr)以分号结尾，而一个程序(program)可以有多个语句构成。

既然一个程序可以由多个语句构成，意味着需要构造多个 `stmt`节点，形成一个 链表。构造的`stmt`节点它有单一的子节点，不像之前的 relational、add、mul构造的节点是具备lhs、rhs子节点的。作者为了方便在原来的`Node` 上直接扩展加了 `next` 字段。来看下链表的实现

```c
Node *program(void){
  Node head={};
  Node *cur= &head;
  while(!at_eof()){
    cur->next=stmt();
    cur=cur->next;
  }
  return head.next;
}

static Node *stmt(void){
  Node *node=expr();
  expect(";");
  return node;
}

```

### 二.codegen修改

codegen从之前的单次`gen`调用，变成了遍历链表再逐条语句`stmt`调用`gen`。

这里需要注意的是由于在`gen`函数的结尾生成的汇编代码将语句的返回值默认压到栈顶,生成的汇编代码本质上是一个函数(main函数)程序退出也就是该函数的退出。所以之前他在`codegen`函数结尾生成一条`pop rax`指令弹出栈顶数据将语句的返回值当作程序退出时的返回值。

```c
static void gen(Node *node) {
  //...
  printf("  push rax\n");
}
void codegen(Node *node) {
  //...
  gen(node);
  printf("  pop rax\n");
  printf("  ret\n");
}
```

虽然这次他修改支持多语句，但是返回值只支持最后一条语句的。

```c
void codegen(Node *node) {
  //...
  for(Node *n=node; n;  n=n->next){
    gen(n);
    printf("  pop rax\n");
  }
  printf("  ret\n");
}
```
