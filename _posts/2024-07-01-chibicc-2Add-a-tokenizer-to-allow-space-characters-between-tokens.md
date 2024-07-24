---
title: Add a tokenizer to allow space characters between tokens
date: 2024-07-01
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---


这节有tokenize写法的雏形(典型的consume、expect、eof标记和空白忽略)。具有两次遍历，一次遍历忽略空白字符串生成token，第二次遍历token链表对加减表达式求和。

## 知识点

- calloc`(size_t nitems, size_t size)`

这个函数声明在stdlib.h，记得malloc但没用过calloc.

malloc和calloc()的主要区别是前者不能初始化所分配的内存空间,而后者能.

- va_\*。声明在**stdarg.h**用来访问函数的可变参数。使用步骤
    - 定义一个可变参数函数，至少要有一个固定参数（即 last 参数）。
    - 声明一个 va_list 变量，用于存储可变参数列表的信息。
    - 使用 va_start 宏初始化 va_list 变量。
    - 使用 va_arg 宏依次获取每个可变参数。
    - 使用 va_end 宏清理 va_list 变量。

    ```c
    void total(int count,...){
        va_list va;
        va_start(va,count);
        int result=0;
        for(int i=0;i<count;i++){
            result+=va_arg(va,int);
        }
        va_end(va);
        printf("result %d",result);
    }
    ```
- isspace

判断是否空格字符串。习惯了手写居然不知道ctype.h还有这个函数

## 结果
```c
#include <ctype.h>
#include <stdarg.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
typedef enum{
    TK_RESERVED,//Keywords or punctuators
    TK_NUM,//intege literals
    TK_EOF,//end of file
}TokenKind;

typedef struct Token Token;
struct Token{
    TokenKind kind;
    Token *next;    
    long val;       //if kind is TK_NUM,its value
    char *str;      //Token string
};
Token *token;//current token

//reports an error and exit.
void error(char *fmt,...){
    va_list ap;
    va_start(ap,fmt);
    vfprintf(stderr,fmt,ap);
    fprintf(stderr,"\n");
    va_end(ap);
    exit(1);
}

// consumes the current token if it matches `op`
bool consume(char op){
    if(token->kind!=TK_RESERVED||token->str[0]!=op)
        return false;
    token=token->next;
    return true;
}

//ensure that the current token is `op`
void expect(char op){
    if(token->kind!=TK_RESERVED||token->str[0]!=op)
        error("expected '%c'",op);
    token=token->next;
}

//ensure that the current token is TK_NUM.
long expect_number(void){
    if(token->kind!=TK_NUM)
        error("expected a number");
    long val=token->val;
    token=token->next;
    return val;
}

bool at_eof(void){
    return token->kind==TK_EOF;
}

//create a new token and add it as the next token of `cur`
Token *new_token(TokenKind kind,Token *cur,char *str){
    Token *tok=calloc(1,sizeof(Token));
    tok->kind=kind;
    tok->str=str;
    cur->next=tok;
    return tok;
}

//Tokenize `p` and returns new tokens
Token *tokenize(char *p){
    Token head={};
    Token *cur=&head;
    while(*p){
        if(isspace(*p)){
            p++;
            continue;
        }

        if(*p=='+'||*p=='-'){
            cur=new_token(TK_RESERVED,cur,p++);//直接指针指到指定的位置，比较的时候所以只比较token->str[0]
            continue;
        }

        if(isdigit(*p)){
            cur=new_token(TK_NUM,cur,p);
            cur->val=strtol(p,&p,10);
            continue;
        }
        error("invalid token");
    }
    new_token(TK_EOF,cur,p);
    return head.next;//头是空的直接指向下一个
}

int main(int argc,char **argv){
    if(argc!=2){
        fprintf(stderr,"%s:invlid number of arguments\n",argv[0]);
        return 1;
    }
    token= tokenize(argv[1]);
    printf(".intel_syntax noprefix\n");
    printf(".global _main\n");
    printf("_main:\n");
    printf("    mov rax, %ld\n",expect_number());
    while(!at_eof()){
        if(consume('+')){
            printf("    add rax, %ld\n",expect_number());
            continue;
        }
        expect('-');
        printf("    sub rax, %ld\n",expect_number());
    }
    printf("  ret\n");
    return 0;
}
```