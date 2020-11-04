# css（1）

## 伪类

CSS **伪类** 是添加到选择器的关键字，指定要选择的元素的特殊状态。

```css
/* 所有用户指针悬停的按钮 */
button:hover {
  color: blue;
}
```

### 常见伪类

#### LVHA

**link**

`:link`伪类选择器是用来选中元素当中的链接。它将会选中所有尚未访问的链接，包括那些已经给定了其他伪类选择器的链接（例如[`:hover`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:hover)选择器，[`:active`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:active)选择器，[`:visited`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:visited)选择器）。为了可以正确地渲染链接元素的样式，:link伪类选择器应当放在其他伪类选择器的前面，并且遵循LVHA的先后顺序，即：`:link` — `:visited` — `:hover` — `:active`。`:focus`伪类选择器常伴随在`:hover`伪类选择器左右，需要根据你想要实现的效果确定它们的顺序。

**visited**

**`:visited`** CSS[伪类](https://developer.mozilla.org/en-US/docs/CSS/Pseudo-classes)表示用户已访问过的链接。出于隐私原因，可以使用此选择器修改的样式非常有限。

**hover**

`:hover` CSS伪类适用于用户使用指示设备虚指一个元素（没有激活它）的情况。这个样式会被任何与链接相关的伪类重写，像[`:link`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:link), [`:visited`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:visited), 和 [`:active`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:active)等。为了确保生效，:hover规则需要放在:link和:visited规则之后，但是在:active规则之前

**active**

**`:active`** [伪类](https://developer.mozilla.org/zh-CN/docs/CSS/Pseudo-classes)匹配被用户激活的元素。它让页面能在浏览器监测到激活时给出反馈。当用鼠标交互时，它代表的是用户按下按键和松开按键之间的时间。

`:active` 伪类一般被用在 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a) 和 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/button) 元素中. 这个伪类的一些其他适用对象包括包含激活元素的元素，以及可以通过他们关联的[``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/label)标签被激活的表格元素。

## 函数

### calc函数

```css
.banner {
  position: absolute;
  left: 40px;
  width: calc(100% - 80px);
  border: solid black 1px;
  box-shadow: 1px 2px;
  background-color: yellow;
  padding: 6px;
  text-align: center;
  box-sizing: border-box;
}
```