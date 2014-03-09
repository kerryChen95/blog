# CSS经典问题系列：理解z-index

对于一个stacking context而言，在z方向上从下到上依次为：
1. 创建这个stacking context的根元素本身，也就是根元素的背景和border;
2. z-index为负数的定位子孙元素
3.1. 普通流中的块级子孙元素
3.2. 浮动的子孙元素
3.3. 普通流中的行内子孙元素，包括根元素直接包含的文本
3.4. z-index为0的定位子孙元素
4. z-index为正数的定位子孙元素

接下来我们看个例子，用之前提到的理论知识实现如下需求：当hover一个元素的时候，让其具有渐变的背景能够以过渡的效果展示出来。
分析一下这个需求，要实现渐变背景，显然要用background-image。但我们还要有一个过渡展现出来的效果，显然要用transition属性，但我们知道，background-image属性不能作为transition-property的有效值，那怎么办呢？opacity可以作为transition-property的有效值，于是我们可以曲线救国一下，用一个元素实现渐变背景，hover时改变这个元素的opacity属性即可。
初步的实现代码如下：

```HTML
<div id="box">
    <p>这是一段文字</p>
</div>
```

```CSS
#box {
    position: relative;
}
#box::before {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(#000, #fff);
    transition: opacity 0.3s;
    opacity: 0;
}
#box:hover::before {
    opacity: 1;
}
```

这里为了减少节点，使用before伪元素来实现渐变背景，而且毕竟这是一个纯粹为了实现样式而存在的元素。
但这个实现方案有个问题，就是before伪元素会挡住#box中的非定位元素，如前文所述，定位了的子孙元素，总是会覆盖在未定位的子孙元素上面。于是这就需要用到z-index来实现我们的目标了。
但是怎么用呢？很多人或许会先给before伪元素先设置z-index试试看，设为0，还是覆盖在上面，再设为-1，虽然不再覆盖在上面了，但如果#box有设置了定位的祖先元素，这个before伪元素就会被那个定位祖先元素覆盖，所以还是不设为-1的好。


