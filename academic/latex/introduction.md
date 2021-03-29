# Latex

规范化的一种模版输出。

## Latex的简介

### 编写结构化文档

### 文档的组成

1. 标题
2. 前言/摘要
3. 目录
4. 正文
   - 篇、章、节、小节、小段
     - 文字、公式
     - 列表:编号的、不编号的、带小标题的
     - 定理、引理、命题、证明、结论
     - 诗歌、引文、程序代码、算法伪码
     - 制表
     - 画图
5. 文献
6. 索引、词汇表

只有<u>具有良好的结构</u>的文档才<u>适合使用Latex</u>进行编写。

### 使用Latex的流程

- 拟定主题
- 列出提纲
- 填写内容
- 调整格式（不要在意格式）

## Latex结构

```
%%%简单文档
% 导言:格式设置
\documentclass{ctexart}
\usepackage[b5paper]{geometry}
% 正文:填写内容
\begin{document}
使用\LaTeX
\end{document}
```

### 文档部件

```
标题: \title, \author, \date- \maketitle
摘要/前言: abstract 环境/ \chapter*
目录: \tableofcontents
章节: \chapter, \section,..
附录: \appendix + \chapter或\section ...
文献: \bibliography
索引: \printindex
```

大型文档: \frontmatter、 \mainmatter、 \backmatter

一般文档: \appendix

| 层次 | 名称          | 命令           | 说明                                     |
| ---- | ------------- | -------------- | ---------------------------------------- |
| -1   | part          | \part          | 可选的最高层                             |
| 0    | chapter       | \chapter       | report，book类最高层                     |
| 1    | section       | \section       | article类的最高层                        |
| 2    | subsection    | \subsection    |                                          |
| 3    | subsubsection | \subsubsection | report类、book类<br>默认不编号、不编目录 |
| 4    | paragraph     | \paragraph     | 默认不编号、不编目录                     |
| 5    | subparagraph  | \subparagraph  | 默认不编号、不编目录                     |

### 相关命令

```
\documentclass:读入文档类文件(.cls)
针对不同的需要投稿的机构，采用不同的文档类文件

\usepackage:读入一个格式文件----宏包(.sty)
数学公式、插图的一些宏包

\include:分页，并读入章节文件(.tex)
读入我们自身的文件，比如每一个章节

\input:读入任意的文件
```

### Latex的语法结构

命令

<img src=".\imgs\image-20210301112013850.png" alt="image-20210301112013850" style="zoom:50%;" />

环境

<img src=".\imgs\image-20210301112104451.png" alt="image-20210301112104451" style="zoom:50%;" />

注释：以%开头。

## Latex编写

### Helloworld（指定编码）

```
%%%简单文档
% 导言:格式设置 article 我这边改用了ctexart
% article不允许有中文的出现 但中文作为注释是可行的
\documentclass{ctexart}
% 正文:填写内容
\begin{document}
Hello World
\end{document}
```

使用UTF-8指定编码

```
documentclass{UTF8]{ctexart}
\begin{document}
今天你吃了吗?
\end{document}
```

### 正文文本

```
直接输入正文文本。
用空格分开单词。一个换行符等同于一个空格，多个空格的效果与一一个相同。
自然段分段是空一行。
```

例如

```tex
\documentclass{ctexart}
\begin{document}
aabbCCddeeff
aabbCCddeeff

aabbCCddeeff
aabbCCddeeff
\end{document}
```

### 正文符号

一些符号被ETpX宏语言所占用，需要以命令形式输入:

```
\#\$\%
\& \{ \}
\textbackslash
```

对应的符号是

```
# $ % & { } \
```

还有一些其他奇怪的符号

```
\S \dag \ddag \P \copyright
\textbullet \textregistered
\texttrademark 
\pounds
```

### 数学公式

数学模式下字体、符号、间距与正文都不同，一切数学公式(包括单个符号n,π)都要在数学模式下输入。

- 行内(inline) 公式:

  ```
  使用一对符号$ $来标示。如$a+b=c$。
  ```

- 显示(display) 公式。

  ```
  - 简单的不编号公式用命令\[和\]标示。(不要使用双美元符号$$ $$)
  - 基本的编号的公式用equation环境。
  - 更复杂的结构，使用amsmath宏包提供的专门的数学环境。(不要使用eqnarray环境)
  ```

#### 数学公式的结构

- 上标与下标:用^和_表示。

- 上下画线与花括号: \overline, \underline, \overbrace, \underbrace

- 分式: \frac{分子}{分母}

- 根式: \sqrt[次数]{根号 下}

- 矩阵:使用amsmath宏包提供的专门的矩阵环境matrix, pmatrix, bmatrix等。特别复杂的矩阵(如带线条)使用array环境作为表格画出。

  ```tex
  \documentclass{article}
  \usepackage{amsmath}
  \usepackage{mathtools}
  \begin{document}
  \begin{align*}
  2^5 &= (1+1)^5 \\
  &= \begin{multlined}[t]
  \binom{5}{0}\cdot 1^5 + \binom51\cdot 1^4 \cdot 1
  + \binom52\cdot 1^3 \cdot 1^2 \\
  + \binom53\cdot 1^2 \cdot 1^3
  + \binom54\cdot 1 \cdot 1^4 + \binom55\cdot 1^5
  \end{multlined} \\
  &= \binom50 + \binom51 + \binom52 + \binom53
  + \binom54 + \binom55
  \end{align*}
  %你好
  $\binom{12}{5}$
  \end{document}
  ```

#### 数学符号

- 数学字母：a,b,c,o、数学字体\mathbb (R)、 \mathcal (P) 等
- 普通符号：如\infty (∞) , \angle（∠）
- 二元运算符：a + b,a - b
- 二元关系符：a = b,a < b
- 括号：<a , b>， 使用 \left, \right放大
- 标点：逗号、分号(\colon)

#### 数学单位

siunitx：数字单位的一揽子解决方案

```
\num{-1.235e96} \\
\SI{299792458}{m/s} \\
\SI{2x7x3.5}{m}
```

chemformula：编写化学式

### 列表、表格环境

#### 列表

- enumerate 编号
- itemize 不编号
- description有标题

```tex
\documentclass{article}
\begin{document}
\begin{enumerate}
  \item aaa
  \item bbb
  \item ccc
  \item ddd
\end{enumerate}
\begin{itemize}
  \item nihao
  \item hho
\end{itemize}
\begin{description}
  \item[study] academic
  \item[no] number   
\end{description}
\end{document}
```

#### 表格

```tex
\begin{tabular}{|rr|}
  \hline
  输入& 输出\\ 
  \hline
  $-2$ & 4 \\
  0 & 0 \\
  2 & 4 \\ 
  \hline
\end{tabular}
```

<img src=".\imgs\image-20210301152412387.png" alt="image-20210301152412387" style="zoom: 80%;" />

其他的表格宏包

- 单元格处理: multirow、 makecell
- 日长表格: longtable（使用表格可以智能的嵌入下一页）、xtab
- 定宽表格: xtabular（超过了宽度，就可以跳到下一行）
- 表线控制: booktabs、 diagbox（表格支持画斜线）、 arydshln
- 表列格式: array
- 综合应用: tabu

#### 表格环境

- figure环境
- table环境
- 其他环境可以使用float宏包得到

浮动体的标题用\caption命令得到，自动编号。

浮动体，这会使得表格的位置脱离了文本流，如下例所示。

```tex
\documentclass{ctexart}
\begin{document}
哈喽
\begin{table}\centering
\begin{tabular}{|ccccc|}\hline
a&b&c&d&e\\
a&b&c&d&e\\
a&b&c&d&e\\
a&b&c&d&e\\ 
\hline
\end{tabular}
\caption{标题}
\end{table}
你好
\end{document}
```

<img src=".\imgs\image-20210301153946540.png" alt="image-20210301153946540" style="zoom:50%;" />

### 定理环境

\newtheorem 定义定理类环境，如：

```tex
\newtheorem{thm}{定理}[section]
```

使用定理类环境

```tex
\documentclass{ctexart}
\newtheorem{thm}{定理}[section]
\begin{document}
\section{节}
\section{节}
\begin{thm}
  一个定理
\end{thm}
\end{document}
```

### 代码环境

使用"verb"命令

搭配"|"符号，来实现单行代码的录入

```tex
\verb| #include <stdio.h>|
```

使用"verbatim"环境，来实现多行代码的录入

```tex
\documentclass{ctexart}
\begin{document}
\begin{verbatim}
  #include <stdio.h>
  int main() {
  puts( "hello world.");
\end{verbatim}
\end{document}
```

使用<u>listwings宏包</u>、<u>minted宏包</u>、<u>clrscode宏包</u> `下面这个例子`、<u>algorithm2e宏包</u>

```tex
\documentclass{ctexart}
\usepackage{clrscode}
\begin{document}
\begin{codebox}
  \Procname{ $\proc{Merge-Sort}(A,p,r)$}
  \li \If $p<r$
  \li \Then $q \gets \lfloor(p+r)/2\rfloor$
  \li
  $\proc{Merge-Sort}(A,p,q)$
  \li
  $\proc{Merge-Sort}(A,q+1,r)$
  \li
  $\proc{Merge}(A,p,q,r)$
  \End
\end{codebox}
\end{document}
```

实现效果

<img src=".\imgs\image-20210301144012554.png" alt="image-20210301144012554" style="zoom: 80%;" />

可以实现代码的语法高亮

```shell
C:\Users\scffz> texdoc listwings
```

`texdoc命令可以很好的查看一个宏包的使用方法`

### 引用其他文件

\input的命令相当于

```
\clearpage
\input{}
\clearpage
```



### 插图

使用这个

```tex
\includegraphics[width=2cm]{pku1ogo.pdf}
```

优先使用外部工具画图，特别是可视化工具，例如一般的矢量图用Inkscape、llustrator 甚至PowerPoint (保存为pdf格式)，数学图形用MATLAB、matplotlib 之类。

<u>tikz宏包</u>，支持了诸如“有限自动机”这类代码作图方式。

###  自动化工具

#### .aux文件与.toc文件。

例如引用公式的时候，我们需要填写公式的编号，但是公式的编号可能会发生变化，这使得如果我们对该部分进行了改动，那么这会导致文章中诸多地方都需要进行改动。因此，我们事先将公式的“变量名”存入一个文件，在tex文件编译过程中，自动去填写这些公式的编号即可。

#### hyperref超引用

点击一个链接进行跳转的功能。

### 参考文献BibTEX

<img src=".\imgs\image-20210301154741727.png" alt="image-20210301154741727" style="zoom: 67%;" />

.bib文件，建议使用<u>JabRef工具</u>进行管理。

```
@BOOK{Shiye,
title = {几何的有名定理}，
publisher = {上海科学技术出版社},
year = {1986},
author = {矢野健太郎}
}

@ARTICLE{quanjing,
author = {曲安京},
title = {商高、赵爽与刘徽关于勾股定理的证明}，
journal = {数学传播},
year = {1998},
volume = {20},
}
```

## 设计文档格式

### 张三杂谈勾股定理

```tex
\documentclass[UTF8]{ctexart}
\title{\heiti杂谈勾股定理}
\author{\kaishu张三}
\date{\today}
\usepackage {geometry}
\geometry{a6paper , centering, scale=0.8}
\usepackage{graphicx}
\usepackage[format=hang, font=small, textfont=it]{ caption}
\usepackage{float}
\usepackage {amsmath}
\usepackage[nottoc]{tocbibind}
\bibliographystyle{plain}
%"\usepackage{hyperref}
\newtheorem{thm}{定理}
\newcommand\degree{^\circ}
\newenvironment{myquote}
{\begin{quote}\kaishu\zihao{-5}
{\end{quote}}
\begin{ document}
\maketitle
\begin{abstract}
这是一篇关于勾股定理的小短文。
\end{abstract}
```

### 格式与内容分离

在LaTex的设计中，将文档的格式设计与内容分离开来。标准的LaTex文档类具有相对固定的排版格式，作者编写文档只使用<u>\title、\section、abstract</u> 这样的命令或环境，而不必考虑其具体实现。而有关格式的细节代码，则被封装在文档类、宏包中，或在导言区分离编写。

使用内容相关的命令与环境！！！

格式与内容的分离不仅需要格式设计者的努力，也需要作者在填写内容时遵循分离原则。基本的方法就是只使用与内容相关的命令和环境。

- 推荐: It is \emph{important}.

  不好: It is \textit{important}.

- 推荐: \caption{ 流程图}

  不好: \textbf{图 1:} 流程图

- 推荐: \begin{verse} 诗行\end{verse}

- 不好: \begin{center} 诗行\end{center}

字体

- \rmfamily, \textrm{...}
- \sffamily, \textsf{...}
- \ttfamily, \texttt{...}

字号

```
:\Huge, \LARGE, \Large, \large, \normalsize, \small, \footnotesize, \scriptsize, \tiny
```

中文字号：

```
\zihao{5}、 \zihao{-3}
```

对齐：

```
\centering、\raggedleft、\raggedright
```

空白间距：

```
\hspace{2cm}、\vspace{3mm}
```

版面布局：

```
geometry（纸张的长宽、版心的位置）宏包、fancyhdr（章节标题标题、页眉页脚）宏包等
```

分页断行：

```
\linebreak、\\（写正文的时候不去使用）
\pagebreak（这一页到此为止，很少使用）、\newpage（另起一页，使用较多）、 \clearpage（比newpage多一个功能，就是除了正文之外，还包含了图表）、 \cleardoublepage（保证另起一章，总可以保证从奇数页开始）
```

盒子：

```
\mbox{内容}
\parbox{4em}{内容}、minipage
```

个性化格式定义：

```
如果预定义的格式不符合需要，就需要设置修改。经常文档作者本人就是格式设计者，此时更应该注意不要把格式和内容混在一起。
直接设置相关参数：如设置\parindent、\parskip、\linespread、\pagestyle。
修改部分命令定义：如修改\thesection、\labelenumi、\descriptionlabel、\figurename。
利用工具宏包完成设置：如使用ctex宏包设置中文格式，使用tocloft宏包设置目录格式。
传统的文档中经常修改eTEX的内部命令，如重定义内部命令：\l@section来修改目录格式。

这体现了当初ETEX设计的不足:没有提供足够的用户层接口来调整格式。不过这类方法比较晦涩，一般应该使用相关宏包功能代替。
```

### 自定义的命令与环境

见下面这段代码：

```tex
\documentclass{ctexart}
% \newcommand\prg[1]{\textsf{#1}}
\newcommand\prg[1]{\textbf{\Huge #1}}
\begin{document}
程序\prg{sort} 很有用。
\end{document}
```

由此可见，使用定义的格式来编写是非常编写的。

\setlist{nosep} 可以更改enumitem环境中的item的间距

可以使用 `texdoc enumitem`命令进行查看相关环境的命令修改

```tex
\documentclass[a4paper,12pt]{ctexart}
\usepackage{graphicx}
\usepackage{multicol}
\title{基于蚁群算法通过移动RPL结点的位置改进能耗的方案}
\author{Fu Fangzhou}
\begin{document}
\maketitle
\clearpage
\end{document}
```

