# Latex问题记录

## 引用参考文献

在下面这段代码中，需要做到如下几步才可以引用参考文献：

1. `\usepackage{cite}` 引用cite包
2. 在文档内部使用cite命令 `\cite{rpl}` ，这里的 `rpl` 代表的是.bib文件中的，参考文献的变量名。
3. 在文档末尾加入参考文献的列表 `\bibliography{ref}` ，这里的 `ref` 代表的是 `ref.bib`，即bib文件的名称。

```tex
\documentclass[a4paper,12pt,UTF8]{ctexart}
\usepackage{cite}
\usepackage{graphicx}
\usepackage{multicol}
\title{基于蚁群算法通过移动RPL结点的位置改进能耗的方案}
\author{Fu Fangzhou}
\begin{document}
	\maketitle
	\clearpage
	\cite{rpl}
	\input{part_abstract}
	\input{part_introduction}
	\bibliographystyle{plain}
	\bibliography{ref}
\end{document}
```

需要注意的是，上面这个步骤仅仅代表代码上这样是可以引用参考文献了，但是在具体编译的步骤中还有一些点需要注意：

1. 编译`ref.bib`文件成为`main.bbl`

   只有成为`.bbl`的文件后，使用 `\cite` 和 `\bibliography` 命令才会真正起效。

   <img src="D:\LearningNotes\academic\latex\image-20210304144655428.png" alt="image-20210304144655428" style="zoom:50%;" />

2. 而这个`步骤1.`的实现，我们将基于 **TeXstudio** 来完成

   `Tools->Commands->Bibtex` 就可以快速的完成 bib文件 到 bbl文件 的转换。 

   <img src="D:\LearningNotes\academic\latex\image-20210304145138976.png" alt="image-20210304145138976" style="zoom:50%;" />

3. 在完成bbl文件的转换后（显然如果更新了参考文献，还需要重新执行这次bbl文件的更新），

   再执行 `F6 compile` ，就可以得到完整的PDF文档。

## 多列并行行文

在计算机科学的文章格式中我们可以看到，它们每一页的行文都是两列并行的。需要做到如下几步才可以执行多列并行行文：

1. 使用 `\usepackage{multicol}` 引入`multicol包`。
2. 使用 `\begin{multicols}{2}` 开启 `multicols` 环境，在这个环境下的文本都会是两列的形式

```tex
\documentclass[a4paper,12pt,UTF8]{ctexart} %制定纸张大小和编码格式
\renewcommand\emph[1]{\textbf{#1}} %强调 从斜体 改为 加粗
\usepackage{cite}	%使用cite包 从而能够正常引用参考文献
\usepackage{multicol}	%使用multicol包 从而可以多列并行
\title{基于蚁群算法通过移动RPL结点的位置改进能耗的方案}
\author{Fu Fangzhou}
\begin{document}
	\maketitle
	\clearpage
	\input{part_abstract}
	% \columnseprule=1pt         % 实现插入分隔线
	\begin{multicols}{2}       % 分两栏 若花括号中为3则是分三列
	\input{part_introduction}
	\end{multicols}
	\bibliographystyle{plain}
	\bibliography{ref}
\end{document}
```

