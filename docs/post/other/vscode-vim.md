---
title: vim 基础命令
date: 2020-12-12
tags:
 - vim
categories:
 - 其他

---



## vim 基础命令

附上一个学习vim的游戏地址https://vim-adventures.com/，前三关免费，后面需要付费，有点小贵😢

### nomal模式

#### 基础移动

j	下移

k   上移

h   左移

l	右移

#### 单词移动

w	移动到下个单词的开头 

e	移动到下个单词的结尾

b	移动到上个单词的开头，w的反向操作

#### 行内移动

^	移动到行首的第一个非空字符

$	移动到行尾

0	移动到行首

g_  移动到该行的最后一个非空字符

推荐用`^`和`$`进行首尾的移动

#### 范围内移动

数字+G	跳转到第N行

gg	跳转到第一行

G	跳转到最后一行

%	光标移到匹配括号的另一半上

\<leader>+k/j  根据字母向上/下方向查找

g;	跳到 上一次编辑的地方

g,	跳到 下一次编辑的地方

#### 删除

x	删当前光标所在的一个字符并存在剪贴板

dw	删除当前光标后部分的单词

d^   删除光标前面的字符直到行首

d$   删除光标及其后面的字符直到行末

dd	删除整行

#### 更改

r	对当前光标所在的位置进行替换，再输入一个字符即可替换

cw	将光标后的剩余的单词部分删除并进入insert模式 

c$	将光标后的所有单词删除并进入insert模式

#### 插入

a	在光标后插入

i	在光标前插入

o	在当前行后插入一个新行，进入插入模式

O	在当前行前插入一个新行，进入插入模式

####  查找替换

/pattern	输入正则后回车回查找匹配的字符，按n查找下一个,N查找上一个

\#	匹配当前光标所在的单词，向上查找

\*	匹配当前光标所在的单词，向下查找

#### 复制/粘贴

ye	从当前光标复制到单词的结尾

y$	从当前光标复制到该行的最后一个非空字符

yy	复制一整行

p	粘贴

#### 其他

数字+命令	重复几次命令

. 	执行上次的命令

v 可视化，可批量选择文本

V	可批量选择多行

gu 变小写

gU 变大写

### Insert 模式

ctrl+h 删除前一个字符

ctrl+w 删除前一个单词

ctrl+u 删除前一行
