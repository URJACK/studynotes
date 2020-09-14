# Latex

## 安装

## 基础

### eg1

#### part1

```latex
\documentclass{ctexart}
\usepackage{amsthm}
\theoremstyle{definition}
\theoremstyle{remark}
\newtheorem{defn}{Definition}
\newtheorem*{rem}{Remark}
\newtheorem{lem}{lemma}


\begin{document}

\begin{lem} Hello Latex \end{lem}
\begin{defn} Nihao Latex \end{defn}
\begin{rem} Nihao Latex \end{rem}
    
\end{document}
```

---

![image-20200913105600928](D:\LearningNotes\other\image-20200913105600928.png)

篡改其中的一些内容

```latex
\documentclass{ctexart}
\usepackage{amsthm}
\newtheorem{defn}{Definition}
\newtheorem*{rem}{Remark}
\newtheorem{lem}{lemma}


\begin{document}

\begin{lem} Hello Latex \end{lem}
\begin{defn} Nihao Latex \end{defn}
\begin{rem} Nihao Latex \end{rem}

\end{document}
```

---

![image-20200913105918898](D:\LearningNotes\other\image-20200913105918898.png)

#### part1 结论theoremstyle

我们发现**theoremstyle**并不是必须的，它仅仅影响了显示的样式。

#### part2

于是，我们将

```
\newtheorem{defn}{Definition}
更改为
\newtheorem{defn}{ArkNize}
```

得到的结果是

![image-20200913110028329](D:\LearningNotes\other\image-20200913110028329.png)

#### part2 结论newtheorem 

**newtheorem** 会给一个\begin \end 开头的编码块起一个别名。

### eg2

新设立一种风格newtheoremstyle

![image-20200913110601533](D:\LearningNotes\other\image-20200913110601533.png)
$$
G=argmin_GE_{z-q(z)}[h(avg(E(G(z)))) - λp(z,E(G(z)))]
$$
