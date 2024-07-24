---
title: Add "return" statement
date: 2024-07-22
categories: [compiler,chibicc]
tags: [chibicc,chibicc] 
---

支持`return`语句

## 知识点

### 一.tokenize部分

tokenize部分为了切割出`return`关键词非常粗暴的通过判断字符串的开头是否`return`

```c
Token *tokenize(void) {
  //...
  while (*p) {
    // Skip whitespace characters.
    if (isspace(*p)) {
      p++;
      continue;
    }
    // Keywords
    if (startswith(p, "return") && !is_alnum(p[6])) {//is_alnum判断是否字母、数组、下划线
      cur = new_token(TK_RESERVED, cur, p, 6);
      p += 6;
      continue;
    }
    //...
  }
}
```

可见现在是只支持关键字`return`，后续要实现其他关键字还得改逻辑。

### 二.parse部分

`return`语句在表达式`expr`前加一个`return`关键字，来看看语法定义

```c
// stmt = "return" expr ";"| expr ";"
```
从这个定义的顺序就可以看出语法先判断是否为`return`语句，再判断是否为纯表达式语句。

```c
static Node *stmt(void) {
  if (consume("return")) {
    Node *node = new_unary(ND_RETURN, expr());
    expect(";");
    return node;
  }
  Node *node=expr();
  expect(";");
  return node;
}

```

### 三.codegen部分

他对return 语句的代码生成也非常简单粗暴。直接生成`ret`指令退出函数，这种实现在当前情况下没有什么问题，不过后续函数要支持局部变量又需要修改实现了。

- `ret`指令

ret会将返回地址弹出堆栈，并跳转到该地址。

- 实现

```c
static void gen(Node *node) {
  switch (node->kind) {
  case ND_NUM:
    printf("  push %ld\n", node->val);
    return;
  case ND_RETURN:
      printf("  pop rax\n");
      printf("  ret\n");
      return;
  }
  //...
}
```