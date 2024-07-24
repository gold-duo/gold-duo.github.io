---
title: Add unary plus and minus
date: 2024-07-04
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---


增加+、- 一元符号。当组合成 ++、-- 时并不是C语言里的递增和递减，而是数学意义上的取正数和取反(或者简单的理解为正负符号)。


## 知识点

### 一.数学意义的正负

`+num`单独出现那在数学中是一个正数。正数不需要运算，其值就是它自己；

`-num`单独出现那在数学中是一个负数。负数就是在正数上取反运算，用数学表达那可以是:`0-num`(是不是有点绕😂).

### 二.怎么区分是加号还是一元符

`2--3`怎么区分2后面的是减号和3前面的就是一元符号呢？

答案可从两个方面得到

- 1.语法分析中

```c
  //expr = mul ("+" mul | "-" mul)*  
```

expr中第一个mul匹配到的是primary 2,第二个匹配的是 -符号最后是mul表达式，由于这里匹配到了`-`所以这里是减号

- 2.数学运算的规律

加减乘除时数字前的符号优先于运算符：2--3 =  2-(-3)


### 三.语法分析

词法上并不需要修改，但语法上他增加了一个unary,由于一元符的优先级仅次于括号(primary)但高于乘除，所以语法分析修改还涉及到了mul

```c
// mul = unary ("*" unary | "/" unary)*         //组成：一元、(0或多个 乘或除符号、一元)
// unary = ("+" | "-")? unary | primary         //组成：1.(可选 加减号) 一元； 2.基数
static Node *mul(void) {
  Node *node = unary();
  for (;;) {
    if (consume('*'))
      node = new_binary(ND_MUL, node, unary());
    else if (consume('/'))
      node = new_binary(ND_DIV, node, unary());
    else
      return node;
  }
}
static Node *unary(void) {
  if (consume('+'))
    return unary();
  if (consume('-'))
    return new_binary(ND_SUB, new_num(0), unary()); //取反相当于0-x
  return primary();
}
```

### 四.扩展：构造 **-** 一元节点,用 `neg`指令取反

- [rvcc](https://github.com/sunshaoce/rvcc)

在rvcc中 `unary` 将 - 构造出特定的一元节点。因为数字取反有单独的汇编指令 `neg`，不过这么一来的话就需要同步修改gen。

```c
static Node *unary(Token **Rest, Token *Tok) {
  // ...
  if (equal(Tok, "-"))
    return newUnary(ND_NEG, unary(Rest, Tok->Next));

  // primary
  return primary(Rest, Tok);
}

static void genExpr(Node *Nd) {
  switch (Nd->Kind) {
  // 。。。
  case ND_NEG:
    genExpr(Nd->LHS);
    printf("  neg a0, a0\n");  //RISC-V中neg操作两个寄存器，第一个返回值，第一个入参
    return;
  //。。。
  }
}
```

- chibicc如何修改

chibicc中构造语法树时遇到一元 `-`相应的构造一个`ND_NEG`节点即可，有点改动的是代码生成。因为至今为止除了`ND_NUM`节点其余类型的节点都是包含左右子节点的并将左右节点的值保存在`rdi、rax`中，`ND_NEG`节点只用到了左子节点。当然最直接的是在`if (node->kind == ND_NUM)`增加`ND_NEG`节点处理，不过这个有点短路编译器也总是报`switch`还有`case`没处理，所以稍微加点逻辑处理。

```c
static Node *new_neg(Node *lhs){
  Node *node = new_node(ND_NEG);
  node->lhs = lhs;
  return node;
}
static Node *unary(void) {
  if (consume('+'))
    return unary();
  if (consume('-'))
    return new_neg(unary());
  return primary();
}

static void gen(Node *node) {
  if(node->lhs!=NULL)
    gen(node->lhs);

  if(node->rhs!=NULL)
    gen(node->rhs);

  if(node->lhs!=NULL)
    printf("  pop rdi\n");

  if(node->rhs!=NULL)
    printf("  pop rax\n");

  switch (node->kind) {
  case ND_NUM:
    printf("  push %ld\n", node->val);
    return;
  //...
  case ND_NEG:
    printf("  neg rdi\n");
    printf("  push rdi\n");  //压栈返回，不然多了条 mov 指令
    return;
  }

  printf("  push rax\n");
}
```