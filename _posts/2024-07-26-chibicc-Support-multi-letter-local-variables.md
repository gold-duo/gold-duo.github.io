---
title: Support multi-letter local variables
date: 2024-07-26
categories: [compiler,chibicc]
tags: [chibicc,chibicc] 
---


这个commit是上一个[《Support single-letter local variables》](https://gold-duo.github.io/posts/chibicc-Support-single-letter-local-variables/)的升级完善。上一个commit基本已经搭好了大体的框架，此次修改也不是很大。

## 知识点

支持多字母变量面临主要的问题还是：如何分配变量地址、如何寻址。还好基本他在 `parse`解决了。

### 一.tokenize修改

```c
Token *tokenize(void) {
    //...
    // Identifier
    if (is_alpha(*p)) {
      char *q = p++;
      while (is_alnum(*p))
        p++;
      cur = new_token(TK_IDENT, cur, q, p - q);
      continue;
    }

    // Multi-letter punctuators
    //...
}
```

`Identifier`典型的字母开头后续允许数字、字母、下划线的

### 二.parse、main修改

允许多字母变量那么就不能再用固定分配26个变量的方式，需要构建一张变量表。

```c
//parse.c
Var *locals;//变量链表
static Var *find_var(Token *tok) {
  for (Var *var = locals; var; var = var->next)
    if (strlen(var->name) == tok->len && !strncmp(tok->str, var->name, tok->len))
      return var;
  return NULL;
}
static Node *new_var_node(Var *var) {
  Node *node = new_node(ND_VAR);
  node->var = var;                      //Node节点持有Var信息（上次commit中Node节点的name替换成var）
  return node;
}

static Var *new_lvar(char *name) {      //构建变量链表
  Var *var = calloc(1, sizeof(Var));
  var->next = locals;
  var->name = name;
  locals = var;
  return var;
}
static Node *primary(void) {
  //...

  Token *tok = consume_ident();
  if (tok) {
    //查找变量是否在变量表里，不在则新建并添加到链表里
    Var *var = find_var(tok);
    if (!var)
      var = new_lvar(strndup(tok->str, tok->len));
    return new_var_node(var);
  }

  return new_num(expect_number());
}
```

那么又是怎么计算出变量在栈上的偏移和需要分配多少栈空间呢？
第一个疑问的解决方案是在`Var` 持有`offset`通过其在链表的位置赋值一个`n*8`的值;分配的栈空间大小那么就是链表总变量数*8

```c
//chibi.h
struct Var {
  Var *next;
  char *name; // Variable name
  int offset; // Offset from RBP
};

//main.c
int main(int argc, char **argv) {
  //...
  Function *prog = program();

  int offset = 0;
  for (Var *var = prog->locals; var; var = var->next) {
    offset += 8;
    var->offset = offset;
  }
  prog->stack_size = offset;

  // Traverse the AST to emit assembly.
  codegen(prog);
  return 0;
}
```

上面的函数`program`改成返回的是`Function` struct

```c
//chibi.h
struct Function {
  Node *node;
  Var *locals;
  int stack_size;
};

//parse.c
Function *program(void) {
  locals = NULL;
  //...
  Function *prog = calloc(1, sizeof(Function));
  prog->node = head.next;
  prog->locals = locals;
  return prog;
}
```

`Function`除了持有parse出来的`node`链表、还有变量表和计算出来的要分配的栈空间大小。


### 三.codegen修改

由于 node节点持有变量分配在栈上的`offset`、且要分配的栈大小在 `main`中都提前计算好了，这次 codegen 修改实际非常小。

```c
static void gen_addr(Node *node) {
  //...
    printf(" lea rax, [rbp-%d]\n", node->var->offset);
  //...
}

void codegen(Function *prog){
  //...
  printf("  sub rsp, %d\n", prog->stack_size);

  // Emit code
  for (Node *node = prog->node; node; node = node->next)
    gen(node);

  //..
}
```