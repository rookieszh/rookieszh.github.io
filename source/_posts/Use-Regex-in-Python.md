---
title: Use Regex in Python
date: 2020-02-09 13:33:30
categories:
    - Programming
tags:
    - Regex
    - Python
description: "【正则表达式系列——在Python中使用正则表达式】 1.原始字符串的概念。2.常用函数的介绍与实战使用。3.常量模块。4.异常"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 原始字符串
```python
# 原始字符串（raw string）是所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符，通常简称为 r-string。
# 可以用于设定匹配规则时使用
print('\blake')
print(r'\blake')
>>> lake
    \blake
```
## 正则表达式的使用
```python
import re
def look_for(pat, str):
    return '没有找到' if re.search(pat, str) is None else re.findall(pat, str)

# 小括号
pat = r'beat(s|ed|en|ing)'
print( look_for(pat, 'beats') )
print( look_for(pat, 'beated') )
print( look_for(pat, 'beaten') )
print( look_for(pat, 'beating') )
>>> ['s']
    ['ed']
    ['en']
    ['ing']

pat = r'(beat(s|ed|en|ing))'
print( look_for(pat, 'beats') )
print( look_for(pat, 'beated') )
print( look_for(pat, 'beaten') )
print( look_for(pat, 'beating') )
>>> [('beats', 's')]
    [('beated', 'ed')]
    [('beaten', 'en')]
    [('beating', 'ing')]

pat = r'(beat(?:s|ed|en|ing))'
print( look_for(pat, 'beats') )
print( look_for(pat, 'beated') )
print( look_for(pat, 'beaten') )
print( look_for(pat, 'beating') )
>>> ['beats']
    ['beated']
    ['beaten']
    ['beating']

# \b \B
pat = r'\blearn\b'
print( look_for(pat, 'learn Python') )
print( look_for(pat, 'relearn Python') )
print( look_for(pat, 'learning Python') )
print( look_for(pat, 'relearning Python') )
>>> ['learn']
    没有找到
    没有找到
    没有找到

pat = r'\Blearn\B'
print( look_for(pat, 'learn Python') )
print( look_for(pat, 'relearn Python') )
print( look_for(pat, 'learning Python') )
print( look_for(pat, 'relearning Python') )
>>> 没有找到
    没有找到
    没有找到
    ['learn']
```
## 常用函数
```python
# match(pat, str)：检查字符串的开头是否符合某个模式
# 该函数返回的是个对象（包括匹配的子字符串和在句中的位置索引），如果只需要子字符串，需要用 group() 函数。
# 由于值匹配句头，那么句中的 Bryant 无法被匹配到。
s = 'Kobe Bryant'
print( re.match(r'Kobe', s) )
print( re.match(r'Kobe', s).group() )
print( re.match(r'Bryant', s) )
>>> <re.Match object; span=(0, 4), match='Kobe'>
    Kobe
    None

# search(pat, str)：检查字符串中是否符合某个模式，但只匹配第一个
s = 'Kobe Bryant'
print( re.search(r'Kobe', s) )
print( re.search(r'Kobe', s).group() )
print( re.search(r'Bryant', s) )
print( re.search(r'Bryant', s).group() )
>>> <re.Match object; span=(0, 4), match='Kobe'>
    Kobe
    <re.Match object; span=(5, 11), match='Bryant'>
    Bryant

# findall(pat, str)：返回所有符合某个模式的字符串，以列表形式输出
s = 'Kobe Bryant loves Gianna Bryant'
print( re.findall(r'Kobe', s) )
print( re.findall(r'Bryant', s) )
print( re.findall(r'Gigi', s) )
>>> ['Kobe']
    ['Bryant', 'Bryant']
    []

# finditer(pat, str)：返回所有符合某个模式的字符串，以迭代器形式输出
# 如果需要匹配子串在原句中的位置索引，用 finditer，此外用 findall。

# split(pat, str)：以某个模式为分割点，拆分整个句子为一系列字符串，以列表形式输出
s = 'Kobe Bryant loves Gianna Bryant'
print( re.split(r'\s', s) )
>>> ['Kobe', 'Bryant', 'loves', 'Gianna', 'Bryant']

# sub(pat, repl, str)：句子 str 中找到匹配正则表达式模式的所有子字符串，用另一个字符串 repl 进行替换
s = 'Kobe Bryant loves Gianna Bryant'
print( re.sub(r'\s', '-', s) )
>>> Kobe-Bryant-loves-Gianna-Bryant

# compile(pat)：将某个模式编译成对象，供之后使用，如match 和 search 使用
email = '''Shengyuan Personal: quantsteven@gmail.com
Shengyuan Work: shengyuan@octagon-advisors.com
Shengyuan School: g0700508@nus.edu.sg
Obama: barack.obama@whitehouse.gov'''


pat = r'[\w.-]+@[\w.-]+'
obj = re.compile(pat)
obj
>>> re.compile(r'[\w.-]+@[\w.-]+', re.UNICODE)

print( obj.match(email), '\n')
print( obj.search(email), '\n' )
print( obj.findall(email), '\n' )
>>> None
    <re.Match object; span=(20, 41), match='quantsteven@gmail.com'> 
    ['quantsteven@gmail.com',
    'shengyuan@octagon-advisors.com',
    'g0700508@nus.edu.sg',
    'barack.obama@whitehouse.gov']

obj1 = re.compile(r'@')
for addr in obj.findall(email):
    print( obj1.split(addr))
>>> ['quantsteven', 'gmail.com']
    ['shengyuan', 'octagon-advisors.com']
    ['g0700508', 'nus.edu.sg']
    ['barack.obama', 'whitehouse.gov']
```
## 常量模块
```python
# 常量可叠加使用，因为常量值都是2的幂次方值，所以是可以叠加使用的，叠加时请使用 | 符号
# 使用举例
re.findall(pattern, string, re.模块)

# re.IGNORECASE / re.I : 忽略大小写
# re.ASCII / re.A : 只匹配 ASCII 码，让 \w,\W,\b,\B,\d,\D,\s和\S 只匹配ASCII， 不匹配 Unicode 等其他码
# re.DOTALL / re.S : 让 . 能匹配所有，包括换行符\n。默认模式下 . 是不能匹配行符\n的
# re.MULTILINE / re.M : 多行模式，当某字符串中有换行符\n，默认模式下是不支持换行符特性的，比如：行开头 和 行结尾，而多行模式下是支持匹配行开头的。
# re.VERBOSE / re.X : 详细模式，可以在正则表达式中加注解！
# re.UNICODE / re.U : 与 ASCII 模式类似，匹配unicode编码支持的字符，但是 Python 3 默认字符串已经是Unicode，所以有点冗余。
# re.DEBUG : 显示编译时的debug信息

```
## 异常
```python
# re模块还包含了一个正则表达式的编译错误
# 当我们给出的正则表达式是一个无效的表达式（就是表达式本身有问题）时，就会raise一个异常

```