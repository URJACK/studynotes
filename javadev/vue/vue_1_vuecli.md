# Vue-cli

## 基础知识

### 生命周期

![img](.\1.png)

## 易错细节

### 访问DOM

```html
<template>
  <div class="container" id="app">
    Canvas
    <canvas id="oCanvas" width="600" height="600"></canvas>
    <button v-on:click="debug">click</button>
  </div>
</template>
```

```javascript
  created() {
    console.log("CanvasPoint created");
    app = this;
    window.onload = function() {
      //在vue的函数中，使用oCanvas就默认使用了document.getElementById(oCanvas)
      initCanvas(oCanvas);
    };
  },
```

### 函数命名

```javascript
    getDrawArrayItemValue(y, x) {
      //正确的命名方式
      return this.drawArray[app.width * y + x];
    },
    getDrawArrayItemValue: (y, x) => {
      //该种命名方式，会导致使用this无法访问自己的成员变量与函数
      return this.drawArray[app.width * y + x];
    },
```

