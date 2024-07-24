---
title: Add *, / and ()
date: 2024-07-03
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---

这节实现了简单的加减乘除计算器。有基本的词法、语法、代码生成的三个过程。

## 知识点

### 一.C语言
- ispunct:判断字符是否为标点符号(ctype.h)

### 二.汇编

- x86
   - cdq

        CDQ（Convert Double to Quad，将双字数据扩展为四字），通常出现在除法运算之前.将符号 EAX 位扩展到 EDX 中（有点绕不太明白了）。

    - idiv
    
        有符号除(imul有符号乘)

    ```nasm
        mov  eax, 1234        ;idiv书写时只涉及一个寄存器，所以被除数存放在eax/rax中
        mov  ecx, 17          ;ecx除数
        cdq                   ; EDX = signbit(EAX)
        idiv  ecx             ; EAX = 1234/17     EDX = 1234%17
    ```

- RISC-V
    - mul、div
    
        mul、div在RISCV-V中都是有符号的

    ```nasm
        mul a0, a0, a1  ; a0 = a0*a1 
        div a0, a0, a1  ; a0 = a0/a1,貌似没有哪个寄存器存放余数
    ```

    - sd、ld指令

        sd(store doubleword)将寄存器的值存储到内存中.ld则是sd的反操作。
    ```nasm
        ;1.sd
        ;   sd x, offset(y),x和y是寄存器，分别表示源操作数和基地址。offset是一个12位的立即数，表示从基地址y偏移的字节数。
        sd a0, 0(sp) ;  a0的数据存储到sp中无偏移动

        ;2.ld
        ld a1, 0(sp) ; sp中无偏移数据加载到a1
    ```
    - 栈操作

        RISC-V没有压栈弹栈指令。RISC-V中压栈和弹栈除了需要手动操作栈指针（压栈 **sp-8**(64位)，弹栈 **sp+8**），还要用**sd/ld**指令将数据加载到栈中或从栈取数据到寄存器。

    ```nasm
        ;压栈(将a0的数据压入栈中)
        addi sp, sp, -8         ;下压栈
        sd a0, 0(sp)

        ;弹栈(将压入栈中数据弹出到)
        addi sp, sp, +8         ;上增栈
        ld a1,0(sp)
    ```

### 三.语法分析方面(expr)

他跳过了BNF只给了注释默认读者明白 expr、mul、primary的遍历逻辑。这个写法拿个yacc、bison或者ANTLR（[calculator](https://github.com/antlr/grammars-v4/blob/master/calculator/calculator.g4#L42)）的语法分析源码对照基本可以知道一二.因为他手写的是递归下降parser所以在解析表达式时优先级越高的越靠后。

```c
// expr = mul ("+" mul | "-" mul)*              //组成：乘数、（0或多个 加或减号、乘数）
// mul = primary ("*" primary | "/" primary)*   //组成：基数、(0或多个 乘或除号、基数)
// primary = "(" expr ")" | num                 //组成：1，括号包围表达式；2.单纯的数字
static Node *expr(void) {
  Node *node = mul();
  for (;;) {
    if (consume('+'))
      node = new_binary(ND_ADD, node, mul());
    else if (consume('-'))
      node = new_binary(ND_SUB, node, mul());
    else
      return node;
  }
}

static Node *mul(void) {//叫mul实际还负责div，有点迷惑人的😂
  Node *node = primary();
  for (;;) {
    if (consume('*'))
      node = new_binary(ND_MUL, node, primary());
    else if (consume('/'))
      node = new_binary(ND_DIV, node, primary());
    else
      return node;
  }
}

static Node *primary(void) {
  if (consume('(')) {
    Node *node = expr();
    expect(')');
    return node;
  }
  return new_num(expect_number());
}
```

### 代码生成(gen)

分别递归左右节点完了看看根节点是否要做加减乘除。当然他这里是粗暴的代码生成：负责计算的节点可能是数字也可能是个表达，所以他通过2个寄存器来运算这就涉及到寄存器的保存和恢复了(如果单单是数字可以只用一个寄存器)。

```c
static void gen(Node *node) {
  if (node->kind == ND_NUM) {
    printf("  push %ld\n", node->val);
    return;
  }

  gen(node->lhs);
  gen(node->rhs);

  printf("  pop rdi\n");
  printf("  pop rax\n");

  switch (node->kind) {
  case ND_ADD:
    printf("  add rax, rdi\n");
    break;
  case ND_SUB:
    printf("  sub rax, rdi\n");
    break;
  case ND_MUL:
    printf("  imul rax, rdi\n");
    break;
  case ND_DIV:
    printf("  cqo\n");
    printf("  idiv rdi\n");
    break;
  }

  printf("  push rax\n");
}
```