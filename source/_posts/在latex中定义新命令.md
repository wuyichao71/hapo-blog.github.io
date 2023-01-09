---
title: 在latex中定义新命令
date: 2023-01-07 15:03:02
mathjax: true
categories:
    - latex
tags:
    - latex
---

# 定义新命令
为了在latex中书写方便，可以用`newcommand`定义新命令:
```latex
\newcommand{\<cmdname>}[<n>]{<command>}
```
+ `<cmdname>`为新定义的命令名字。  
+ `<n>`为参数个数，各个参数可以在命令体中用`#1`、`#2`表示  
+ `<command>`为命令体。  
例如，可以用`newcommand`定义平均值的表示:
```latex
\newcommand{\mean}[2]{\frac{#1_1 + #1_2 + \cdots + #1_#2}{#2}}
```
之后可以这样在数学环境中使用
```latex
\mean{a}{n}
```
但是`newcommand`不允许定义一个已经存在的命令，如果要防止报错，可以使用`providecommand`，该命令使用和`newcommand`一致，当命令不存在时，它相当于`newcommand`，当命令存在时，它沿用之前的定义。例如:
```latex
\providecommand{\mean}[2]{\frac{#1+#2}{2}}
```
则`\mean{a}{n}`依旧等于
$$ \frac{a_1+a_2+\cdots+a_n}{n} $$
有时候我们需要重新定义一个已经定义的命令，此时，我们可以用`renewcommand`，`renewcommand`的使用和`newcomand`相同，但是它必须以及存在原命令，否则会报错。
这些命令的定义会受到局部环境的影响，即在环境内部定义的命令在外部无法使用。
<!--more-->
# 使用局部命令
为了让书写简单美观，我们常常希望命令能够像编程语言的变量一样，即可以重新定义后对之后的代码都生效。使用`newcommand`和`renewcommand`我们可以如下操作:
```latex
\newcommand{\mean}[2]{\frac{#1_1 + #1_2 + \cdots + #1_#2}{#2}}
$$\mean{a}{n}$$
\renewcommand{\mean}[2]{\frac{#1+#1}{2}}
$$\mean{a}{n}$$
```
这时候两个`\mean`的效果是不一致的。但是这样做有个缺陷，即我们无法知道该命令是否已经定义，如果已经定义，那么第一个`\newcommand`需要改为`\renewcommand`。  

为了克服这个缺陷，我们可以使用`def`,`def`的语法为:
```latex
\def\⟨name⟩<parameter text>{⟨definition⟩}
```
* `<name>`为新定义的命令名字。  
* `<parameter text>`为参数定义，可选，比如我不需要参数时候可以不写，我需要三个参数时为`#1#2#3`。  
* `<definition>`为命令体，其中的参数用`#1`、`#2`……表示。  

`def`不需要检查该命令是否定义，因此适合用来该操作。同时`def`会受到局部环境的影响，需要定义全局的命令可以用`gdef`，如果要对`<definition>`中的命令进行展开，可以用`edef`，全局定义展开时可以用`xdef`

