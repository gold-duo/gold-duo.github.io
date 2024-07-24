---
title: Add ==, !=, <= and >= operators
date: 2024-07-05
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---

支持 ==、!=、<=、>=比较符号。

## 知识点

### 一.词法部分修改

在判断单一的标点符号前先判断字符是不是 ==、!=、<=、>=。

```c
    //...
    while(*p){
        //...
        if(startswith(p,"==") || startswith(p,"!=") ||startswith(p,"<=")|| startswith(p,">=")){
          cur=new_token(TK_RESERVED,cur,p,2);
          p+=2;
          continue;
        }
        if(ispunct(*p)){
            cur=new_token(TK_RESERVED,cur,p++,1);
            continue;
        }
        //...
    }
    //..
```

### 二.语法分析修改

原来的`expr`变成了新加的`add`，增加equality、relational。
```c
// expr = equality            
// equality = relational ("==" relational | "!=" relational)*
// relational = add ("<" add | "<=" add | ">" add | ">=" add)*
// add = mul ("+" mul | "-" mul)*               //组成：乘数、（0或多个 加或减符号、乘数）
static Node *expr(void) {
    return equality();
}
static Node *equality(void){
  Node *node=relational();
  for(;;){
    if(consume("=="))
      node=new_binary(ND_EQ,node,relational());
    else if(consume("!="))
      node=new_binary(ND_NE,node,relational());
    else
      return node;
  }
}
static Node * relational(void){
  Node *node=add();
  for(;;){
    if(consume("<"))
      node=new_binary(ND_LT,node,add());
    else if(consume("<="))
      node=new_binary(ND_LE,node,add());
    else if(consume(">"))
      node=new_binary(ND_LT,add(),node);//反转节点位置就是 >
    else if(consume(">="))
      node=new_binary(ND_LE,add(),node);
    else 
      return node;
  }
}
```

这里有意思的是他用了一个小技巧将 >、>=转换乘<、<=。😂


### 三.代码生成

在switch中添加相应节点的汇编代码生成。
```c
  case ND_EQ:
    printf("  cmp rax, rdi\n");
    printf("  sete al\n");
    printf("  movzx rax, al\n");
    break;
  case ND_NE:
    printf("  cmp rax, rdi\n");
    printf("  setne al\n");
    printf("  movzx rax, al\n");
    break;
  case ND_LT:
    printf("  cmp rax, rdi\n");
    printf("  setl al\n");
    printf("  movzx rax, al\n");
    break;
  case ND_LE:
    printf("  cmp rax, rdi\n");
    printf("  setle al\n");
    printf("  movzx rax, al\n");
    break;
```
这里需要注意的是他用到的指令`movzb`居然报 "invalid instruction mnemonic 'movzb'"在作者的9cc项目也有人提[issues](https://github.com/rui314/9cc/issues/14)，改为`movzx`即可。

### 四.汇编知识
- 1.x86
    - cmp x1,x2比较两个数据影响到ZF、CF标志位
    - MOVZX零扩展传送,用0天充剩余空间

    ```nasm
    movzx rax,al ;al是低8位，传递到64位rax意味着高56位都填充0
    ```

    - 取ZF标志位指令

    下面的指令从ZF取标志位置，存放到寄存器,通常配合着cmp指令使用

    指令|作用
    ---|---
    sete|==时设置为1
    setne|!=时设置为1
    setl|<时设置为1
    setle|<=时设置为1

    ```nasm
    cmp rax,rdi
    sete al         ;取出是否相等标志位
    movzx rax,al    ;低8位扩展到64位寄存器
    ```

- 2.RISC-V

    - xor r1,r2,r3

      r1= r2 异或 r3。异或两个数(相应位)相同为0，不同为1

      - xori. xor的立即数版本

    - 比较用到的指令

      指令|作用|例子
      ---|---|---
      seqz|等于0则置1
      snez|不等于0则置1
      slt|有符号比较，小于时设置为1|`slt a0,a0,a1`;`a0<a1`则a0=1
      sltu|slt的无符号版本|
      slti|slt的立即数有符号版本|
      sltiu|slt的立即数无符号版本|

    - [RISC-V中如何比较两个数字](https://github.com/sunshaoce/rvcc/blob/965c2c153afb64e9860aed004d228ba5be8f6c6f/main.c#L470)

      RISC-V不像x86一样提供`cmp`指令，判断==、!=、<、<=需要绕点湾子

      - ①.判断 == 和 !=

        由于两个相等数xor等于零(x86通常用xor指令来对寄存器快速清零比如xor rax,rax)，所以可以用来判断 ==、!=

        ```nasm

        xor a0,a0,a1    ; a0 = a0 xor a1
        ; 判断==
        seqz a0,a0      ; 如果 a0=0，则设置a0为1

        ; 判断!=
        snez a0,a0      ; 如果 a0!=0，则设置a0为1。（这里一定要经过snez，因为xor是按位异或不相等值也不一定是1）
        ```

      - ②.判断<

        判断小于，有特定的指令`slt`

        ```nasm
        slt a0,a0,a1    ; a0<a1,则a0=1
        ```
      
      - ③.判断<=

        反向思维变了个花样：如果 a0<=a1成立,那么则有 a1<a0不成立，所以执行 `slt a0,a1,a0` 得到a0=0,再 a0 异或 1=1（约定<=则返回1）

        ```nasm
        slt a0, a1, a0
        xori a0, a0, 1
        ```