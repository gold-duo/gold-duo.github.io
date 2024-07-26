---
title: Support single-letter local variables
date: 2024-07-24
categories: [compiler,chibicc]
tags: [chibicc,chibicc] 
---

支持单字母本地变量。需要注意的是他这里只支持小写字母。

## 知识点

这次commit主要的修改还是parse和codegen，tokenize有小的修改

### 一.tokenize部分修改

tokenize部分在`TokenKind`增加了一个`TK_IDENT`(标识符)类型，现在切割的标识符就是 a-z 的字母

```c
Token *tokenize(void) {
  //..
  while (*p) {
    // Skip whitespace characters.
    //...
    // Keywords
    //...
    // Identifier
    if ('a' <= *p && *p <= 'z') {
      cur = new_token(TK_IDENT, cur, p++, 1);
      continue;
    }
    //..
  }
}
```

相应的增加一个消耗标识符的函数

```c
Token *consume_ident(void) {
  if (token->kind != TK_IDENT)
    return NULL;
  Token *t = token;
  token = token->next;
  return t;
}
```

### 二.parse部分修改

既然是本地变量那么就涉及到变量的赋值(现在还没有变量定义的语法),作者给的语法定义是这样的

```c
// expr = assign
// assign = equality ("=" assign)?
// primary = "(" expr ")" | ident | num 
```

原来的 `expr=equality` 改成了 `expr = assign`。

赋值到现在来说优先级是最低的，所以`expr`要先parse `assign`表达式。

`ident`和`num`一样归属到`primary`是典型写法。

```c
static Node *assign(void) {
  Node *node = equality();
  if (consume("="))
    node = new_binary(ND_ASSIGN, node, assign());
  return node;
}

static Node *primary(void) {
  if (consume("(")) {
    Node *node = expr();
    expect(")");
    return node;
  }

  Token *tok = consume_ident();
  if (tok)
    return new_var_node(*tok->str);

  return new_num(expect_number());
}
```

### 三.codegen部分修改

codegen部分涉及四个方面：栈空间的分配、怎样根据变量寻址、怎样对变量赋值、怎样获取到变量值

#### 1.栈空间的分配

由于变量名只能是从 a-z 的单字母，不管3721他直接在栈上分配`208`个空间，208正好是26个字母变量每个占用8个字节

```c
  printf("  sub rsp, 208\n");
```

RISC-V 版

```c
  printf("  addi sp, sp, -208\n");
```

#### 2.根据变量寻址

既然26个字母的空间都分配好了，那么 a-z 变量就跟数组一样从 0-25 排列访问就可以了

```c
static void gen_addr(Node *node) {
    //...
    int offset = (node->name - 'a' + 1) * 8;
    printf("  lea rax, [rbp-%d]\n", offset);
    printf("  push rax\n");
    return;
    //...
}
```

RISC-V 版

```c
static void genAddr(Node *Nd) {
  //...
  int offset = (node->name - 'a' + 1) * 8;
  printf("  addi a0, fp, %d\n", -Offset);
  return;
  //...
}
```

#### 3.变量赋值

搞定了前面的变量寻址，变量赋值就是将等号右边表达式的值拷贝到变量的地址空间里

```c
static void store(void) {
  printf("  pop rdi\n");        //弹出右边=表达式值
  printf("  pop rax\n");        //弹出变量地址
  printf("  mov [rax], rdi\n"); //赋值到变量地址里
  printf("  push rdi\n");  
}

static void gen(Node *node) {
  switch (node->kind) {
    //...
    case ND_ASSIGN:
      gen_addr(node->lhs);      
      gen(node->rhs);
      store();
      return;
  }
  //...
}
```

RISC-V 版

```c
static void gen(Node *node) {
  switch (node->kind) {
    //...
    case ND_ASSIGN:
      gen_addr(node->lhs);
      push();//将地址压入栈中
      gen(node->rhs);
      pop("a1");//弹出地址到a1
      printf("  sd a0, 0(a1)\n");//将右值存储到地址中
      return;
  }
  //...
}
```

#### 4.获取变量值

由于少了右边的表达式，获取变量值比变量赋值更简单了，只要拿到变量的地址取出地址里的数据即可

```c
static void load(void) {
  printf("  pop rax\n");        ///弹出变量地址
  printf("  mov rax, [rax]\n"); //取出地址里的数据
  printf("  push rax\n");       
}
static void gen(Node *node) {
  switch (node->kind) {
    //...
    case ND_VAR:
      gen_addr(node);
      load();
      return;
    //..
  }
}
```

RISC-V 版

```c
static void gen(Node *node) {
  switch (node->kind) {
    //...
    case ND_VAR:
      gen_addr(node);
      printf("  ld a0, 0(a0)\n");//a0地址中存储的数据加载到a0
      return;
    //..
  }
}
```

### 5.其他

这次他生成的汇编是一个典型的函数模版（我在[Add the notion of the expression statement](https://gold-duo.github.io/posts/chibicc-Add-the-notion-of-the-expression-statement/)中提到）。由于之前`ND_RETURN`节点直接用`ret`指令跳出，这次由于分配了栈空间给26个字母变量所以不能直接这么退出了，他改成了跳转label的方式

```c
static void gen(Node *node) {
  //...
  switch (node->kind) {
  //...
  case ND_RETURN:
      gen(node->lhs);
      printf("  pop rax\n");
      printf("  jmp .L.return\n");
      return;
   //...
  }
}
void codegen(Node *node) {
  //...
  printf("  push rbp\n");
  printf("  mov rbp, rsp\n");
  printf("  sub rsp, 208\n");

  //...
  printf(".L.return:\n");
  printf("  mov rsp, rbp\n");
  printf("  pop rbp\n");
  printf("  ret\n");
}
```

RISC-V 版

```c
static void gen(Node *node) {
  //...
  switch (node->kind) {
  //...
  case ND_RETURN:
      gen(node->lhs);
      //为什么RISC-V 不像x86 pop呢？因为计算不靠栈而是用寄存器
      printf("  j .L.return\n");    //RISC-V直接用j比x86的jmp更省字母
      return;
   //...
  }
}
void codegen(Node *node) {
  //...
  // 将fp压到栈上。fp=Frame Pointer,作用相当于x86的rbp为了退出时恢复sp
  printf("  addi sp, sp, -8\n");
  printf("  sd fp, 0(sp)\n");
  // fp=sp,思路跟 mov rbp,rsp一样
  printf("  mv fp, sp\n");
  //分配栈空间
  printf("  addi sp, sp, -208\n");


  //...


  printf(".L.return:\n");
  // 将fp的值改写回sp（丢弃掉分配的栈空间），等效于mov rsp, rbp
  printf("  mv sp, fp\n");
  // 弹出fp,恢复fp。等效于pop rbp
  printf("  ld fp, 0(sp)\n");
  printf("  addi sp, sp, 8\n");
  printf("  ret\n");
}
```