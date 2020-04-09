---
title: Use Regex in Python
date: 2020-04-09 02:33:30
tags:
    - Regex
    - Python
description: "How to use regex in python?"
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