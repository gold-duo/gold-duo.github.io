---
title: 0302Beautiful Soup
date: 2023-09-02 13:30
categories: [python,网络爬虫开发实战]
tags: [python,网络爬虫开发实战,Beautiful Soup,XML,XML解析库] 
---

Beautiful Soup是一个HTML/XML解析库，提供简单的、Python式的函数来处理导航、搜索、修改分析树等功能。

## 安装:`pip3 install beautifulsoup4`
### vcode  `No module named 'bs4'` 问题：
- 1.`commond+shift+p`(windows`ctrl+shift+P`)
- 2.Python Interpreter -> Python 3.12.0
### 代码验证安装

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup('<p>Hello</p>', 'lxml') #第一个参数为html文本，第二给为解释器(支持的解析器有:lxml、xml、html.parser、html5lib)
print(soup.p.string)
```

## 使用（p101）
### 选择器操作集合

```python
html = """<html>
  <head>
    <title>The Dormouse's story</title>
  </head>
  <body>
    <p class="title" name="dromouse">
      <b>The Dormouse's story</b>
    </p>
    <p class="story">Once upon a time there were three little sisters; and their names were
      <a class="sister" href="http://example.com/elsie" id="link1">
        <!-- Elsie --></a>,
      <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>and
      <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>; and they lived at the bottom of a well.</p>
    <p class="story">...</p>
  </body><button name="b1"/><button name="b2"/><button name="b3"/><button name="b4"/>
</html>"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
```

操作|描述|实例
---|---|---
prettify()|格式化|
节点选择器|直接用 **.节点名** 即可选择节点|soup<font color="red">.title</font>，选择到**title**节点，返回`<title>The Dormouse's story</title>`
获取节点名称| .n.name利用name属性获取节点名称|soup.title<font color="red">.name</font>，返回`title`
获取节点属性|1, .n.attrs["xx_attr"]<br/>2, .n["xx_attr"]|soup.p.attrs["name"]<br/>soupt.p["name"]，返回`dromouse`
获取节点内容|string|soup.title<font color="red">.string</font>，返回`The Dormouse's story`
嵌套节点选择|.n1.n2.n3|soup<font color="red">.head.title</font>.string，返回`The Dormouse's story`
获取**直接子节点**|1, contents（返回list）<br/>2, children（返回迭代器）|soup.body<font color="red">.contents</font>。**注意子节点内的嵌套节点不会单独列出来**<br/>soup.body<font color="red">.children</font>
获取父/祖先节点|parent<br/>parents(返回所有祖先节点，迭代器)|soup.p<font color="red">.parent</font>，返回`body节点`<br/>soup.p<font color="red">.parents</font>，递增向上返回节点`p、body、html、document`
兄弟节点|next_sibling（下一个节点）<br/>next_siblings<br/>previous_sibilng（前一个节点）<br/>previous_sibilngs<br/><font color="red">注意：空格、换行也被当作一个节点</font>|soup.button<font color="red">.next_sibling</font>["name"]，返回`b2`<br/>soup.button<font color="red">.previous_sibling.name</font>，返回`body`

### 方法、属性集合(p107)

方法|描述|实例
---|---|---
find_all(name,attrs,recursive,text,\*\*kwargs)|<span style='white-space:nowrap;'>查找所有符合条件的节点。参数:<br/>name:节点名<br/>attrs:节点属性(list类型。对于 **id、class[class_]等常用参数可直接当作参数传入**)<br/>text:匹配节点文本，返回也是文本(可以是正则表达式)</span>|<span style='white-space:nowrap;'>soup.find_all(name="p",attrs={"class":"story"} )<br/>soup.find_all(name="p",class_="story" )<br/>soup.find_all(text=re.compile("Once upon"))，返回`['Once upon a time there were three little sisters; and their names were\n      ']`</span>
find|find_all的弱化版。只返回一个匹配节点|soup.find(name="p")
select()|css选择器|soup.select("body p a#link1")，选择 body->p->a且id="link1"的节点<br/>soup.select("p.story")，选择p节点且class="story"
string<br/>get_text()|获取文本|soup.select("body p a#link2")[0].string<br/>soup.select("body p a#link2")[0].get_text()，返回`Lacie`