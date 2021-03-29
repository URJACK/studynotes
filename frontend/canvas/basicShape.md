# 基础图形绘制

## 圆形circle



```javascript
X = Math.PI*0.5
cxt.arc(15,15,15,X,Math.PI*2,false);
```

X的不同取值，绘图如下

![image-20201224095756072](D:\LearningNotes\frontend\canvas\image-20201224095756072.png)

```javascript
X = Math.PI*0.25
cxt.arc(15,15,15,Math.PI*0.25,Math.PI*1,true);
```

![image-20201224100048939](D:\LearningNotes\frontend\canvas\image-20201224100048939.png)