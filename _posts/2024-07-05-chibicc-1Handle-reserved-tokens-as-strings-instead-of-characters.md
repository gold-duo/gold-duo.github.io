---
title: Handle reserved tokens as strings instead of characters
date: 2024-07-05
categories: [compiler,chibicc]
tags: [chibicc,rvcc] 
---

将词法切割出的字符用字符串表示，现在只处理 +-*/()符号。为后续要处理关键字、不止一个字符的符号做准备


## 知识点

一个strncmp函数，比较两个字符串的特定长度，相等则返回0
